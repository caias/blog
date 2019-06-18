---
layout: post
title: 논리연산자 활용
categories: devlog
tags: Javascript devlog
comments: true
---

> 논리연산자
{:.lead}
* list
{:toc}

## 논리연산자란?

|연산자|구문|설명|
|:---|:---|:---|
|AND(&&)|expr1 && expr2| expr1을 true로 변환할 수 있는 경우 expr2을 반환하고, 그렇지 않으면 expr1을 반환합니다.|
|OR (<code>||</code>)|<code>expr1 || expr2</code>| expr1을 true로 변환할 수 있으면 expr1을 반환하고, 그렇지 않으면 expr2를 반환합니다.|
|NOT (!)|!expr| 단일 피연산자를 true로 변환할 수 있으면 false를 반환합니다. 그렇지 않으면 true를 반환합니다. |
{:.stretch-table}
출처:[MDN](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/%EB%85%BC%EB%A6%AC_%EC%97%B0%EC%82%B0%EC%9E%90(Logical_Operators)) - 논리연산자
{:.faded}

## 개요

논리연산자는 보통 Boolean값과 함께 쓰이며, Boolean값을 반환한다고만 알고있었고 단순히 1차원 적으로 논리연산자란 함수/변수/Value값에서 return된 Boolean값을 비교 해서 Boolean값을 Return한다 라고만 생각했다. 

그러나 <code>&&</code> 와 <code>||</code> 연산자는 그냥 **피연산자중 하나의 값을 반환**하는것이었다. 

### 개선 예시1
실무중에 flag값이 true일때만 특정 함수를 실행하게 해야만 했던 상황이 있었고 예시로 아래와 비슷한 코드를 작성했다가 코드리뷰중에 논리연산자를 이용하여 축약형으로 쓸 수 있는 조언을 반영했다.
~~~js
let flag = true;

func() {
  ...
}

// Bad
if(flag === true) {
  func();
}

// Good
flag && func();
~~~

### 개선 예시2
API호출후에 받는 Data값이 Null 인경우를 체크해야 하는 상황이였다.  
Null로 호출 되는 경우에는 Default Value값을 대신 넣어줘야 했고,
이부분 역시 곧이 곧대로 그냥 Null값만 체크를 하는 소스를 작성했었다. 
~~~js
//Data Example
Data: {
  a: 'null',
  b: foo,
}
// Bad
const dataCheck = Data.a === 'null' ? 'default value' : data.a;

// Good
const datacheck = data.a || 'default value';
~~~

이 부분은 특정 Null값만 체크를 하는것 보다는 False로 체크 되는 부분을 일괄적으로 체크할수 있는 <code>||</code> 연산자를 이용하여 축약했다. Data의 값이 Null이 아니라 다른 False값으로 변경이 되도 수정할 일이 없어서 이런 방식이 더 좋은 듯 하다.

그렇다면, 추가적으로 기본 Boolean값이 False값을 갖는 것들이 어떤것들이 있는지 알아봤다.

* NaN
* Null
* 0
* '' , [], {} (비어있는 String, Array, Object)
* Undefined