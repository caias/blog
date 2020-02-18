---
layout: post
title: chrome extension으로 bitbucket stash pull-request관련 기능을 만들어보자#1
categories: frontend
tags: chrome-extension
comments: true
---

> chrome extension#01
{:.lead}
* list
{:toc}

## 개요

재밌는 사이드 프로젝트를 할게 없을까 고민해보다 react를 사용해서 <code>chrome extension</code>을 한번 만들어볼까 라는 생각이 들었다.  
chrome extension이라 하면 편의성이 제공되야 하는데 어떤 extension을 만들어야 내 손이 편해질까 생각하다가 메일로 보통 알림이 가지만 **stash bit bucket에서 Pull request를 요청했을때 Reviewer들에게 Slack으로 메시지를 보내서 빠른 코드 리뷰 및 Approve를 유도**해야 겠다 라는 생각이 들었다.
  
extension을 개발하면서 모든 소스관련 전부 다 기록은 못하겠지만, 제작에 필요한 부분들에 대해 회고할겸 하나씩 기록해 보고자 한다.

## extension의 구성요소
<code>chrome extension</code>의 구성요소는 크게 4가지로 나눌수 있다.  
  
1. manifest.json
2. popup.js
3. content.js
4. background.js

### manifest.json
**필수 설정 파일**인 manifest.json파일에 대해서 알아보면서 나머지 구성요소들에 대해서도 간략하게 알아보자.  


~~~json
{
  "name": "Extension",
  "version": "1.0.5",
  "manifest_version": 2,
  "description": "제작 도즈언",
  "browser_action": {
    "default_icon": "icon.png",
    "default_popup": "popup.html"
  },
  "content_scripts": [
    {
      "js": [
        "a.js"
      ],
      "matches": [
        "https://naver.com/*",
      ]
    },
    {
      "js": [
        "b.js"
      ],
      "matches": [
        "https://daum/*",
      ]
    }
  ],
  "background": {
    "scripts": [
      "background.js"
    ]
  },
  "content_security_policy": "script-src 'self' 'unsafe-eval'; object-src 'self'",
  "key": ,
  "permissions": [
    "activeTab",
    "<all_urls>",
    "tabs",
    "cookies",
    "storage",
    "notifications"
  ]
}
~~~

위에 구성요소들을 하나씩 설명해보자면 
- **name** : extension을 등록했을시에 등록되는 name이다.
  
- **version**: 1.0.0 부터 시작하며, extension 업데이트시에는 [semver](https://semver.org/lang/ko/)에 의거하여 꼭 버전을 업데이트 해야지만 업데이트가 가능하다.(등록된 버전과 업데이트버전이 같으면 에러 발생)
  
- **manifest_version**: [chrome extension에서 명시해놓은 version](https://developer.chrome.com/extensions/manifestVersion)으로 이 글을 쓴 날짜 기준으로는 2로 규약이 되있고 이 규약에 맞는 버전을 기입해야 에러가 안난다.
  
- **description**: 제작하는 extension의 부가 설명을 적는 곳이다.
  
- **browser_action**: <code>default_icon</code>은 extension 설치시에 크롬 브라우져 우상단에 나타나는 icon이고 <code>default_action</code>은 extension 아이콘을 눌렀을때 나타나는 팝업에 대한 <code>html파일</code>이다.
  
- **content_scripts**: 배열로 이루어져 있으며, 위의 manifest파일 기준으로는 <code>a.js</code>라는 파일을 https://naver.com으로 시작되는 모든 URL에 사용을 하겠다이며, <code>b.js</code>파일은`https://daum.net으로 시작되는 모든 URL에 사용하겠다 라는 예시이다. wild card(*)패턴은 누구나 다 알고 있을거라 생각한다.
  
- **background**: background.js라는 파일을 크롬 브라우져의 front단이 아닌 background에서 실행하겠다 라고 명시해둔 예시이다. 쉽게 생각하면 <code>web worker</code>랑 비슷한 기능이라고 보면 될거같다.
  
- **content_security_policy**: 이부분은 명시를 해줘야 에러가 안나는 부분이다. 애석하게도 정확하게 왜 필요한지. 없으면 왜 에러가 나는지에 대해서는 정확하게 모르겠다.
  
- **key**: chrome extension 설치가능 기준을 Team단위로 묶고 등록된 이메일에 한해서만 설치하고싶다면 해당 키값이 extension 등록시에 부여된다. <code>privacy 기능</code>이라고 보면 될듯하다.
  
- **permissions**: chrome extension자체 기능들에 대한 권한을 부여하는 곳이다. 위의 기준예시로는 <code>activeTab</code>은 현재 활성화된 브라우져탭에 대한 권한  
<code>all_urls</code>는 모든 url에 대하여 extension을 허용, 특정 url기입도 가능하다.  
<code>tabs</code>는 현재 열려있는 chrome의 모든탭에 대한 권한 허용  
<code>cookies</code>는 extension에 사용되는 cookie에 대한 권한 허용  
<code>storage</code>는 extension에서 사용되는 storage에 대한 권한허용  
(**local storage랑 비슷하면서도 다른 특징들이 있다.**)  
<code>notifications</code>는 chrome browser에 내장되있는 알림팝업에 대한 권한허용

### chrome.storage VS local.storage
기본적으로 storage API기능은 비슷하지만 다른점을 간략하게 설명해보고자 한다.  
1. local.storage는 String만 저장 가능한반면, chrome.storage는 <code>Object단위의 data가 저장 가능</code>하다.
2. local.storage는 동기방식으로 다른 스크립트를 막지만, chrome.storage의 <code>read/write기능은 비동기방식</code>이다.
3. storage.sync기능을 이용해서 크롬 브라우져간의 동기화가 가능하다.

큰 차이점은 이정도가 있으며, 자세한 내용이 궁굼하다면 [chrome storage API 명세](https://developer.chrome.com/apps/storage)를 참고해보자.

## 마치며..
extension을 등록하는 포스트들은 너무 많아서 따로 정리하진 않았다. chrome extension 구성요소들에 대해 간략하게 알아보았고 다음 포스트 부터는 실제 개발 내용을 토대로 정리를 이어가보고자 한다. chrome extension의 API는 Deep하게 들어가면 정말 많지만, 개발에 쓰였던 부분들 위주로 설명을 해보자 한다.