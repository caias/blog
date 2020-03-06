---
layout: post
title: E2E test - Cypress#2 (device별 config 설정)
categories: frontend
tags: e2e cypress test e2e-test
comments: true
featured-img: /assets/img/blog/e2e/cypress-min.png
---

> Cypress device별 config 설정을 나눠보자.
{:.lead}
* list
{:toc}

## 개요
Cypress를 실행해보면 처음에는 example폴더 안의 TC들이 전부 다 같이 포함되서 실행된다. 이게 싫어서 폴더구조를 바꾸다보니 각 폴더별로 어떻게 나눠서 띄울수 있을까 고민하다가 <code>Cypress env</code>를 활용하여 설정을 나누는 법을 기록하고자 한다.

## Plugins/index.js
해당 파일에서 각각의 환경변수(ENV)를 가지고 설정을 해보자.  
<code>path</code>모듈은 node.js에서 기본제공하는걸 쓰면 되니 패스하고 <code>fs-extra</code>모듈을 설치하자.  
~~~js
npm i -D fs-extra
yarn add --dev fs-extra
~~~

**Plugins/index.js**
~~~js
const fs = require('fs-extra');
const path = require('path');

function getConfig(file) {
  const pathToConfigFile = path.resolve('./cypress', 'config', `${file}.json`);
  return fs.readJson(pathToConfigFile);
}

module.exports = (on, config) => {
  const file = config.env.configFile;
  return getConfig(file);
}
~~~
  
위에 소스를 설명 해보자면 cypress 실행시 env argument에 정의된 파일명을 불러와 cypress.config에 파일에 정의된 Object를 merge 시킨후 그 파일을 읽어들인 다는 의미이다. 나는 각각의 cypress 폴더 하위에 <code>pc</code> / <code>m</code>이라는 폴더를 놔두고 그 각각의 하위에 <code>pc.json</code> / <code>m.json</code>이라는 파일들을 생성했다.
  
![image](/assets/img/blog/e2e/cypress-folder-min.png){: data-width="720" data-height="366"}
폴더 구조
{:.figure}

## pc.json
Device별로 해당되는 test파일들만 리스트에 보여지고 싶었고 PC기준으로 테스트 해상도는 1920 X 1280으로 아래와 같이 예를들어 정해보았다. test파일의 구조는 본인들의 파일 구조에 맞게 변경하면 된다.
~~~js
{
  "viewportWidth": 1920,
  "viewportHeight": 1280,
  // 현재 나의 파일구조상으로는 /cypress/integration/pc/ 이다.
  "testFiles": "**/**/pc/*.*"
}
~~~

## m.json
개인적으로 생각하는 Cypress의 최고 단점중 하나인거같다. UA(**User Agent**)의 관한 지원이 굉장히 미흡하다고 본다. view port에서는 Device이름만 적으면 그에 맞는 해상도를 지원 해주긴 한데 UA를 일일이 찾아서 적어놔야지 모바일 디바이스로 인식을 한다. 그냥 viewport만으로는 PC모드에 해상도만 mobile디바이스인 셈이다. 물론 Proxy를 통해서 특정 디바이스에 연결하여 직접 테스트를 해볼순 있지만, 1차적으로는 PC의 모바일 mode로 확인을 하게되니 불편할수 밖에 없다고 생각한다. 게다가 디아비스별 Agent도 전부 나눠야 된다고 생각하면 Agent찾는것도 세팅하는것도 사실 꽤 경악스럽다.
~~~js
{
  "userAgent": "Mozilla/5.0 (iPhone; CPU iPhone OS 10_3_1 like Mac OS X) AppleWebKit/603.1.30 (KHTML, like Gecko) Version/10.0 Mobile/14E304 Safari/602.1",
  "viewportWidth": 375,
  "viewportHeight": 812,
  "testFiles": "**/**/m/*.*"
}
~~~

## npm script 추가
npm script에 하단의 2가지 script를 추가하자. <code>--env</code> 환경변수에 configFile이라는 값에 파일확장자를 제외한 이름을 적어주게 되면 끝. 이걸로 각 디바이스별 실행을 분리할 수 있는 기반이 완성되었다.
~~~js
"start:pc": "cypress open --env configFile=pc",
"start:m": "cypress open --env configFile=m"
~~~
  
실행을 한후 open 된 dash board의 settings tab을 보면 기본 설정에 override된 부분에서 확인 할수 있다.  

![Full-width image](/assets/img/blog/e2e/cypress-pc-min.png){:.lead data-width="1059" data-height="573"}
PC기준 실행 예시.
{:.figure}

## Cypress의 단점이라고 생각되는 점들.
1. cy.get(Selector)로 선택을 했을때 해당되는 **length가 한개밖에 없어도 꼭 .eq(0)을 붙여줘야된다**. TC를 작성하는데 있어서 이점은 생각보다 엄청 불편했다.
2. 위에서도 적어놨듯이 **UA설정 및 핸들링이 좀 까다롭다**. 
3. 다른 기타 E2E 테스트 툴이랑 가장 큰 차이점은 <code>async</code> / <code>await</code>가 없다는 점이기때문에 시점에 신경써야 된다.
4. error가 났을때 errorHandling이 다른 E2E Tool보다는 까다롭다. 또한 Cypress 구문에 대한 debugging이나 console을 찍어서 확인하는것 또한 어렵다.

## 마치며..
E2E Tool 선정을 위해 각 Tool마다 약 1주일 정도씩 테스트를 해본 후에 느낀점들을 간략하게 써봤다. 업무 우선순위가 높은게 아니라 짬짬히 시간내서 테스트 해보는 부분이기도 하고 너무 Deep하게 들어가지도 않았다. TC를 작성하는데 얼마나 진입장볍이 높은지, 디바이스별 설정은 쉬운지, 외부 툴로 제공할 수 있는지 이런 여부에 대해 우선으로 기준을 잡고있다. <code>Puppeteer</code> + <code>Majestic</code>도 E2E Tool로서 테스트를 좀 해보다가 어차피 puppeteer는 E2E전용으로 나온 툴이 아니기도 했고 느려터짐과 puppeteer안의 javascript를 쓸수 있게 해주는 Evaluate함수를 써보고 이건 진짜 아니다 싶어서 패스했다.  
  
  다음 E2E Test tool관련 Post는 <code>Testcafe</code>관련 포스팅이다.