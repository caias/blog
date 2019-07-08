---
layout: post
title: 01. Drum kit
categories: javascript30
tags: javascript30 javascript vanillaScript es6
comments: true
featured-img: /assets/img/blog/javascript30/drumkit.jpg
---

> javascript30의 첫번째 Tutorial 01. Drum kit
{:.lead}
* list
{:toc}

## 개요
우연히 구글링을 하던중에 [https://javascript30.com/](https://javascript30.com/) 이라는 사이트를 발견하였고 <code>leetcode</code>의 자료구조 같은 문제보다 더 끌려서 시작하게 되었다. 이메일만 입력하면 누구나 무료이고 강좌 동영상이 제공되기 때문에 그냥 풀어보기도 편하고 따라해보기도 편하다. 관련문제는 [https://github.com/wesbos/JavaScript30](https://github.com/wesbos/JavaScript30) 여기에서 다운받거나 clone받을수 있다.

## 사용방법
각각의 문제별로 폴더링이 되있고 그 안에는 <code>START.html</code>과 <code>FINISHED.html</code>이라는 파일이 같이 들어있다. 
1. **FINISHED 파일**을 실행해 보고 **동작 방식 이해 후, 스스로 프로그래밍**을 해본다. (개발 유경험자)
2. 동영상을 보고 따라하기.(입문자, 또는 초보)

따라 치는게 무슨 의미가 있을까 싶지만, 다른사람의 실전 프로그래밍을 볼 기회가 시실 흔치가 않다. 여러사람의 스타일을 볼수록 그 안에서 분명 얻을 것이 있을 것이고 따라하다가 막히면 더 찾아보고 자기것으로 만들면 된다고 생각한다.   

개인적으로는 <code>vanilla script</code>가 modern script라는것만 알지 제대로 써볼 기회가 없었기에 따라해보면서 알게 된것들에 대해 정리해 보고자 한다.
홈페이지에 적혀있듯이 다음과 같은 사항만 지키면 된다. 

**NOTE**: **No** Frameworks **No** Compilers **No** Libraries **No** Boilerplate
{:.message}

상단의 이미지처럼 해당되는 키보드를 누르면 효과를 동한반 소리가 나는 드럼 킷을 만드는 tutorial이다. 

## 제시된 코드
~~~html
  <div class="keys">
    <div data-key="65" class="key">
      <kbd>A</kbd>
      <span class="sound">clap</span>
    </div>
    <div data-key="83" class="key">
      <kbd>S</kbd>
      <span class="sound">hihat</span>
    </div>
    <div data-key="68" class="key">
      <kbd>D</kbd>
      <span class="sound">kick</span>
    </div>
    <div data-key="70" class="key">
      <kbd>F</kbd>
      <span class="sound">openhat</span>
    </div>
    <div data-key="71" class="key">
      <kbd>G</kbd>
      <span class="sound">boom</span>
    </div>
    <div data-key="72" class="key">
      <kbd>H</kbd>
      <span class="sound">ride</span>
    </div>
    <div data-key="74" class="key">
      <kbd>J</kbd>
      <span class="sound">snare</span>
    </div>
    <div data-key="75" class="key">
      <kbd>K</kbd>
      <span class="sound">tom</span>
    </div>
    <div data-key="76" class="key">
      <kbd>L</kbd>
      <span class="sound">tink</span>
    </div>
  </div>

  <audio data-key="65" src="sounds/clap.wav"></audio>
  <audio data-key="83" src="sounds/hihat.wav"></audio>
  <audio data-key="68" src="sounds/kick.wav"></audio>
  <audio data-key="70" src="sounds/openhat.wav"></audio>
  <audio data-key="71" src="sounds/boom.wav"></audio>
  <audio data-key="72" src="sounds/ride.wav"></audio>
  <audio data-key="74" src="sounds/snare.wav"></audio>
  <audio data-key="75" src="sounds/tom.wav"></audio>
  <audio data-key="76" src="sounds/tink.wav"></audio>
~~~

### sound play function
~~~js
function play(e) {
  const audio = document.querySelector(`audio[data-key="${e.keyCode}"]`);
  const key = document.querySelector(`div[data-key="${e.keyCode}"]`);
  if (!audio) { return }; // 1
  audio.currentTime = 0; // 2
  audio.play();
  key.classList.add('playing');
}
~~~
1. 현재 이 기능에서 쓰이고 있는 key는 총 9가지이다. 9가지 이외에 키를 눌렀을때는 함수가 실행될 필요가 없기에 return을 해준다. 이 부분을 혼자 코드를 짰다면 아마 놓치는 부분이였을거같다.
2. <code>currentTime</code>이라는 <code>HTML5 Audio/Video</code> Properties를 정확하게 몰라 검색을 좀 해보니 왜 이속성이 필요한지 알게되었다.
- 기본적으로 음악/영상 관련 method에는 <code>stop()</code>이라는 메소드가 없다. 그래서 중지를 하려면 <code>pause</code> 메소드를 쓴 후에 <code>currentTime=0</code>으로 돌려주는 방식으로 쓰게된다.
- 또한 여기에서 쓴 <code>currentTime</code>은 음악파일의 재생시간이 예를들어 10분이라면 이 소리가 끝나기전에 키를 한번 더 누르게 될때 소리재생시간이 이어져서 재생되는걸 방지하기 위해 연타를 위한 재생시간 리셋을 시켜주는 방식으로 쓰인다. 드럼에서 비트가 끊기면 안되잔소? 연타하는 맛이 있어야지...

### effect off function
~~~js
function catchEnd(e) {
  if (e.propertyName !== 'transform') return; // 1
  this.classList.remove('playing');
}

const keys = document.querySelectorAll('.key');
keys.forEach(key => key.addEventListener('transitionend', catchEnd)); //2

window.addEventListener('keydown', play);
~~~
1. 이거는 개인적으로 전혀 예상치 못한부분이였다.    
<code>transition</code>이 일어나는 모든 CSS관련 속성들이 <code>addEventListener</code>에 감지가 된다는 사실이 굉장히 놀라웠다. 아니 어찌저찌 알수야 있겠지 싶었지만 이런식으로 이벤트에 <code>propertyName</code>으로 들어올 줄이야.. 신박하다.

![Full-width image]({{'/assets/img/blog/javascript30/propertyName.png'| relative_url}}){:.lead data-width="100%"}
<code>console.log(e.propertyName)</code>입력시 결과
{:.figure}
- 이런식으로 <code>transition</code>이 일어나는 모든 property를 체크할 필요 없이 우리는 <code>transform</code>만 체크하면 되기때문에 다른 속성이 끝났는지 체크하는 부분들에 대해서는 <code>return</code>으로 스킵한다.
- <code>transition</code>이 일어나는 속성들이 많으면 많을수록 위처럼 스킵하기 위한 리턴은 성능향상에 도움이 되지 않을까 싶고, 개인적으로는 1번 튜토리얼에서는 이 부분이 제일 크게 얻어가는 부분인거 같다. 

2. 이부분은 음.. 사실 아직 몬가 개운치가 않다. 아직은 정확하게 모르지만 꼭 forEach를 돌려야만 되나 싶기도하고, 다른 포스트에 남겨뒀던거처럼 유사배열을 배열로 바꿔서 <code>Array.some</code>으로 구현도 해봤지만, 크게 다르진않은거 같다.
- <code>key event</code>에 대해 아직 지식이 부족한듯 하다. 흔히 쓰는 <code>$(this)</code>처럼 내가 누른 키만 찾는 방법을 아직은 못찾은 듯하다.
- 진짜 <code>transtionEnd</code>로밖에 감지할수 없는것인가 이부분도 아직은 감이 잘 안온다. 몬가 다른 방법이 있을거 같은데 미숙해서 떠오르지 않는다. 나중에라도 찾으면 이 포스트에 추가해놔야겠다. 

## 마치며..
따라해보고 모르는 단어 나오면 찾아보고 파악해보고 다시짜보고 하다보니 생각보다 굉장히 많은 공부가 되는거 같고, 확실히 <code>leetcode</code>보다는 훨씬 재밌다. DESIGN / HTML / CSS까지 전부 작성되있는 튜토리얼에 실무에 쓰일법한 여러가지 기능들을 주제로 하다보니 굉장히 유익한거 같다. 가능한 매일은 못해도 2일에 하나씩은 꼭 해봐야겠다. 