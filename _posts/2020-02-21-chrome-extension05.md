---
layout: post
title: chrome extension으로 bitbucket stash pull-request관련 기능을 만들어보자#5
categories: frontend
tags: chrome-extension react axios typescript slack
comments: true
featured-img: /assets/img/blog/200221/extension-thumb-min.png
---

> chrome extension#05

{:.lead}
* list
{:toc}

**NOTE**: reviewer setting을 하기전에 Slack에 관한 특징을 간략하게 파악해보고 넘어가보자.
{:.message}


## Slack Mansion
<code>Slack Mansion</code>이란 여느 이메일과 같이 <code>@</code>앳 기호를 이용하여 Slack에 메시지를 보냈을때 그 해당 유저에게 알림 및 알림 카운트를 표시 시켜주는 기능이다. **slack에 메시지를 보낼때 mansion을 안달아서 보내면 unread count**가 안뜬다. (나만 이런가?)

## Slack ID Token
<code>Slack ID Token</code>이란 Slack에 로그인할때 쓰는 ID의 개념이랑은 다른 개념이고 각 ID(또는 팀원)마다 고유의 Token값이 부여가 되있다.  
**내 ID가 caias라고 @caias 이렇게 mansion을 붙여서 reviewr 메시지를 보낸다고 해도 알림은 안온다**. Token값으로 mansion을 해야지만 알림 및 메시지가 간다고 보면 된다.

<figure>
  <img alt="slack message" src="/assets/img/blog/200221/slackid-min.png" />
  <figcaption align="center">slack id token 예시</figcaption>
</figure>  
  
위의 이미지처럼 **각 slack channel에 속해있는 member마다 고유의 Token값이 있다**라는 걸 명심하자.  
  
**NOTE**: Token값을 보는 방법은 **Profile & Account > ...(더보기 버튼) > copy MemberId** 를 보면된다.  각 팀원들에게 확인 요청을 해서 취합하자.
{:.message}
  
그리고 아래 예시 소스처럼 <code>Interface</code> 정의를 해주자.  
하단의 member[숫자]는 atlassian id가 key값이 된다.
  
~~~js
const enum ESlackToken {
  caias = 'U02TUQJ23',
  member1 = 'U45GZT4AB',
  member2 = 'U753VBXAP',
  member3 = 'U624BDU3Z',
}
~~~

## Reviewer Setting
**Reviewer 관련 정보는 stash api와 통신해서 얻을 수 있고**, 그 정보는 <code>atlassian계정</code>과 이름이 포함되 있는데 나는 이중에 atlassian계정을 이용했다. 이걸 이용하기 위해 상단에 ESlackToken key값을 atlassian계정으로 세팅했다.  
atlassian계정을 Slack Id Token값으로 변환해주는 함수를 하나 만들자.
  
~~~js
const getSlackToken = (bitbucketId: string): string => {
  return ESlackToken[bitbucketId];
};
~~~
  
그리고 reviewer정보를 얻어와 reviewers의 value값에 넣어줄 string값으로 변환해보자.  
Slack incoming Webhook payload 구문에는 `<user>` 이렇게 꺾쇠로 Token값을 감싸줘야 된다고 명시가 되있다. 그 안에 <code>mansion</code>을 위한 @추가도 잊지말자.
  
~~~js
const convertName = (members: string[]): string => {
  let tokenList = '';

  members.forEach((member) => {
    const user = getSlackToken(member);
    user && tokenList += `<@${user}> `;
  });
  return tokenList;
};
~~~

## payload array push Reviewer
자 이제 reviewers Data setting이 끝났고. 그럼 이부분을 payload안에 밀어넣어줘야 하는일만 남았다. 일단 지난 포스트의 parameter구조가 기억이 안나서 이전 포스트를 다시 보고 올거 같은 분들을 위해 간략하게 다시 남겨본다.  
~~~js
const payload = {
  text: ,
  username: ,
  channel: ,
  attachments: [
    {
      fields: [
        {
          title:,
          value:,
          short:,
        }
      ],
      actions: [],
    }
  ]
}
~~~

여기서 **fields안에다가 형식에 맞게 지금까지 구한 reviewer정보들을 push**해주면 된다.
  
~~~js
// 정해진 형식에 맞게 Object로 맞춰준다.
const reviewrs = {
  title: '리뷰어',
  value: convertName(members),
  short: false,
};

//payload형식에 맞게 그냥 푸쉬해주면 된다.
payload.attachments[0].fields.push(reviewrs);

// 그리고 incoming webHook으로 POST보내주면 끝
AxiosLoader(YourIncomingWebHookURL, {data: JSON.stringify(payload), method: 'post'});
~~~

## Event Binding
지금까지 payload를 설정하는 방법들에 대해 알아보았다. **payload설정 및 POST까지 조합해서 함수**로 만들어 놓은 후 Event Binding은 본인들의 입맛에 맞게 설정해보면 된다.  
참고로 나는 Pull request 완료 페이지에 버튼을 2개 생성하고 PR이 생성완료후 버튼을 클릭하면 바로 payload함수를 실행하게 했다.  

<figure>
  <img alt="slack message" src="/assets/img/blog/200221/pr-complete-min.png" />
  <figcaption align="center">대충 이런 느낌으로...</figcaption>
</figure>  
  
재촉하기 기능도 마찬가지로 <code>stash api</code> 통신 후 **Approve status가 false**인사람만 다시 추려서 reviewer로 등록한 후에 payload로 보내는 방식의 응용일 뿐이다.

## 마치며...
실제 개발완료한지가 어느정도 시간이 흘렀지만, 이제서야 정리라도 할겸 기록해두는 이 포스트가 누군가에게는 조금이라도 도움이 됬으면 좋겠다.  

extension popup에 관한 글까지 포스트 올리려다가 **popup에 관한 내용은 너무 흔한 내용**이라서 굳이 추가하지는 않았다.  

나는 popup page에다가 각 <code>환경별 jenkins</code>와 <code>Swagger</code>페이지들로 이동할수 있는 링크들을 제공해주는 기능을 넣었다.  

이렇게 한번 만들고 나면은 Extension으로 할수 있는것들이 얼마나 많은지 새삼 깨닫게 되고 개발하면서 재미있고 즐기면서 할수 있는 <code>side project</code>였던거같다.  
꼭. 포스트와 같은 기능이 아니더라도 한번쯤은 도전하는걸 추천한다.