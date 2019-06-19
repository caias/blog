---
layout: post
title: jekyll blog에 Tag를 추가해보기
categories: jekyll
tags: jekyll
comments: true
---


> page build failure
{:.lead}
* list
{:toc}

## 개요

**NOTE**: 내 블로그에도 tag와 tag cloud를 추가해보자.
{:.message}

다른 사람들의 블로그를 보다보니 나도 태그 기능을 구현해놓고 나중에 포스트가 쌓이면 좀 찾기 쉽겠다는 생각이 들어서 태그 기능을 추가해 보고자 한다.  
또 열심히 구글링을 해본다. 모든 구글링의 결과에는 심플하게 나와있고 심플하게 따라해보면 잘 안된다..역시 이해를 해야지 몬가 되나보다.


## 시도1

보통 liquid문법의 차이와 디테일차이, 디자인의 차이일뿐 일반적으로는 <code>_data</code>폴더에 <code>tag.yaml</code>을 추가해서 쓰는 방법이 많이 보인다.
~~~yaml
- slug: tag1
  name: Tag1

- slug: tag2
  name: Tag2

- slug: tag3
  name: Tag3
~~~

이 방법을 따라해보려다가 그때 그때 태그가 추가되면 <code>tag.yaml</code>에다가 수동으로 추가를 해줘야 하는방법인듯 하다.   
귀...찮....다.. 귀찮은것도 있지만, 태그가 엄청 많아지면 **관리가 안될거같다**. 
몬가 다른 방법들이 분명 있겠지만, 음.. 내입맛에 맞는걸 찾고싶다. 그러니 이 방법은 패스!

## 시도2

일단 liquid문법을 이용해서 페이지부터 만들어보자.

### Post파일에다가 태그를 추가해보자 
~~~markdown
---
layout: posts
title: 태그를 추가해보자
categories: jekyll
tag: jekyll tag add blabla
---
~~~
이런식으로 태그를 띄어쓰기로 여러가지를 추가해줄수 있다.

### <code>_includes</code>폴더에다가 tag를 모을수 있는 html을 추가해보자.   
예시로 <code>colletctags.html</code>라고 만들었다.
{% raw %}
```liquid
{% assign rawtags = "" %}
{% for post in site.posts %}
  {% assign ttags = post.tags | join:'|' | append:'|' %}
  {% assign rawtags = rawtags | append:ttags %}
{% endfor %}
{% assign rawtags = rawtags | split:'|' | sort %}

{% assign site.tags = "" %}
{% for tag in rawtags %}
  {% if tag != "" %}
    {% if tags == "" %}
      {% assign tags = tag | split:'|' %}
    {% endif %}
    {% unless tags contains tag %}
      {% assign tags = tags | join:'|' | append:'|' | append:tag | split:'|' %}
    {% endunless %}
  {% endif %}
{% endfor %}
```
{% endraw %}

### <code>_includes/head.html</code>의 안쪽에다가 위에서 만든 html을 불러온다.   
참고로 나는 _includes의 폴더아래 components라는 폴더를 쓰고있는 구조이다.
{% raw %}
```liquid
{% if site.tags == "" %}
    {% include components/collecttags.html %}
{% endif %}
```
{% endraw %}

### 추가하고 싶은 곳에 태그 리스트를 랜더링해보자.   
나는 포스트의 타이틀 밑쪽에 랜더링을 했습니다. 
{% raw %}
```liquid
<span>[
    {% for tag in page.tags %}
    {% capture tag_name %}{{ tag }}{% endcapture %}
    <a href="/tag/{{ tag_name }}"><code class="highligher-rouge"><nobr>{{ tag_name }}</nobr></code>&nbsp;</a>
    {% endfor %}
]</span>
```
{% endraw %}

### tag를 눌렀을때 tag 관련 페이지를 불러올 html 생성
예시: tagpage.html
{% raw %}
```liquid
---
layout: default
---
<div class="post">
  <ul>
    {% for post in site.tags[page.tag] %}
    <li><a href="{{ post.url }}">{{ post.title }}</a> ({{ post.date | date_to_string }})<br>
      {{ post.description }}
    </li>
    {% endfor %}
  </ul>
</div>
<hr>
```
{% endraw %}

### 이왕 여기까지 왔으니 tag cloud 페이지도 만들어서 추가하자.
예시: archive.html
{% raw %}
```liquid
<h2>Archive</h2>
{% capture temptags %}
  {% for tag in site.tags %}
    {{ tag[1].size | plus: 1000 }}#{{ tag[0] }}#{{ tag[1].size }}
  {% endfor %}
{% endcapture %}
{% assign sortedtemptags = temptags | split:' ' | sort | reverse %}
{% for temptag in sortedtemptags %}
  {% assign tagitems = temptag | split: '#' %}
  {% capture tagname %}{{ tagitems[1] }}{% endcapture %}
  <a href="/tag/{{ tagname }}"><code class="highligher-rouge"><nobr>{{ tagname }}</nobr></code></a>
{% endfor %}
```
{% endraw %}

tagpage.html 하단에 넣어보자. 
{% raw %}
```liquid
{% include components/archive.html %}
```
{% endraw %}


자 여기까지는 어느정도 구색이 갖춰졌으나, 문제는 tag들을 관리를 어떻게 하느냐가 문제이다. 현재 tag관련 .md파일들이 없어서 동작을 제대로 하진 않는다. 

## 태그 자동생성
구글링 하다 찾은것이 파이썬의 힘을 좀 빌려 보는것이다. 나는 파이썬에 대해 1도 모르지만 플러그인없이 자동으로 tag생성을 해준다기에 일단 실행해본다.   
블로그 root 폴더에 <code>tag_generator.py</code>라는 파일을 생성뒤에 아래 소스를 적고 저장한다.
~~~python
import glob
import os

post_dir = '_posts/'
tag_dir = 'tag/'

filenames = glob.glob(post_dir + '*md')

total_tags = []
for filename in filenames:
    print("filename: ", filename)
    f = open(filename, 'r', encoding='UTF-8')
    crawl = False
    for line in f:
        if crawl:
            current_tags = line.strip().split()
            if current_tags[0] == 'tags:':
                total_tags.extend(current_tags[1:])
                crawl = False
                break
        if line.strip() == '---':
            if not crawl:
                crawl = True
            else:
                crawl = False
                break
    f.close()
total_tags = set(total_tags)

old_tags = glob.glob(tag_dir + '*.md')
for tag in old_tags:
    os.remove(tag)
    
if not os.path.exists(tag_dir):
    os.makedirs(tag_dir)

for tag in total_tags:
    tag_filename = tag_dir + tag + '.md'
    f = open(tag_filename, 'a')
    write_str = '---\nlayout: tagpage\ntitle: \"Tag: ' + tag + '\"\ntag: ' + tag + '\nrobots: noindex\n---\n'
    f.write(write_str)
    f.close()
print("Tags generated, count", total_tags.__len__())
~~~

그리고 터미널에서 <code>./tag_generator.py</code>를 입력하면 <code>tag/</code>폴더에 각 태그들에 대한 .md파일들이 생성되며 잘 작동한다.   
본인이 윈도우OS라면 python홈페이지 들어가서 install하면 되고 MAC OS라면 기본적으로 설치 됬을것이다. 

1. 윈도우에서 <code>"UnicodeDecodeError: 'cp949' codec can't decode byte 0xe2 in position 6987: illegal multibyte sequence"</code> 에러가 뜬다면 인코딩 문제다.
파일을 ANSI로 바꾸던가. 위에 소스중에 <code> f = open(filename, 'r', encoding='UTF-8')</code>부분에 UTF-8 인코딩을 해주면 된다.
2. MAC OS에서 <code>./tag_generator.py</code> 실행시 permission denined가 뜨거나 없는 명령어라고 나온다면 <code>chmod +x tag_generator.py</code>를 치면 권한설정이 된다.

## 마치며..
다른 좋은방법들도 있겠지만 지금까지 찾아본것들에 대해 최대한 쉽게 풀어서 적어보려 했다. 포스팅 수를 늘리기 전에 이런 기반을 다져놓는것도 중요하다고 생각하기 때문에 이정도로 만족하고 이 글을 보게 되는 분들이 계시다면 도움이 되길 바랍니다. 

