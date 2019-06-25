---
layout: post
title: template-literals
categories: devlog
tags: Javascript devlog es6 template-literals
comments: true
---

> Template-literals
{:.lead}
* list
{:toc}

## Template-literals란?
Template-literals란 javascript에서 문자열을 입력하는 방식중의 하나이고 <code>Back Tick</code>이라는 기호와 함께 정의한다.

## 기존 사용 예시
ES6 구문중에 template literals방식이 없었을 경우에는 string과 변수명을 합치는 방식에서 다음과 같은 방식을 썼었다.
~~~js
const language = 'Frontend';
const text = 'Im in' + language + 'team';
console.log(text) // Im in Frontendteam
~~~

위와 같이 문자열에 특정 변수의 값을 같이 사용하려면 <code>+</code>를 이용하여 문자열 중간에 변수를 연결시켜줘야만 가능했다.   
하지만 ES6 Tempalte literals함수를 쓰게되면은 다음과 같이 더 축약해서 쓸수 있다.  

~~~js
const language = 'Frontend';
const text = 'Im in ${language}team';
console.log(text) // Im in Frontendteam
~~~

## 개요
이렇게 Template literals방식을 string과 연계해서 쓰는거야 알고있었지만 코드리뷰중에 받았던 지적을 통해 활용 가능한 예시 소스를 남겨두고자 한다. 

## 상황
보통 UI 핸들링을 할때 <code>toggleClass('on')</code>을 쓰는 경우 보다는 on class를 가 있는지를 확인한 다음에 특정 function을 실행시키는 일이 많다. 이부분에 template literals을 추가해보자.

### 최초 소스
간략하게 toggle방식으로 on 클래스가 없을때만 실행해야 되는 함수가 있다고 가정을 한다면 아래와 같은 방식의 예이다.
~~~js
const isActive = $(this).hasClass('on');
if (isActive) {
    $(this).removeClass('on');
} else {
    $(this).addClass('on');
    somefunction();
}
~~~

### 개선 예시
<code>isAcitve</code>라는 변수가 <code>true</code>이면 on클래스를 삭제 <code>false</code>면 on클래스 추가를 하는 toggle방식이며 <code>isActive</code>가 <code>false</code>일때만 함수가 실행한다. 이부분은 지난 포스트에 썼던 논리연산자 부분과 같이 쓰면 좀더 소스를 줄일수있다.
~~~js
const isActive = $(this).hasClass('on');
const type = isActive? 'remove' : 'add';
$(this)[`${type}Class`]('on');
!isActive && somefunction();
~~~

## 마치며
위와 같은 방식은 한가자의 예시일 뿐이긴 하나, 소스를 한줄 줄이는건 굉장히 어려운 일이라고 생각한다. 이런 예시를 토대로 응용한다면은 분명 소스를 더 줄일수 있는 방법들이 많아질 것이라고 보며, 누군가한테는 별거 아님 팁일지 몰라도 이 포스트를 보는사람중에 한명이라도 도움이 된다면 다행일듯 하다.