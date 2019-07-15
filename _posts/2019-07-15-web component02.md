---
layout: post
title: web component - Custom Element
categories: frontend
tags: Javascript web-component
comments: true
---

> web components 구성요소 - 01. Custom Element
{:.lead}
* list
{:toc}

## 개요
Web Component의 구성요소 중에 첫번째로 <code>Custom Element</code>에 대해 어떤 방식으로 쓰이고 어떤 개념인지 한번 파악해보자. 

## Custom Element의 명명규칙
Custom Element를 사용함에 있어서 특별한 네이밍 규칙이 있다. 그것은 바로 <code>-</code>를 하나 이상 포함해야 되는 규칙이다. 또한 예약된 태그 이름이 아니여야 한다. 
~~~html
//bad
<font-face></font-face> // 예약된 태그 이름
<userinfo></userinfo> // '-' 없음

//good
<current-time></current-time>
<search-input></search-input>
~~~
이러한 제약을 가지는 이유는 HTML parser가 javascript에서 선언된 Custom Element를 모르는 상황에서 Custom Element가 될지도 모르는 태그들을 구분하기 위함이다. Custom Element 네이밍 규칙에 맞지 않는 태그들은 <code>HTMLUnknownElement</code>의 interface로 할당이 된다. 그러나 Custom Element들은 <code>HTMLElement</code>로 부터 직접 상속되어야 하므로 Custom Element 네이밍 규칙에 맞는것들의 <code>HTMLUnknownElement</code> 상속을 막기 위해서 이런 규칙이 필요하다.   
[HTMLElement 인터페이스가 할당 되는 방법](https://html.spec.whatwg.org/multipage/dom.html#elements-in-the-dom)

## Custom Element 사용방법
~~~js
window.customElements.define('my-customtag', class extends HTMLElement {});

<my-customtag></my-customtag>
~~~
제일 간단한 코드로 Custom Element를 등록하는 방법이다. window의 CustomElementRegistry에 Custom Tag와 주어진 Class를 묶어주는 역활이다. 아직까지는 별다른 로직도 없고 별 의미는 없지만 <code>&lt;my-customtag&gt;</code>는 HTML에서 사용 될 수 있는 Custom Element가 된것이다. 

### 그럼 HTMLElement 라는게 몰까?
모든 엘리먼트는 HTMLElement의 자식이다. 따라서 HTMLElement의 프로퍼티를 똑같이 가지고 있고, 동시에 엘리먼트의 성격에 따라서 자신만의 프로퍼티를 가지고 있는데 이것은 엘리먼트의 성격에 따라서 달라진다. HTMLElement는 <code>Element</code>의 자식이고 Element는 <code>Node</code>의 자식입니다. Node는 Object의 자식입니다. 이러한 관계를 <code>DOM Tree</code>라고 합니다.   

![Full-width image]({{'/assets/img/blog/190710/domtree.png'| relative_url}}){:.lead data-width="100%"}
출처: 생활코딩
{:.figure}

## Custom Element lifeCycle
그렇다면 <code>Custom Element</code>에 대한 <code>Life Cycle</code>을 한번 알아보자.   

~~~js
class MyCustomElement extends HTMLElement {
  constructor() {
    super();
  }
  
  connectedCallback() {
    ...
  }
  
  disconnectedCallback() {
    ...
  }
  
  attributeChangedCallback(name, oldValue, newValue) {
    ...
  }
  
  adoptedCallback() {
    ...
  }
}
~~~

이렇게 기본적으로 Life Cycle을 가지고 있다. 여기에 몇가지 추가해서 하나하나씩 알아보자.
### Construtor
<code>ES6 Class</code>문법을 알고 있는 사람이라면 대부분 어떤 기능인지 알것이다. 다만 중요한 포인트는 <code>HTMLElement</code>를 상속받아야 하기 때문에 <code>super()</code>가 항상 **Constructor의 최상단에 선언이 되야만 한다.** 이부분이 제일 중요포인트.   
또한 Custom element를 생성할때는 constructor안에 리소스를 렌더링 하거나 가져오는 작업을 피해야 하고, 이런 작업들은 <code>connectedCallback</code>으로 불러와야 한다. 
- 생성자 첫번째 구문 <code>super();</code>는 구성 요소가 올바른 프로토타입 체이닝과 확장된 클래스의 모든 속성과 메소드를 상속받도록 **인자가 없는 호출이여야 된다.** 
- <code>document.write</code> 또는 <code>document.open</code>등은 **사용 불가**하다.

### connectedCallback
요소가 돔에 추가되면 connectedCallback 메소드가 트리거 되며, 그 순간부터는 DOM에서 사용할 수 있으며, 설정코드를 실행하거나 템플릿 랜더링을 할수 있다. **조금 쉽게 보자면 Ajax의 Success Callback이라고 보면 될듯 하다**.   
찾다가 보니 같은 페이지안에 같은 요소가 다른곳에 쓰이는 경우에 관한 팁이 있다. 한마디로 2번 호출하는건데 아직은 정확하게 모르지만 나중을 위해 기록해둔다.   
~~~js
class MyCustomElement extends HTMLElement {
  ...
  
  connectedCallback() {
    console.log('connected');
  }
}

customElements.define('my-custom-element', MyCustomElement);

const myCustomElement = new MyCustomElement();

document.body.appendChild(myCustomElement);
document.body.appendChild(myCustomElement);

// result:
// 'connected'
// 'connected'
~~~
<code>Node.appendChild()</code>요소가 현재 위치에서 connectedCallback 메서드를 트리거하는 새 위치로 이동 하기 때문에 **두 번 호출된다** . 이 동작이 문제가되면 <code>requestAnimationFrame</code>콜백에 **작업을 연기하고 이후의 모든 프레임 connectedCallback호출을 취소 할 수 있다** .
- requestAnimationFrame이 무엇인지 부터 나중에 한번 찾아봐야겠따......일단 기록!

### disconnectedCallback
이 함수는 요소가 DOM에서 제거 될때 트리거되며 <code>.remove()</code>함수로 지우는 것과 같은 효과를 가진다고 보면 될듯하다.

### attributeChangedCallback(attrName, oldVal, newVal)
CustomElement에 Data를 전달하는 방법중 한가지는 Attribute를 이용하는것이며 아래와 같이 지정할수 있다.
~~~html
<my-custom-element 
  prop1="foo" 
  prop2="bar" 
  prop3="baz">
</my-custom-element>
~~~
그리고 이 attribute에 접근하는 방식은 아래와 같다.
~~~js
class MyCustomElement extends HTMLElement {
  ...
  
  connectedCallback() {
    const prop1 = this.getAttribute('prop1'); // foo
    const prop2 = this.getAttribute('prop2'); // bar
    const prop3 = this.getAttribute('prop3'); // baz
  }
}
~~~
하지만 이 속성은 connectedCallback이 일어날때만 읽을수 있기때문에 data를 update하려면 DOM을 제거한후에 다시 연결해줘야 되는 단점이 있다. 이 단점을 보완하기 위해 LifeCycle에 들어가진 않지만 <code>Observer pattern</code>인 <code>static get observedAttributes</code>라는 함수가 있다. 

### static get observedAttributes
이 함수는 모든 종류의 속성을 관찰하는게 아닌 이 함수안에 배열로 나열된 인자들만 관찰을 한다.  인자로는 
- **name** : attribute name
- **oldVal** : attribute의 업데이트 전 값
- **newVal** : attribute의 업데이트 후 변경된 값

속성을 관찰하고 업데이트 방법은 아래와 같은 예시로 볼수 있다.
~~~js
class MyCustomElement extends HTMLElement {
  ...
  
  static get observedAttributes() {
    return ['prop1', 'prop2', 'prop3'];
  }
  
  attributeChangedCallback(name, oldValue, newValue) {
    console.log(`${name}'s value has been changed from ${oldValue} to ${newValue}`);
  }
}
~~~

### adoptCallback
이 요소는 어디에서 찾아봐도 크게 알 필요도 없다고 하고 중요하게 다뤄놓은 블로그를 찾지못했다. 원문을 읽어봐도 음.. 잘 모르겠다. 일단 이런게 있다라고 알아만 두고 추후에 더 추가해야겠다. 

## 마치며
일단 Custom Element에 대해 알아봤는데 이거 하나만 가지고는 큰 힘을 발휘 못한다. 이것만 보면 그냥 딸랑 Tag 이름만 바꾼거 뿐이라서 굳이 왜 이렇게해?? 그냥 DIV쓰면 되지라고 생각하는 사람도 있을것이다. 이부분은 Web Component의 구성요소들과 합쳐져야지만 진짜 큰힘이 발휘되는듯 하다. 다음번에는 <code>Shadow root</code>에 대해 한번 포스팅 해봐야겟다.

