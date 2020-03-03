---
layout: post
title: chrome extension으로 bitbucket stash pull-request관련 기능을 만들어보자#2
categories: frontend
tags: chrome-extension react axios typescript
comments: true
featured-img: /assets/img/blog/200221/extension-thumb-min.png
---

> chrome extension#02
{:.lead}
* list
{:toc}

## Background script
chrome extension 요소중 첫번째로 background script에 대해 설명해보고자 한다.
chrome extension을 개발하면서 사용한 tech는 <code>react</code> / <code>typescript</code> / <code>axios</code> 만 사용을 했다.
  
나는 팀원들에 대한 정보를 하드코딩으로 하기보다는 <code>stash bitbucket 또는 jira, confluence 등등</code>의 API를 이용하면 얻어 올수 있을거라 생각을 했다. 그리고 관련 API들을 검색을 해서 찾아보았다.  
(**각 회사마다 Atlassian 제품을 쓰는 회사들은 cloud형인지 설치형인지에 따라 API 엔드포인트가 다르다.**)
  
설계는 대략 background.tsx에서 API호출을 해 **팀원들의 정보**와 해당 extension은 혼자 쓰는게 아니고 팀원들이 같이 쓸 예정이기 때문에 설치한 사람이 팀원중 어떤유저인지를 알기 위해 Atlassian 관련 제품중 현재 **로그인된 유저 정보**를 가져와서 chrome.storage에 저장을 해두는게 나의 설계였다.  
(**회사에서 Atlassian관련 제품들의 아이디나 이름은 동일할 것이다**)

### axios module
일단 API를 몇번 쓰게 될테니 공통 모듈을 하나 만들어놓자.
  
**AxiosLoader.ts**
~~~typescript
import axios, { AxiosRequestConfig } from 'axios';

export async function AxiosLoader<T>(url: string, config?: AxiosRequestConfig): Promise<T> {
  try {
    const response = await axios({
      method: config?.method || 'get',
      url,
      data: config?.data || {},
    });
    return response.data;
  } catch (err) {
    console.log(err);
  }
}
~~~

### chrome storage module
이부분도 몇번 쓰이게 될테니 API 통신 후 또는 content script에서 storage정보를  <code>Get/Set</code> 하기 위한 공통 모듈을 하나 만들어놓자.
  
**Storage.ts**
~~~typescript
interface IResult {
  msg?: string;
  status?: number;
}

const rejectCheck = (status) => {
  if (!status) {
    reject({
      status: 500,
      msg: '[Storage] Error: storage is not defined',
    });
  }
}

const storages = (type: string = 'local', key: string): Promise<T> => {
  get<T>(type: string = 'local', key: string): Promise<T> {
    return new Promise<T>((resolve, reject) => {
      const storage = chrome.storage[type];

      rejectCheck(storage);

      storage.get(key, function(result: T) {
        resolve(result[key]);
      });
    });
  },
  set<T>(type: string = 'local', data: T): Promise<IResult> {
    return new Promise<IResult>((resolve, reject) => {
      const storage = chrome.storage[type];

      rejectCheck(storage);

      storage.set(data, function () {
        resolve({
          status: 200,
          msg: '[Storage] Success',
        });
      });
    });
  },
};

export function storage<T>(method: string = 'get', type: string = 'local', params: T) {
  const storage = storages[method];
  return storage(type, params);
}
~~~

### API통신후 Storage에저장하자
팀원정보 / 현재로그인한 유저 2가지의 정보를 가져오려면 API end point가 2군데였다.
2가지의 API를 get 한뒤에 storage정보에 담아주면 background에서 실행할 내용은 이게 끝이다. 

~~~typescript
import { AxiosLoader } from 'util/AxiosLoader';
import { storage } from 'util/Storage';

function initData() {
  Promise.all([
    AxiosLoader<IResponseUser>('현재 로그인한 유저 가져오는 API end point'),
    AxiosLoader<IResponseMembers>('팀원들의 정보를 가져오는 API end point')
  ])
    .then((allData: [IResponseUser, IResponseMembers]): void => {
      const [userData, memberData] = allData;

      // 로그인한 유저정보
      const userInfo = { name: userData.name, displayName: userData.fullName };
      // 유저정보 스토리지 저장
      storage<IStorage<IUser>>('set', 'local', { userInfo });

      // 프론트엔드개발팀 멤버 정보
      const frontMembers = memberData.results.map(member => {
        return { name: member.username, displayName: member.displayName };
      });
      // 프론트엔드 멤버정보 스토리지 저장
      storage<IStorage<IUser[]>>('set', 'local', { frontMembers });
    })
    .catch(err => console.log(err));
}
~~~

## 마치며..
이정도만 가지고도 얼추 필요한 data setting은 끝났고, interface에 대해선 구구절절히 따로 설명을 하진 않았다.  
여기에서 더 응용하면 얼마든지 응용하겠지만, 처음부터 파이를 너무 크게 잡으면 프로젝트가 산으로 가게 될까바 이 당시에는 목표를 이정도만 잡았다. 다음 포스트에서는 content script에서 storage에 저장해둔 data에 접근해서 react로 필요한 페이지에다가 랜더링 및 이벤트 바인딩에 대해 정리해보고자 한다.
