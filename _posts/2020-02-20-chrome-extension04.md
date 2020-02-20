---
layout: post
title: chrome extension으로 bitbucket stash pull-request관련 기능을 만들어보자#4
categories: frontend
tags: chrome-extension react axios typescript slack
comments: true
---

> chrome extension#04

{:.lead}
* list
{:toc}

## Slack연동 해보기
<code>slack</code>으로 메시지를 보내는 법은 생각보다 굉장히 간단하다.  
<code>incoming webhook</code>을 slack app안에서 생성하고 부여되는 webhook url에 **Ajax Post통신으로 payload를 parameter**로 보내기만 하면 된다!  

**NOTE**: incoming webhook 생성하는 포스트는 발에 치일정도로 WEB상에 많으니 따로 적진않겠다.
{:.message}
  
대신에 **incoming webhook으로 보내는 payload**를 <code>디자인</code>??? 해보는 포스트를 기록해보고자 한다.

## payload template
<code>payload design</code>이란 말그대로 slack에 메시지를 보낼때 어떤 template을 써서 보낼지를 구현하는것이다. 여러가지 template이 어느정도 정해져있고 그에 맞는 parameter들이 정해져있다. 그중에서 나는 아래와 같은 template으로 꾸몄다.
  
<figure>
  <img alt="slack message" src="/assets/img/blog/200221/slack-temp-min.png" data-width="445" data-height="306" />
  <figcaption align="center">구현했던 slack message template</figcaption>
</figure>
  
위와 같은 template예시로 소스를 한번 보자.
~~~js
const isTest = true;
const payload = {
  text: '`PR ' + state + ' (#' + id + ')` : ' + title,    //(1)
  username: organizer,    //(2)
  channel: isTest ? '@caias' : slackAPI.getChannel(stashState),   //(3)
  attachments: [
    {
      color,    //(4)
      fields: [
        {
            title: 'PR생성자',
            value: organizer,
            short: true     //(5)
        },
        {
            title: '대상 브랜치',
            value: toRefId,     //(6)
            short: true
        },
        {
            title: '상세내용',
            value: description,     //(7)
            short: false
        }
      ],
      actions: [    //(8)
        {
            type: 'button',
            text: 'Overview',
            url: `${currentUrl}/overview`
        },
        {
            type: 'button',
            text: 'Diff',
            url: `${currentUrl}/diff`
        },
        {
            type: 'button',
            text: 'Commits',
            url: `${currentUrl}/commits`
        }
      ]
    }
  ]
};
~~~
  
사실 이 payload안에 value값들을 구하는 과정과 기능들의 **business logic은 굉장히 많이 들어간다.**  
간략하게 팁을 드리자면...
1. 최대한 PR생성 완료페이지 안에서 활욜 할수 있는 정보나 값들을 활용하자.
2. 처음부터 너무 많은걸 구현하려 하지말고 PR OPEN에 관한 message만 생각하고 구현해보자.

몇가지의 stash end point도 추가적으로 더 필요하기도 하고, 일부분들은 사정상 다룰수가 없기때문에 위 소스 주석의 순번대로 라도 간략하게 기록하고자 한다.  
참고로 business logic이 엄청 많은 이유는 모든 PR state와 MERGE / APPROVE / UNAPPROVE 등등의 상태값을 다 체크해서 상황별 메시지를 구현했기 때문이다.

### (1) text
<code>'`PR ' + state + ' (#' + id + ')` : ' + title</code>  
- **state**는 현재 PR상태가 OPEN / MERGED / DECLINED에 관한 상태값이다.
- **id**는 PR이 생성될때 부여되는 ID값이다. PR완료 페이지 <code>location.href</code>에서도 값을 구할수 있다.
- **title**은 작업 Branch이며 이 역시 완료페이지에서 값을 얻을 수 있다.  
Backtick이 들어간 이유는 이미지에서 보는거와 같이 slack 특성상 highlight를 주기 위해서이다. 

### (2) username 
말그대로 PR을 생성한 사람이다. 이미지에서는 ID로 나오지만 실제로는 이름으로 구현을 했다. PR완료 페이지 또는 <code>chrome.storage</code>에 저장해둔 현재 로그인된 유저에서 값을 얻을 수 있다.

### (3) channel
<code>channel: isTest ? '@caias' : slackAPI.getChannel(stashState)</code>  
**message를 보낼 slack channel을 입력하는 부분**이다. 위에 쓰인 함수가 어떤 함수인지 설명은 안하겠지만, destination Branch에 맞게 각각의 slack channel을 설정해두었다.  

**NOTE**: isTest라는 변수를 넣은 이유는 테스트를 할때마다 실제 쓰고있는 slack channel에 계속 message를 보내는게 싫어서 개인 channel로 보내 테스트했다.
{:.message}

### (4) color
<code>상단 template 이미지에 초록색 라인</code>을 찾아보자.  
바로 저부분을 담당하는 parameter이며, hexacode를 직접 입력하는것도 가능하다.  
기본적으로 제공하는 good, bad도 있으며. good은 초록색 bad는 빨강색으로 표기가 된다.  
평소에는 good, decline 또는 unapproved 이런 nagative 요소에는 bad로 나눠서 표기되도록 했다.

### (5) short
<code>boolean</code>값으로 들어가며, true일때는 주최자, 대상브랜치 처럼 한줄에 column이 2개가 되고 false일때는 그냥 한줄에 column이 1개가 된다.

### (6) toRefId
<code>Destination Branch</code>명이다. 이부분은 API를 통신해서 얻을수도 있고 PR완료 페이지에서도 값을 얻을 수 있다.

### (7) description
<code>commit 할때 적었던 commit message</code>를 나타내는 부분이다.

### (8) actions
actions에도 여러가지 기능이 있지만, 그중에서도 comment가 적히는 <code>overview</code>, 실제 변경점을 보여주는 <code>diff</code>, commit이력을 보여주는 <code>commits</code> 링크를 연결시켜주는 버튼을 생성해보았다.

## 마치며..
리뷰어 관련소스가 payload 소스안에 없는것을 발견하셨나요?  
이부분은 다음 포스트에 연결해서 기록해 보고자 한다. 별거 아닌 기록이 될 수도 있겠지만, 리뷰어 관련해서는 다음과 같은 기록들을 할 예정이다.  
- <code>Reviewer세팅</code>
- <code>Slack mention</code>
- <code>Slack ID Token</code>
- <code>payload array push Reviewer</code>  

생각보다 짜잘하게 기록 및 설명을 하고자 하는양이 좀 된다.  
마지막으로 slack을 쓰고있다면, template playground인 url을 공유한다.  
payload template을 테스트 할 수 있는 사이트라 본인의 입맛에 맞게 payload를 설정해보자!  
  
[slack message template](https://api.slack.com/tools/block-kit-builder?mode=message&blocks=%5B%5D)