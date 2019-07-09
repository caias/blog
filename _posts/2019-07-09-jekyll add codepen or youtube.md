---
layout: post
title: jekyll blog에 youtube 또는 codepen을 embed하는법.
categories: jekyll
tags: jekyll
comments: true
---

> jekyll blog에 youtube 또는 codepen을 embed하는법
{:.lead}
* list
{:toc}

## 개요
포스팅을 조금씩 올리다보니 역시 필요한 부분이 생기기 마련인듯 하다. 소스가 좀 복잡하다 싶은 에시를 올리자니 <code>codepen</code>을 embed하는게 더 보기 편할듯 싶고, 가끔 <code>youtube를 embed</code>하고 싶을때도 있어서 이번에는 jekyll에 어떤식으로 넣어야 될지 정리해보자. 이 2가지를 embed시키는 방식은 굉장히 비슷하기 때문에 굉장히 쉽다. 

**NOTE**: 단.. 개인적으로 jekyll 관련 plugin을 쓰고 싶지는 않다!!! 필요성을 못느낌..
{:.message}

## youtube embed
마음에 드는 <code>youtube</code>영상의 embed 소스를 가져오면 다음과 같은 소스를 보일것이다.
~~~html
<iframe width="782" height="440" src="https://www.youtube.com/embed/-62uPWUxgcg" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
~~~

저 영상을 그대로 <code>markdown</code>파일에 넣어도 잘 나오기는 한다. 그러나 <code>responsive</code>를 따로 지원하지는 않는다. 그러므로 <code>_includes</code>폴더안에 html의 힘을 좀 빌려야 겠다.   

### include폴더에 youtube.html 생성
개인적인 폴더 구조 기준으로 보자면 **/includes/components/youtube.html** 이라는 경로에 youtube.html이라는 파일을 하나 작성하고 다음과 같이 적는다.    

~~~html
<div class="embed-container">
  <iframe width="800" height="600" src="https://www.youtube.com/embed/{{ include.id }}" frameborder="0" allow="accelerometer; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>
~~~

### css 소스 추가
blog css에 추가해줄 소스이다. 
~~~css
.embed-container {
  position: relative;
  padding-bottom: 56.25%;
  height: 0;
  overflow: hidden;
  max-width: 100%; 
 } 
.embed-container iframe{ 
  position: absolute;
  top: 0;
  left: 0;
  width:100%;
  height: 100%; 
}
~~~

### liquid tag 추가
markdown형식의 파일에서 포스팅 게시물 넣고싶은 곳에 다가 아래처럼 include파일 한줄만 추가해주면 된다.
{% raw %}
```liquid
{% include components/youtube.html id='-62uPWUxgcg' %}
```
{% endraw %}

#### 56.25% 수치란?
잡담이지만 왜 56.25%의 padding값을 줘야되는지 궁굼한 분들이 있을거 같아 짧게 남겨놓는다. 별거없다. **16:9 비율로 반응형을 나타내기 위해 쓰는거다.** (9/16) X 100 = 56.25가 나오기 때문에 저렇게 값을 주는것이다. 예시로 12:9비율을 원한다 싶으면 (9/12) X 100 값을 75%로 주면 된다.

## codepen embed
youtube embed하는 방식과 똑같다고 봐도 무방하다. 다만 codepen은 repsonisive를 지원한다라는 점

### codepen.html 생성 
같은 include폴더 경로 안에 codepen.html이라는 파일을 생성

{% raw %}
```liquid
{% assign username = include.username %}
{% unless username %}
{% assign username = site.codepen_username %}
{% endunless %}
<p class="codepen" data-height="265" data-theme-id="0" data-default-tab="html,result" data-user="caias"
  data-slug-hash="{{ include.hash }}"
  style="height: 265px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;"
  data-pen-title="{{ include.title }}">
  <span>See the Pen <a href="https://codepen.io/{{ username }}/pen/bPQmRa/">
      {{ include.hash }}</a> by {{ username }} (<a href="https://codepen.io/{{ username }}">@{{ username }}</a>)
    on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>
```
{% endraw %}

상단에 username은 다른사람 소유의 codepen소스를 가저올 경우를 대비해 user값이 있으면 그 user값을 쓰고 없으면 config.yml에 지정해놓은 나의 계정을 기본값으로 쓴다는 의미다. 

### liquid tag 추가
<code>markdown</code>에서 똑같이 넣고싶은 곳에 다가 아래처럼 include파일 한줄만 추가해주면 된다.
{% raw %}
```liquid
{% include components/codepen.html hash='bPQmRa' title='test' %}
```
{% endraw %}

이런식으로 embed가 필요하거나 import가 필요한 부분에 대해서는 비슷하게 응용해서 쓰면 쉽게 해결이 될것이다.   
참고용으로 상단에 youtube에서 추가한 html/css 구문을 예시로 codepen을 아래와 같이 embed해봤다.

{% include components/codepen.html hash='bPQmRa' title='test' %}

## 마치며..
포스팅 하다가 jekyll 커스터마이징 하느라 자꾸 삼천포로 빠진다..ㅠㅠ 그래도 몬가 조금씩 추가해가는 소소한 재미는 있긴 한데 포스팅이 자꾸 미뤄진다. 급하게 할거야 없지만.. 혹시 모를 이글을 누군가 봤을때 도움이 되면 좋겠다.