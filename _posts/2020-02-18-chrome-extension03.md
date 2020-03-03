---
layout: post
title: chrome extension으로 bitbucket stash pull-request관련 기능을 만들어보자#3
categories: frontend
tags: chrome-extension react axios typescript
comments: true
featured-img: /assets/img/blog/200221/extension-thumb-min.png
---

> chrome extension#03
{:.lead}
* list
{:toc}

## content script
<code>content sciprt</code>를 개발하기에 앞서 <code>manifest.json</code>에다가 **stash에서 create pull request**페이지 url을 js파일과 연결을 시키자
  
**manifest.json**
~~~json
"content_scripts": [
  {
    "js": [
      "pullrequest.js"
    ],
    "matches": [
      "https://your-stash-site/pull-requests?create*",
    ]
  }
],
~~~
  
그리고 위에 적어준데로 pullrequest.tsx파일을 하나 만들어보자  
(**manifest의 파일이 js인 이유는 bundle된 파일 기준이다.**)
  
사실 Atlassian제품을 지금 다니는 회사 말고는 써본적이 없어서 다 같은 화면인지 정확히 모르겠다..;;(만약이란게 있으니...) 이 포스트는 화면이 같다 라는 전제하에 기록한다.  
같지 않다면, 어느 사이트든 extension으로 만들수 있으니 이글이 도움이 되어 응용할수 있기를 바란다.  
pull request create화면 자동완성 밑부분에다가 원클릭 검색 버튼을 아래처럼 만들것이다.  
  
create pullrequest 화면을 들어가보면 reviewer등록하는곳에 자동완성 기능이 있고 사람 이름으로 검색(자동완성)을 하여 reviewer를 등록하게 된다. 이름도 매번 타이핑해서 검색하는게 귀찮으니 팀원 전용 버튼을 만들어서 바로 검색이 가능하게 만들어보자.

이제 <code>background script</code>에서 저장해두었던 data들을 가져와서 <code>react 부모 component</code>를 만들고 props로 저장해두었던 data들을 넘겨보자.
  
### pullrequest.tsx

~~~typescript
import React from 'react';
import ReactDOM from 'react-dom';
import { storage } from 'util/Storage';
import ButtonList from './components/ButtonList';
~~~

module로 만들어두었던 storage module에서 get하기 위해 import시키고 팀원 멤버수 만큼 생성할 ButtonList component를 만들었다.

~~~typescript
async function init() {
  const container = document.querySelector('React Dom생성할 컨테이너');
  // 현재 로그인되있는 유저의 아이디
  const currentUser = await storage<string>('get', 'local', 'userInfo');
  // 팀원 멤버 정보
  const memberInfo = await storage<string>('get', 'local', 'memberInfo');
  // 로그인되있는 유저의 아이디를 제외한 나머지 멤버 추출
  const memberList: IUser[] = memberInfo.filter((value: IUser): boolean => value.name !== currentUser.name);
  
  ReactDOM.render(<ButtonList memberList={memberList} />, container);
}
~~~

로그인되있는 유저를 제외하고 추출하는 이유가 혹시 이해가 안되는 분들을 위해 간략하게 설명을 하자면 예를들어 팀원이 10명이고 이 익스텐션을 전부 사용한다면 팀원들 각각의 멤버는 본인을 제외한 나머지 9명이 되야 되기 때문이다. (본인이 본인한테 reviewer할거 아니자나요?ㅎㅎ)
  

### ButtonList.tsx
이제 <code>ButtonList.tsx</code> Component를 만들어봅시다. <code>memberList</code>라는 props가 예를들어 아래와 같은 구조라고 가정을 해보자.
~~~javascript
memberList: [
  {
    displayId: aaabbb,
    displayName: 팀원1
  },
  {
    displayId: cccddd,
    displayName: 팀원2
  },
  {
    displayId: eeefff,
    displayName: 팀원3
  },
  ...
]
~~~
**ButtonList.tsx**
~~~typescript
import React from 'react';
import UserButton from './UserButton';

const MakeButtonList = (props: ImemberList = defaultProps) => {
  const { memberList } = props;

  return (
    <React.Fragment>
      {memberList.map((value) => (
        <UserButton key={value.displayId} name={value.displayName} />
      ))}
      <AllDeleteButton />
    </React.Fragment>
  );
};

export default ButtonList;
~~~

component는 쪼개야 맛이죠. UserButton이라는 component를 하나 더만들었습니다.
<code>stash에 로그인할때 쓰이는 아이디값</code>이랑 <code>실제 이름</code>을 다시 props로 넘기고 buttonList를 렌더링 하면서 각각의 button component에 이벤트 바인딩을 해야겠죠?  
  
### UserButton.tsx
  
**UserButton**
~~~typescript
const UserButton = (props: Iname = defaultProps) => {
  const { name } = props;
  
  const nameBinding = (e: React.MouseEvent<HTMLButtonElement>) => {
    e.preventDefault();
    
    // 버튼을 클릭했을때 input.value값을 팀원 이름으로 넣는다.
    const searchArea: HTMLInputElement = document.querySelector('value값을 넣은 input element');
    searchArea.value = name;
    // 자동완성은 input창 위를 클릭을했을때 실행이 되기때문에 클릭이벤트를 실행해준다.
    const choice: HTMLElement = document.querySelector('자동완성 활성화 이벤트가 걸린 Element');
    choice.click();
  };

  return (
    <React.Fragment>
      <button onClick={nameBinding} style={ { 'margin': '0 10px 10px 0' } } className='aui-button'>{name}</button>
    </React.Fragment>
  );
};

export default UserButton;
~~~

이렇게까지 만들면 아래와 같은 그림이 나온다.  
![Full-width image]({{'/assets/img/blog/200218/extension-sample-min.png'| relative_url}}){:.lead data-width="100%"}
  
위의 Reviewers에 자동완성이 활성화 되는 경우는 2가지이다.
1. 포커스가 된 후 키보드 이벤트가 일어난다. 
2. 또는 input창에 마우스를 클릭한다.(실제 input창 위에 다른 Dom이 absolute로 띄어져있었음.)

그렇기 때문에 클릭이벤트를 줘서 자동완성 활성화를 시켜주고 그 다음에 input에다가 팀원 이름을 적어주면 자동으로 검색된 결과값이 나오게 된다.

## 마치며..
이미지에서 보는거처럼 전체삭제 기능도 만들어놨지만,중요한 부분은 아닌거라 생각하여 이부분은 다루지않았다.
- 이기능의 **장점**: 매번 reviewer들을 일일이 타이핑 안해도 된다.
- 이기능의 **단점**: 마우스 클릭을 2번씩 해줘야된다. (1클릭: 이름 자동완성, 2클릭: 자동완성 결과선택)

그래도 마우스 2번클릭이 더 편하지않은가... 사실 팀원 전체선택 기능도 만들었었다. 전체선택 버튼을 누르면 모든 팀원들이 들어가게 만드는 기능이였는데 reviewer를 항상 다 등록한다기 보다 co-worker위주로 등록을 하고 배포 환경에 따라 달라지니 그렇게 많이 쓰이지는 않을거 같아 뺐다. 또한 구현을 하더라도 bitbucket stash 페이지 안에서 바인딩되고 있는 이벤트 들이랑 꼬이게 되어 자동완성이랑 같이 쓰는거 자체가 너무 복잡해진다.
  
다음 포스트 에서는 pull request를 생성하고 난후 완료페이지에서 slack으로 메시지 보내기, approve안해준사람 재촉하기 버튼2개를 만들고 slack으로 연동하는 포스트들로 찾아뵙겠습니다.