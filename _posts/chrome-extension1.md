---
layout: post
title: chrome extension 을 만들어보자#1
categories: frontend
tags: chrome-extension
comments: true
---

> 크롬 익스텐션 제작기
{:.lead}
* list
{:toc}

## 개요

재밌는 사이드 프로젝트를 할게 없을까 고민해보다 react를 사용해서 <code>chrome extension</code>을 한번 만들어볼까 라는 생각이 들었다.  
chrome extension이라 하면 편의성이 제공되야 하는데 어떤 extension을 만들어야 내 손이 편해질까 생각하다가 **stash bit bucket에서 Pull request를 요청했을때 Reviewer들에게 Slack으로 메시지를 보내서 빠른 코드 리뷰 및 Approve를 유도**해야 겠다 라는 생각이 들었다.
  
extension을 개발하면서 모든 소스관련 전부 다 기록은 못하겠지만, 제작에 필요한 부분들에 대해 회고할겸 하나씩 기록해 보고자 한다.

## extension의 구성요소
<code>chrome extension</code>의 구성요소는 크게 4가지로 나눌수 있다.  
  
1. manifest.json
2. popup.js
3. content.js
4. background.js

### manifest.json
**첫번째로 필수 설정 파일**인 manifest.json파일에 대해서 간략하게 알아보자.

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
  "update_url": "https://clients2.google.com/service/update2/crx",
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