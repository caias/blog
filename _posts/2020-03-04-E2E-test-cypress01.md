---
layout: post
title: E2E test - Cypress#1
categories: frontend
tags: e2e cypress test e2e-test
comments: true
featured-img: /assets/img/blog/e2e/cypress-min.png
---

> E2E test tool Cypress의 기능중에 commands/Cypress.$에 대해 알아보자.
{:.lead}
* list
{:toc}

## 개요
실무에 적용할 E2E Test 자동화를 위해 관련 Tool들에 대한 리서치 및 간략한 테스트를 했었던 기억을 떠올리며 기록을 해본다. 이 포스트 역시 <code>Jest</code> / <code>chai</code>등에 관련된 기본 Test Code(이하 **TC**)에 관련된 기능들에 대해서는 따로 언급하진 않겠다. 모르는 분들은 관련 문서를 찾아보면 굉장히 쉽게 되있고, TDD라도 해본 분들도 역시 굉장히 친숙하기 때문이다.  
  
앞으로 E2E관련 포스팅에 들어가는 내용들은 주로 해당 툴을 쓰게 된다면 **설정**이나 **이런 기능들을 활용**하면 좋을거 같다. 또는 **이런 이슈가 있었다** 라는 부분에 대해 조그만한 팁이라도 공유 및 기록하고자 포스팅을 쓴다.  

## Commands.js
Cypress의 기능중 <code>Commands.js</code>라는 부분이 괜찮다고 느껴졌다.  
물론 그냥 다른 js에서 만들고 export 및 import 해오면 되긴 하지만 어쨌든 자신들만의 API중에 Helper로 등록해서 사용할수 있는 기능을 제공을 해준다는거 자체가 좋다고 해야될까..? 기본적인 예시를 보자.
  
- 다음은 Cypress의 기본문법중에 <code>getAttribute</code> 및 <code>innerText</code>를 구하는 구문이다.
~~~js
cy.get(Selector).invoke('attr', 'data-filter-type').should('eq', '최신');
cy.get(Selector).invoke('text').should('contains', 'test code');
~~~
  
사실 그냥 사용을 해도 나쁘진 않다. 다만 E2E 테스트를 하다보면 생각보다 코드가 엄청 길어지고 가독성이 떨어지게 된다. 이런 부분을  Commands.js에서 helper를 등록하여 바꿔보자.
  
**cypress/support/commands.js**
  
~~~js
Cypress.Commands.add('getAttribute', (selector, value) => {
  cy.get(selector).invoke('attr', value);
});

Cypress.Commands.add('getValue', (selector, value ) => {
  cy.get(selector).invoke(value);
});
~~~
  
Cypress를 설치하고 이 파일을 처음 열게 된다면 주석으로 sample 코드들에 대한 정보가 있으니 참고하면 된다.  
위 코드에 대한 설명을 간략하게 하자면
  
1. Cypress.Commands에 추가를 한다.
2. 추가할 Event name space이다.
3. cypress문법에 전달할 arguments이다.
  
이렇게 추가를 해주고 다시 위에 작성한 TC를 바꿔보자면 아래와 같이 적용하면 된다.
  
~~~js
// 이전 code
cy.get(Selector).invoke('attr', 'data-filter-type').should('eq', '최신');
cy.get(Selector).invoke('text').should('contains', 'test code');

// command를 적용한 code
cy.getAttribute(selector, 'data-filter-value').should('eq', '최신');
cy.getValue(selector, 'text').should('contains', 'test code');
~~~
  
활용할수 있는 제일 기본적인 예시이지만, 소스양의 차이는 별로 안나도 <code>command</code>를 적용한게 개인적으로는 훨씬 가독성이 좋다고 생각한다.

## Cypress.$ (Builtin Jquery)
<code>Jquery</code>는 아마 대부분 친숙할 것이다. Cypress안에도 이 Jquery가 내장되있어서 같이 활용할순 있지만, 아쉬운점은 Cypress문법과는 별개이다. 이 말이 이해가 안된다면 소스를 보는게 이해 빠를것이다. 예시로 카운트 value값을 먼저 구한뒤 increase button을 누르면 value값은 +1이 되야 한다라는 TC이다.
  
~~~js
const $ = Cypress.$
const $inputSelector = $('input[type="number"]');
const value = $inputSelector.val(); // (1)

cy.get(increaseButton).click();
cy.get($inputSelector).should('eq' value + 1); // (2)
~~~
  
대부분 이 소스를 보면 정상 작동 하겠지 라고 생각하겠지만, 정상 작동 하지않는다. 그 이유들에 설명을 해보자면,  
1. <code>cy.get($inputSelector)</code>이부분이 첫번째 error이다. 위에서 얘기한거처럼 Jquery를 쓴 구문을 cy.get으로 불러올때는 호환이 되지않으므로 에러가 발생한다. Jquery구문을 사용한 selector를 쓰려면 <code>cy.wrap(el => el)</code>이런식으로 return 시켜주는 함수를 써야되지만 이렇게까지 해야될 이유를 아직은 찾지못했기 때문에 그냥 cy.get('input[type="number"]')으로 쓴다.
  
2. 2번째 이유는 **(1)** 주석처리 해놓은 부분이 Number가 아닌 <code>String</code>으로 반환이 되기때문이다. value가 String이기 때문에 **(2)**주석의 기대값은 2가 아니고 11로 나와서 테스트 실패가 된다. 그럼 이렇게 한번 고쳐보자.
  
~~~js
cy.get('input[type="number"]').should('eq' Number(value) + 1);
~~~
  
그럼 이제 해결이되서 잘 작동하겠지? 라고 생각해도 안된다. 이번에 이유는 <code>cy.get('input[type="number"]')</code>이부분이 또 String을 반환을 해서 <code>'2' === 2</code>가 False가 되기때문에 테스트 실패이다.  Cy.get()으로 불러오는 값을 Number로 변환해주는 방법을 아직 찾지 못했기에 하단의 방법으로 TC를 일단 작성했었다.  

~~~js
cy.get('input[type="number"]').should('eq' `${Number(value) + 1)};
~~~
  
참 별거 아닌 흔한 TC인데도 Cypress의 이런저런 특성을 알수 있는 좋은 예시였던거같다.

## 마치며..
2가지만 적어봤는데도 분량 조절 실패다. 글이 너무 길어도 잘 안 읽히기 때문에 한번 짤라가고자 한다. 다음 <code>Cypress</code> 2번째 포스팅에서는 PC/MW 디바이스별 TC들을 좀 나눠서 open하고 싶어서 분리 시킨 부분 및, 실제로 테스트 해보면서 불편했던 점들에 대해 적어볼 예정이다.