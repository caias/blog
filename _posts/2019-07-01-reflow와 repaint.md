---
layout: post
title: reflow 와 repaint
categories: frontend
tags: Javascript css html reflow repaint
comments: true
---

> Template-literals
{:.lead}
* list
{:toc}

**NOTE**: Frontend개발 관점에서 reflow와 repaint의 차이점과 최적화 하는 방법을 한번 알아보자. 양쪽 관점에서 어떤 부분이 성능을 떨어뜨리게 되고, 어떻게 해야 성능 최적화를 할수 있는지에 대한 고찰이다. 
{:.message}

## Reflow란?
- 생성된 DOM 노드의 레이아웃 수치(너비, 높이, 위치 등 수치관련 전부) 변경 시 영향 받는 모든 노드(자신, 자식, 부모, 조상 모두다) 수치를 다시 계산하여(Recalculate), 렌더 트리를 재 생성 하는 과정을 일컫는 용어이다.

### Reflow에 영향을 미치는 속성

|width|height|
|padding|margin|
|display|border-width|
|border|top|
|position|font-size|
|float|text-align|
|overflow-y|font-weight|
|overflow|left|
|font-family|line-height|
|vertical-align|right|
|clear|white-space|
|bottom|min-height|
{:.stretch-table}
등이 있으며, 기입이 안되있더라도 수치 관련된 속성들은 전부라고 봐도 무방하다.

## repaint란?
reflow과정이 끝난 후 재생성된 렌더 트리를 다시 그리는 작업으로 수치와는 상관없는 가시적인 요소로 인한 reflow과정이 생략된 repaint 작업만 수행하는 과정을 일컫는 용어이다.

### repaint에 영향을 미치는 속성

|color|border-style|
|visibility|background|
|text-decoration|background-image|
|background-position|background-repeat|
|outline-color|outline|
|outline-style|border-radius|
|outline-width|box-shadow|
|background-size||
{:.stretch-table}

## reflow 와 repaint를 피하거나 최소화 하기 위한 최적화 방법
reflow와 repaint가 어떤것인지 알아봤으니 최적화 하기 위한 방법들에 대해 알아보자.    
Frontend 개발자 입장에서 고려할 점과 UI개발자 입장에서 고려할점이 조금씩 다르지만, Front-end개발 입장에서만 기록하고자 한다.

### class를 핸들링해서 랜더링이 변하게 될 경우에는 DOM구조상 최하단에 위치한 노드에 추가해야된다.

class가 변할때는 reflow를 막을 수 없지만 최소화 해야한다. first/second/change 아이디에 각각 addClass를 줘서 width를 50%로 만든다는 가정으로 보자.

~~~html
<html>
<body>
    <div id="first">
 
        <div>
            <!-- div 약 100개 중첩 -->
        </div>
        <div>
            <!-- div 약 100개 중첩 -->
        </div>
        <div id="second">
          <!-- div 약 100개 중첩 -->
          <div>
            <div id="change">
                블록
            </div>
          </div>
        </div>
    </div>
</body>
</html>
~~~

~~~scss
  #change {
      width: 100%;
  }
  #first.addclass{
      width: 50%;
  }
  #second.addclass {
      width: 50%;
  }
  #change.addclass {
      width: 50%;
  }
~~~

~~~js
  $('#first, #second, #change').addClass('addclass');
~~~
예시로 이런식의 addClass를 해준다고 하면은 결과는 이렇게 나온다. DOM이 많으면 많을수록, 중첩이 많으면 많을수록 격차는 많이 나게 될것이며, first와 change의 속도차이는 따지고보면 눈 깜빡할 사이겠지만, 티끌모아 최적화라고 1초도 안되네 라고 보기보다는 약 15배 차이나는 부분에 주목하는게 좋을 듯 하다. 

~~~html
<div id="change">(1.431ms) <div id="second">(1.812ms)  <div id="first">(15.37ms)
~~~

### JS를 통해 style변화를 줘야 될 경우 가급적 한번에 처리
cssText를 이용한 스타일 변경을 하나로 묶어서 reflow 수행을 최소화 한다. 

~~~js
// BAD
div.style.height = '80px';
div.style.backgroundColor = '#00f';
div.style.display = 'inline-block';
div.style.overflow = 'hidden';
div.style.fontSize = '40px';
div.style.color = '#fff';

//GOOD
div.style.cssText = 'backgroound:red;width:200px;height:200px;';
~~~

하지만 제일 좋은 방법은 변경되야될 style을 한군데다가 묶음 활성화 class를 통해 한번에 적용 하는것이 최적화의 방법이다.
~~~scss
.on{
  display:inline-block;
  overflow:hidden;
  font-size:40px;
  color:#fff;
  ...
}
~~~
~~~js
  addClass('on');
~~~

### inline style을 최대한 배제하라.
위의 내용과 비슷한 내용이다. 예를 들면 UI개발자 입장에서 파가 나뉠것이다. 
1. Frontend팀에서 show() / hide() 해주길 바라는 입장
2. 
~~~scss
.block{
  display:block;
}
.none{
  display:none
}
~~~
이런식으로 클래스로 만들어놓는 입장   
결론은 후자가 오히려 reflow 비용을 줄이며 최적화 할수 있는 방법중의 하나라는 뜻이다. 

### 레이어를 분리하라
어찌보면 UI개발자 관점일수도 있다. 우리가 흔히 아는 <code>position</code>관련 속성인 <code>absolute, fixed</code>이 부분이 바로 주변 노드와 상관없이 별도로 렌더링 되는 경우다. 여기에다가 추가적으로 적고자 하는 부분은 바로 <code>transform:translateZ(0)</code>속성 때문이다. 아마 다른 웹사이트의 CSS를 많이 살펴본 사람이라면은 위 코드가 심심치 않게 들어가 있는것을 봤을 것이다. 하지만 저부분은 그냥 노드의 Z축 값으로 0을 주는 무의미한 코드로 왜 굳이 이걸 썼을까? 라는 생각을 할수 있다. 그 이유는 해당 노드를 강제로 레이어로 만들어 버리는 일종의 (Hack)개념으로 현재 널리 쓰이는 방법이기 때문이다. 이 부분은 다음 포스트인 <code>will-change</code>에서 부가적으로 설명을 추가할 예정이다.

## 마치며
reflow 와 repaint라는 용어가 생소해서 정리를 해봤지만, 기본적인 부분도 있고 새로 알게 된 사실도 있는거 같다. 어찌보면 기본중의 기본이기도 하지만, 이런 부분을 알고 염두에 두고 작업하게 되면 조금이라도 퍼포먼스가 좋아지지 않을까 싶다. 