---
layout: post
title: jekyll 배포시 page build failure
categories: jekyll
tags: jekyll
comments: true
---

> page build failure
{:.lead}
* list
{:toc}

## 문제

**NOTE**: page build failure.
{:.message}

![Full-width image]({{'/assets/img/blog/190617/build_fail.png'| relative_url}}){:.lead data-width="100%"}

열심히 블로그를 커스터마이징 하고 이거 저거 해보다가 보니 어느순간부터 master에 push 할때마다 친절하게 메일이 왔다. 그것은 바로 **page build failure.**  
처음에는 왜때문에 에러가 나는지를 모르고 어디서도 에러 로그를 보기가 힘들었기 때문에 검색 키워드를 에러 로그쪽으로 초점을 맞춰서 검색을 해봤다.


### 문제 유추
일단 에러 로그를 보는방법중에 찾다보니 [travis](https://travis-ci.org/) CI툴을 써보는것이였다.   

1. travis 홈페이지에 github 계정으로 로그인 후 자신의 블로그 repository 연결
2. .travis.yml 파일을 블로그 프로젝트 루트안에 생성한다. 
3. 생성된 파일에 아래와 같이 입력
~~~yaml
language : ruby
script: "bundle exec jekyll build"
~~~
4. 다시 git push

... 아놔.. 또 메일이 온다. 그래.. 일단 Travis job에 로그를 확인해보러 가보자.. Travis 사이트의 Dashboard를 들어가보면 배포 상황과 에러가 났을 경우 에러 메시지가 출력된다.   
근데 필자는 에러가 없다. 없어..  하물며 Travis에서 메일로 배포 성공시에는 Passed 라고 또 친절히 메일이 온다.   
하긴.. 보통 jekyll serve로 로컬서버 돌리면 watch에서 에러 뜰때는 뜨긴 한데.. 별개의 문제인가보다.  

jekyll은 github page를 이용한 정적 파일 generator라고 볼수 있으니 github page관련해서 구글링을 해보니 page build failure 메시지의 이유는 크게 2가지 이유이다.
1. liquid 문법에 오류가 있을때
2. github pages에서 **지원하지 않는 jekyll plugin**을 썼을 때 

나는 2번에 해당하는 듯하다. 정확하게 어떤걸 지원하고 안하고를 검색하는거보다 좀더 자유롭게 운영하고 싶었기 때문에 다른 방안을 찾아보고자 했다.  

**NOTE**: custom plugin을 사용하는 경우라면은 보안상의 이유로 build할때 liquid tag를 인식 못하기 때문에 그렇게 메일을 날려댄다고 한다.
{:.message}

## 해결방안1
구글링을 하다보니 많이 알려진 방법 중에 Master branch와 Sourcebranch를 나누는 법을 시도해봤다.
~~~bash
origin/master
origin/source
~~~

soucre branch  
![Full-width image]({{'/assets/img/blog/190617/source_branch.png'| relative_url}}){:.lead data-width="100%"}
master branch  
![Full-width image]({{'/assets/img/blog/190617/master_branch.png'| relative_url}}){:.lead data-width="100%"}

1. github에서 **메인 branch를 source로 바꾸고** 소스 파일들을 관리한다.   
2. Master branch에는 _site 폴더안에 있는 모든 소스들을 관리한다 (위 이미지 참고).
3. source branch에 deploy.sh 파일을 하나 생성한후 다음과 같이 적는다.
~~~bash
git checkout source
git branch -D master
git checkout -b master
git filter-branch --subdirectory-filter _site/ -f
git push --all
git checkout source
~~~
4. .bash_profile에 다음과 같은 alias를 생성한다. 
~~~bash
# .bash_profile 열기
vi ~/.bash_profile
# alias 소스 추가
alias build-jekyll="bundle exec jekyll build"
alias deploy-jekyll="bash deploy.sh"
# 저장후 닫기
:wq
# bash_profile 실행
source ~/.bash_profile
~~~
5. .nojekyll 파일 생성
6. .gitignore 에서 _site폴더 해제
7. <code>build-jekyll</code>
8. <code>git commit -m'commit message'</code>
9. <code>deploy-jekyll</code>

일단 구글링과 다른 블로그를 참조하면서 그대로 따라해봤더니 메시지도 더이상 안날라오고 배포도 잘된다. 해결이 된듯 하다.  
그 후에 위에 소스와 git flow를 파악해보니 source branch에서 local master branch 랑 remote master branch를 삭제하고 _site폴더만 필터링 한후 마스터에 푸쉬 하는 형태이다.  
가만히 생각해보니 굉장히 조잡스럽고 짜친다. 문득 드는 생각이 <code>gulp</code> 또는 <code>webpack</code>으로 Dist 폴더를 배포 하는 대신에 _site폴더만 배포하면 되는일 아닌가 라는 생각이 들었다. 이 상황에서 드는 생각은 <code>Netlify</code>를 쓰면 그냥 쉽게 해결될 일 같다는 생각이 번뜩 든다.

## 해결방안2
1. [Netlify](https://www.netlify.com/) 사이트에 접속해 github 아이디로 로그인 한후 자신의 블로그 repository를 연결한다. 
2. build command와 publish directory 설정
![Full-width image]({{'/assets/img/blog/190617/netlify.png'| relative_url}}){:.lead data-width="100%"}
3. netlify의 기본 도메인은 괴랄한 문자로 되있고 보기 안좋으니 change site name을 통해서 입맛에 맞게 바꿔준다. 
4. netlify에 기본 branch 설정후 설정된 branch에서 작업하고 push를 하든. 타 branch에서 Default branch에서 PR을 날려서 머지를 하든 감지하여 배포는 자동으로 이루어 진다.(auto deploy옵션을 끄지않는한)
5. **기본 production brnach는 Master를 제외한 다른 branch로 설정한다.**   마스터로 하면 또 github에서 친절하게 page build failure. 메일을 쏴줄것이다. 왜냐하면 github pages는 master branch 또는 gh-pages branch 기준 배포가 Default값으로 설정되있다. 

**NOTE**: github page를 이용하는게 아닌 netlify를 도메인을 이용하기 때문에 github.io가 뒤에 붙는게 아니고 netlify.com이 뒤에 붙는다. custom 도메인도 사용 가능하나 이건 개인적으로 찾아보시길 바랍니다.
{:.message}

## 마치며..

github-page를 기반으로 굳이 해야 될 필요는 없다고 생각한다. 이 방식 외에도 해결 방법은 분명 많겠지만, 개인적으로 쓰던 CI툴인 netlify를 통해서 해결하였고 앞으로 여기에 webpack이나 gulp를 추가 적용 해봐야겠다. 아직까지는 더 많은 기능들을 자유롭게 쓰고 이 블로그를 커스터마이징 할게 많지만 차근 차근히 변화시켜나가야겠다.  
jekyll netlify라고 구글에 검색해보니 역시 지킬 공식 블로고 번역사이트에 굉장히 많은 방법이 나와있다.  
참고가 되길 바라며 마친다..

[jekyll](https://jekyllrb-ko.github.io/docs/deployment-methods/#웹-호스팅-제공자-ftp) - 한글 번역 공식 블로그
