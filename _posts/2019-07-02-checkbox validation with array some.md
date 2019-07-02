---
layout: post
title: checkbox validation with array some method
categories: devlog
tags: Javascript devlog array some es6
comments: true
---

> Array method 중 some을 이용한 checkbox validation
{:.lead}
* list
{:toc}

## 개요
회원탈퇴 쪽의 수정작업을 맡았다가 checkbox validation 관련 기능을 개발하게 되었다. 2개의 checkbox의 체크 여부와 체크가 안되있다면 그에 맞는 에러 메시지를 알럿으로 띄우는 기능이다. 여기에서 받은 코드 리뷰중에 Array method중에 ES6에 추가된 <code>some</code>을 이용해서 한번 해보는게 어떠냐 라는 제안을 받고 이용해 봤다. <code>some</code>이나 <code>every</code> method는 개인적으로 사용 빈도가 낮았었는데 이번 기회에 한번 써보고 기록하고자 한다.    

html파일은 예시로 이런 식으로 되있으며, <code>custom checkbox</code>라서 label에 on클래스 활성화 클래스가 붙는 경우다. 
~~~html
<input type="checkbox" data-input="checkbox" class="blind" id="a">
<label for="a" class="chk_box"><span class="ico"></span>진짜 탈퇴?</label>
<input type="checkbox" data-input="checkbox" class="blind" id="b">
<label for="b" class="chk_box" ><span class="ico"></span>레알 탈퇴?</label>
~~~

### 최초 작성
~~~js
const $quitCheckbox = $container.find('[data-input]');
$quitCheckbox.each(function() {
  const isChecked = $(this).next().hasClass('on');
  if(!isChecked) {
    return false;
  }
});
~~~

일단 처음에는 <code>forEach</code>를 쓰려고 했지만, forEach에서는 <code>break</code> 또는 <code>return false</code>가 안되고 계속 <code>continue</code>이기 때문에 패스하고 <code>each</code>로 사용을 했다.

### 코드리뷰 조언
일단 <code>custom checkbox</code> 또는 <code>custom radiobutton</code>같은 경우에는 체크를 할때 class가 아닌 실제로 checkbox가 check 되었는지를 확인 하라였다. 웹쪽 관련 직업 종자사 들이나 개발자 도구를 알고 잇는 사람은 체크박스를 체크 안하고 임의로 클래스를 수동으로 넣어놓고 진행을 하게되면 validation을 통과시켜버릴수 있는 여지가 있기 때문에 좋은 방법이 아니다. 하여 isChceked 라는 조건 변수를 변경 하였다.   
또한 <code>each</code>를 쓰는게 틀린건 아니지만 이번 기회에 <code>some</code> method를 한번 써보라는 권유로 시도해보았다.

### 유사배열
~~~js
const $quitCheckbox = $container.find('[data-input]');
~~~
일단 <code>$quitCheckbox</code>라는 변수에 <code>data-input</code>이라는 data-attritute를 가진 노드를 찾아서 할당을 했다. 이부분을 콘솔로 찍어보면 배열인거처럼 나오지만 정확하게는 배열이 아니다. 이런 부분을 유사배열이라고 불리우는듯 하다.    
이 유사배열을 가지고 바로 <code>quitCheckbox.some(() => {})</code> method를 실행하면 에러가 난다.    
왜? 배열이 아닌데 배열 메소드를 쓰니까!   
그렇다면 유사배열을 배열로 바꿔보는 것 부터 알아보자.   

1. array의 prototype을 빌려쓰게 되는 방식
~~~js
Array.prototype.some.call($quitCheckbox, () => {});
~~~
2. ES6 기능중에 Spread Operator 사용
~~~js
const arr = [...$quitCheckbox];
~~~
3. ES6 기능의 from
~~~js
const arr = Array.from($quitCheckbox);
~~~

이 정도가 있는듯 하며, 굳이 <code>for</code>문을 돌리고 <code>push</code>해서 새로 창조해 내는 또는 유사한 방법들은 굳이 적고싶지는 않다. 

### 코드리뷰 반영
코드리뷰의 요건은 어쨌든 2가지이다.   
1. input check는 class가 아닌 실제로 체크 되있는지를 판별할것.
2. 이런 기회에 some method를 써보는것

하여 대략적으로 하단과 같이 개선해 보았다. 

~~~js
function Validation() {
  const $quitCheckbox = $container.find('[data-input]');
  const checkArr = Array.from($quitCheckbox);

  return checkArr.some(function (item) { // 1
    const isValid = $(item).prop('checked'); // 2

    if (!isValid) {
      // 알럿을 띄우시게~~
      return true; // 3
    }
  });
}
~~~

1. 1번 주석을 달은 이부분 자체를 return시키는 이유는 바로 false를 반환 받기 위함이다. return을 빼버리면 계속 true만 반환된다. 
2. 코드 리뷰의 조언을 받은 대로 class check 가 아닌 실제 input이 check되었는지를 확인할수 있도록 개선한 부분이다. 
3. 다른 함수들과는 다르게 return false가 아닌 <code>return true</code>가 break로 쓰이며, **return true가 되는 즉시 true를 반환 하고 즉시 함수가 중지된다.**

> some method를 쓰면서 제일 중요한 포인트는 바로 **return true가 Break**라는 것이다. 
{:.lead}


## 마치며
some 메소드를 대략적으로 알고는 있었지만, 이번 기회에 좀더 명쾌하게 어떤상황에 써야 되는지 감이 온거 같다. each문을 쓰는게 틀린 방법은 아니지만, 이런 방법도 있다라는 점과 이렇게 직접 해봐야지 some 메소드를 써보지 언제 써보겠냐... 라는 생각이 든다. 역시 코드리뷰로 많이 지적당하고 까여봐야지 내 실력과 경험치가 느는거 같다. 