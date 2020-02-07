---
layout: post
title: GoogleAd 검색광고 도메인 소유권 소실 이슈 및 해결
categories: devlog
tags: Frontend Domain GoogleAd GMC
comments: true
---

> GoogleAd 검색광고 도메인 소유권 소실 이슈 및 해결
{:.lead}
* list
{:toc}

## 개요
어느날 갑자기 웹사이트 인증을 한 GMC(Google Merchant Center) 계정이 구글 서치 콘솔에 더이상 확인이 되지 않는현상이 발생했다.  
이로 인해 검색광고가 중단되는일이 발생하여 긴급해결해야될 일이 생겼다.

## 원인
예상되는 원인은 2가지였다 
1. 메타 태그의 키가 중간에 변경됐나??
2. 현재 메인페이지에서 쓰고 있던 도메인을 리뉴얼로 인해 다른 URL로 Redirect 시키고 있기때문에??  
(Redirect 전에는 metatag가 있고, 후에는 없기 때문일까?)  

## 해결방법
1. html head안에 meta tag를 추가 한다.  
```html
<meta name="google-site-verification" content="{key value}">
```  
  
2. Google에서 다운받은 Google{key value}.html 파일을 서버에 업로드 한다.

### 긴급배포완료
Google은 MetaTag에서 먼저 검색을 한 다음에 html파일을 찾기 때문에  
위 방법중에 하나만 골라서 해도 상관은없다. 


