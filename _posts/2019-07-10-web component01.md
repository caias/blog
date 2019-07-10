---
layout: post
title: web component
categories: frontend
tags: Javascript web-component
comments: true
---

> web components
{:.lead}
* list
{:toc}

## 개요
<code>Web Component</code>라는 단어로 검색을 해보니 naver.d2에서는 2012년도부터 정리가 됬던 글이 보였다.. 아 겁나 오래된거구나... 자괴감이 든다 이렇게 뒤쳐져있다니...    
그 당시의 신기술이였기때문에 지원도 미흡하다고 나와있었고 지금도 찾아보니 IE를 제외한 나머지는 대부분 지원하고 <code>Polymer</code>을 사용한다면 IE+11/EDGE 브라우져 까지는 지원을 하는듯 하며, 대부분의 모바일에서는 사용 가능할듯 하다.   

**NOTE**: Polymer란 web Component를 polyfill 해주는 framework이다.
{:.message}

<code>Web Component</code>라는 단어는 모르고 이 블로그의 원본 테마인 hydejack테마를 쓰고 소스를 파악하던중에 shadow root라는 부분이 Dom에 자리잡고 있었고 생전 처음보는 태그이름으로 쓰이고 있는 부분이 있어서 궁굼하던 참에 한번 찾아보고 정리해야겠다 싶었다. 현재 이 블로그를 개발자 도구로 element를 확인해보면 다음과 같은 부분이 보일것이다.    

~~~html
#shadow-root(open)
<hy-drawer class="loaded" align="left" threshold="10" touch-events="" prevent-default="" mouse-events="" range="0,508">
  <header id="_sidebar" class="sidebar" role="banner" style="transform: translate(730px, 0px);">
  ...
  </header>
</hy-drawer>
~~~

## Web Component란?
<code>Web Component</code>의 핵심 개념은 최근 framework 3대장인 <code>React</code>. <code>Angular</code>, <code>Vue</code>와 같은 프레임 워크 구성 요소와 비슷하다. 여기에서 가장 큰 차이점은 **framework에 의존하지 않고 브라우져가 기본적으로 제공하는 기술을 활용한 component이며, framework에 의존성이 1도 존재하지 않는 기술이다**. (그러나 어떠한 framework와도 결합 가능하며 webpack에서도 loader가 나와서 같이 사용 가능하다.)

### Web Component를 왜 써야 될까?
현재 Hot하다는 framework들은 전부 훌륭하고 각각의 장단점 들이 있으며, IE하위버전을 운영하는 회사입장에서는 못쓰는 framework들도 많다. 또한 예를들어 Angular컴포넌트 안에서 React나 Vue를 정말 쉽게 가져다 쓸수 있는 상호운용성이 없다고 생각하다. 반대의 경우도 물론이고, 그 framework들은 그들의 생태계 안에서만 훌륭한게 아닐까 하는 생각이 든다.

### Web Component의 장점
- framework의 종속성이 없고 네이티브 엘리먼트로 동작하기 때문에 성능이 좋다.
- Web Component는 framework을 넘어서 다른 기술 스택의 프로젝트 또는 다른 framework에서도 아무 제약없이 동작한다.
- 컴포넌트 자체가 framework에 제약이 없기 때문에 더 긴 수명을 갖게 되고 새 기술에 맞춰서 새로 작성할 필요가 줄어든다.
- 의존성이 없기 때문에 도입에 대한 장벽이 상당히 낮고 효율은 더 높일수 있다.

## Web Component의 구성 요소
Web Component의 구성요소에는 크게 3가지가 있다. 한가지 한가지씩 따로 포스팅을 해야될 정도로 양이 꽤 많으니 일단은 가볍게 한번씩 정리해보고 포스팅을 나눠서 해봐야겟다.   

- <code>HTML import</code>방식까지 원래는 4가지였으나 2017년도에 polymer3.0을 release하면서 이 방식을 포기했고 <code>ES6 MODULE</code>로 대체되면서 **webpack의 생태계 안으로 들어가게 됬다.**
- polymer3.0 때부터 Tempalte Element도 Template literals로 바뀌게 되었지만 이부분은 포기라기보다는 겸용할수 있기 때문에 아직까지 [living standard](https://html.spec.whatwg.org/multipage/scripting.html#the-template-element)로 명시되있다.

### Custom Element
<code>Custom Element</code>는 HTML의 엘리먼트를 사용자가 문서에서 확장하여 사용할 수 있도록 기능을 제공한다. 모든 웹 개발자들이 새로운 타입의 <code>HTML element</code>를 정의할 수 있도록 하여 본질적으로 최종 개발자들이 Web Component를 쉽게 사용하기 위한 방법을 제공하며 Web Component에서 가장 중요한 API라고 할수 있다.   
아래는 Custom Element를 사용하여 적절한 이름을 가지게 하는 Markup의 예시이다. 보기좋잔소~
~~~html
<div class="user-profile">
    <div class="layout card small">...</div>
</div>
<user-profile>
    <card-layout type="small">...</card-layout>
</user-profile>
~~~

### Shadow DOM
<code>shadow DOM</code>을 이용하여 컨텐츠와 UI를 완벽하게 분리할 수 있다. shadow DOM의 가장 중요한 기능이라고 한다면 모든 <code>scope</code>자체를 <code>shadow root</code>안에서만 작동할 수 있게 해주는 기능일것이다. html tag / css / js variable , selector 등등 global하게 쓰이고 있거나 혹시 어디서 쓰이고 있지 않나? side effect는 없을까? 하는 고민자체를 한방에 날려버릴 정도로 모든 scope는 이 component안에서 독립적으로 수행이 가능하게 만들어주는 API다. (java의 private속성이나 javascript의 closure랑 비슷하다고 봐도 무방하려나..?)   
아래는 **1개의 component를 shadow DOM Tree로 캡슐화 하여 외부와는 완전히 차단시킨 독립적인 랜더링을 하는 파일 구조의 예시이다.**
~~~html
#shadow-root(open)
<style>
  {...}
</style>
<my-element>lorem ipsum</my-element>
<script>
  customElements.define('my-element', class extends HTMLElement {
    {...}
  }
</script>
~~~

### HTML template
&lt;template&gt; 태그로 이루어진 이 tag는 runtime에만 활성화되는 복제 가능한 마크업 템플릿이라고 볼수 있다.   
[WhatWG HTML Templates](https://html.spec.whatwg.org/multipage/scripting.html#the-template-element) 표준규격은 템플릿을 위한 표준적인 DOM 기반의 접근방법을 기술하는 새로운 엘리먼트인 `<template>`을 정의 하였으며 <u>템플릿 컨텐츠는 사용시까지 비활성화되어 렌더링되지 않고 템플릿 안의 스크립트나 DOM이 다른 곳에 영향을 미치는 부작용이 없다.</u> 선언/재활용 역시 마찬가지이며 또한 적용 위치 역시 자유롭기 때문에 많은 부분에서 활용이 가능하다.

## Web Component 및 Framework
[React](https://www.sitepen.com/blog/2017/08/08/wrapping-web-components-with-react/) 와 [Angular](https://www.sitepen.com/blog/2017/09/14/using-web-components-with-angular/) 와 같은 프레임 워크에서 네이티브 Web Components를 사용하는 방법에 대한 설명이 있지만 두 가지 모두 고유성과 관련된주의 사항이 명시되있다. [Custom Elements Everywhere](https://custom-elements-everywhere.com/) 여기에서 다양한 프레임 워크가 사용자 지정 요소 (웹 구성 요소의 핵심 요소)와 얼마나 잘 통합되어 있는지를 확인할수 있으니 한번쯤 살펴보면 좋을듯 하다.

## 접근성은 그럼?
아직까지 Web Component를 직접 손으로 쳐보면서 테스트 해본것도 아니고 shadow DOM과 Custom Element을 썼을때 포커스 기능이나 접근성 측면에서 어떻게 동작하는지도 알수가 없지만, 궁굼해서 한번 찾아봤다. 이부분에 대해서 일단은 링크로만 대체 해놓고 추후에 직접 테스트 해보고 포스팅으로 하나 더 추가해 놓던가 해야겠다.   
[accessibility-for-custom-elements](https://robdodson.me/the-future-of-accessibility-for-custom-elements/) - Custom Element에 대한 접근성의 미래

## 마치며
Web Component의 구성요소에 대해서 각각의 포스팅을 해야될 정도의 분량인듯 해서 하나씩 포스팅 해 볼 생각이다. 나온지는 엄청 오래지났고 기간동안 바뀐점들도 파악하면서 포스트를 쓰려니 오래걸리긴 하지만, 현재 이 기술에 꽂혀있고 간단하게라도 실제로 코드를 작성하면서 구성요소에 대해 조금더 자세히 추가된 포스팅을 올려볼 예정이다. 



