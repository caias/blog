---
layout: post
title: will-change
categories: frontend
tags: Javascript css will-change
comments: true
---

> will-change 속성
{:.lead}
* list
{:toc}

## TLDR
will-change는 실제로 변화시키는 속성과 실제로 변화가 발생할 엘리먼트에 설정하라. 그리고 변화가 종료되면 삭제하라.

## 개요
<code>reflow</code>와 <code>repaint</code>를 알아보다가 최적화 방법중에 한가지 방법인 CSS3 속성중에 <code>will change</code>라는 속성이 최적화 스킬중에 한가지라는 글을 봤고 안그래도 예전에 UI개발자로 있을때 touch slide 라이브러리인 <code>slick.js</code>를 쓰다가 슬라이드가 마지막 인덱스에서 처음으로 돌아갈때 <code>깜빡임 현상이슈</code>가 있었을때 이 속성으로 해결을 했던 적이 있어서 나중에 찾아봐야지 하다가 이번기회에 한번 정리 해보고자 한다.

{% include components/youtube.html id='-62uPWUxgcg' %}

## animate vs transform
아마 Mobile과 PC 양쪽 애니메이션 관련 기능을 구현해본 사람들중 **모바일에서는 animate를 쓰는거보다 transform을 쓰는것이 더 부드럽게 움직인다.**라는 말을 한번쯤을 들어봤을것이다. 나도 자주 들어봤던 얘기였고 그냥 그런가보다 하고 모바일에서는 <code>transform</code>기능을 우선순위로 쓰기 시작했었다.   
이전 포스트에서 <code>reflow</code>에 대해 보셨거나 또는 이 용어에 대해 알고 있는 사람들이라면 <code>animate</code>랑 보통 제일 많이 같이 쓰는 속성들은 <code>position</code>속성들인 left, top, right, bottom값들이 있을것이다. 이 속성들은 전부 <code>reflow</code>를 유발하는 대표적인 속성들이기 때문에 모두가 레이아웃 동작을 발생하며 이 동작자체의 비용이 비싸다는 뜻이기 때문이다. 반대로 <code>transform</code>관련 동작들은 단일 레이어 안에서 실행 되기때문에 레이아웃의 비용을 발생하지 않고 저비용으로 좀더 부드러운 애니메이션을 발생한다. 여기에다가 GPU에서 처리를 분담할수 있게 <code>GPU가속</code>을 시켜 성능을 최대한으로 끌어올린 다면, **모바일 디바이스에서 주로 발생하는 CPU의 부하를 줄일수 있다.** 

## GPU가속의 조건
<code>GPU가속</code>의 조건은 단순하다. <code>transform</code>을 예로 들자면, 같은 transform이라도 <code>2D</code>와 <code>3D</code>로 나뉜다. **가속 조건은 3D일때만 일어난다.**   
또한 CSS animation, transform, transition 속성에 자동으로 GPU 가속이 활성화 되지 않는다. 

**NOTE**: Opacity속성은 reflow와 repaint의 개념이 아닌 **composite**(합성)의 개념이며, 기본적으로 GPU가속을 사용하는 몇 안되는 속성중의 하나이다. 
{:.message}

## 예전방식의 null transform Hack
기존에는 브라우져를 속여 강제로 animation이나 transform을 시키기 위해 썼던 방식이 있다. 3D가 필요하지 않은 엘리먼트에 단순히 3D변형을 지시하여 단일 레이어로 만든후에 렌더링 처리를 고속화하는 꼼수? hack? 개념인것이다.   
~~~css
transform: translateZ(0);
transform: translate3d(0, 0, 0);
~~~
이러한 방법으로 GPU가속을 시키면 <code>composite layer</code>(합성레이어)라는 단일 레이어가 GPU에 의해 합성된다. 다만 출력을 빠르게 하는만큼 너무 많이 쓰게 되면 메모리 사용량이 커지면서 그만큼 악영향을 미치게 되고 모바일 기기에서는 더 두드러지게 나타나기 때문에 신중하게 써야된다.   
이 시점에서 새로운 composite layer로 엘리먼트를 분리 하기 위한 비용이 발생하고 애니메이션의 몇 분의 1초 단위의 눈에 띄는 pending현상이 발생하게 되며 이 현상이 바로 깜빡임으로 이어지게 된다.

## will-change란?
이제부터 CSS3에 새롭게 등장한 will-change속성에 대해 알아보겠다. 쉽게 얘기하자면 **브라우져한테 나 이렇게 엘리먼트를 동작 또는 변형 시킬거야** 라고 미리 알려주는 속성이 바로 <code>will-change</code>속성이다. 이것을 사용하면 변경이 시작되기전에 페이지 출력에 악영향을 줄수 있는 처리 비용을 줄일수 있고, 효율적으로 엘리먼트의 변경 또는 렌더링을 할수 있으며, 부드러운 화면 처리가 가능하게 된다.   

브라우져 지원 여부는 [Caniuse](https://caniuse.com/#search=will-change) 여기에서 확인 가능하다.
{:.faded}

### will-change 기본속성
~~~css
will-change: auto;
will-change: scroll-position;
will-change: contents;
will-change: transform;
will-change: top, left;
~~~
- auto: 기본값으로 브라우저는 별다른 최적화를 실행하지 않는다.
- scroll-position: 스크롤할때 엘리먼트 위치가 변경될 것을 알려준다. 한번에 많응 양을 스크롤 하거나 빠른 스크롤이 필요한 경우에 유용함.
- contents: 엘리먼트의 컨텐츠가 변경될것을 미리 알려주며, 브라우저는 보통 엘리먼트 렌더링 결과를 캐싱하지만, 이 기능을 사용하게 되면 캐시를 하지않고 변경될때마다 처음부터 랜더링을 하게 됩니다. 
- transform: 말 그대로 transform관련 변경 기능들을 미리 브라우져에게 알려준다. 
- custom-ident: 변경하고 싶은 속성을 사용할 수 있지만, 현재 지원 여부는 (opacity, transform, top, right, bottom, left)만 적용된다. 

### will-change 사용시 주의할 점
- 너무 많은 속성과 요소에 will-change 속성을 사용하지 말자.
~~~css
// Bad Case
*,
*:before,
*:after {
    will-change: top, left, bottom, right, transform, opacity;
}
~~~
이 속성을 너무 남발하게 되면 무리한 자원을 소모하기 때문에 오히려 성능이 저해될수 있다. 

- 애니메이션 동작이 끝난 후 기본상태로 항상 reset시키자.

기본적으로 브라우져가 최적화를 시도하면 일반적으로 비용이 발생하고, 필요가 없다면 다시 원래되로 되돌아 온다. 하지만 <code>will-change</code>속성의 경우는 **최적화를 길게 유지하기 때문에 엘리먼트가 변경이 종료되면 반드시 삭제 하여 메모리를 회수해야 한다**. 

~~~js
// 간단한 예제
// 클릭할 때 애니메이션을 재생할 엘리먼트를 선택합니다.
const el = document.getElementById('element');

// 엘리먼트에 마우스 커서가 올라가면 will-change를 설정합니다.
el.addEventListener('mouseenter', hintBrowser);
el.addEventListener('animationEnd', removeHint);

function hintBrowser() {
	// 애니메이션의 키프레임 블럭을 최적화할 수 있는 속성을 사용합니다.
	this.style.willChange = 'transform, opacity';
}

function removeHint() {
	this.style.willChange = 'auto';
}
~~~

- slide의 auto slide같이 지속적으로 변화가 일어나야 되는 부분에 한해서는 오히려 reset없이 CSS에 선언해주는것이 더 좋다.

~~~css
.slide {
  will-change: transform;
}
~~~

## 마치며..
<code>will-change</code>가 CSS속성이긴 하지만, 현재로서는 모바일 관련 Front-end개발에서도 분명히 쓰이는 상황이 많을거 같다. 물론 적당한 상황과 적당한 요소에 써야지 정말 큰 힘을 발휘하겠지만, 이걸 판단 할수 있는 능력은 실제로 써보고 실무에 도입해보면서 경험치를 통해 판단할수 있는 능력을 키워야겠따.  

