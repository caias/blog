---
layout: post
title: Leet Code 7. Reverse Intger (easy)
categories: leetcode
tags: leetcode
comments: true
---

> Reverse Intger
{:.lead}
* list
{:toc}

## 문제

**Example 1:**
~~~bash
Input: 123
Output: 321
~~~
**Example 2:**
~~~bash
Input: -123
Output: -321
~~~
**Example 3:**
~~~bash
Input: 120
Output: 21
~~~

**NOTE**: Assume we are dealing with an environment which could only store integers within the 32-bit signed integer range: [−231,  231 − 1]. For the purpose of this problem, assume that your function returns 0 when the reversed integer overflows.
{:.message}

2번 문제로 넘어갔어야 됬지만 아직는 내자신을 알기에 일단 easy문제부터 쭉 풀어보고자 한다.   

말그대로 Number형식을 reverse 시켜서 output으로 반환하는 문제이다.   
마지막 노트는 32비트 Integer 최대, 최소값을 넘으면 0으로 반환하라.

~~~js
/**
 * @param {number} x
 * @return {number}
 */
var reverse = function(x) {
}
~~~

### 첫번째 시도

~~~js
var reverse = function(x) {
  const arr = x.toString().split('').reverse().join('');
  return parseInt(arr);
};
~~~

음.. 그래도 문제를 보자마자 배열에 대해 공부한 보람이 조금 있나? 어떻게 할지 느낌이 빨리 왔다.  Number를 문자열로 반환후 '' String기준으로 자른후에 reverse method로 변환후 다시 ('')으로 조인하여 parseInt로 정수로 반환하면 되겠다 싶어 바로 답을 제출햇다. 결과는 **fail**.. 그럼 그렇지.. 내가 한번에 될리가... 문제는 음수를 체크하지 못했던 것이다. 

### 두번째 시도
~~~js
var reverse = function(x) {
  const numCheck = x > 0? 1 : -1;
  const arr = x.toString().split('').reverse().join('');
  return numCheck * parseInt(arr);
};
~~~
그렇다면 양수일때와 음수일때를 한번 체크해보자. numCheck라는 변수에다가 양수이면 1을 음수이면 -1을 저장해놓고 마지막에 변환된 arr을 정수로 만들고 리턴값에서 곱해주었다. 음수로 넣어서 테스트도 해보고 제출했다. 결과는 **fail** ....ㅠㅠ  무엇이 문제인가 봤더니 노트에 적혀있떤 부분이 문제다. 32비트 정수중에 Max/Min값을 체크하는 항목을 잊어먹었다.  그래서 이게 대체 몬데..??   
일단 구글링을 해보니 <code>MAX_SAFE_INTEGER</code>라는 기본 number 관련 표준내장객체 용어가 보이지만 이부분이 32비트 정수 최대값은 아닌듯 하다. javascript 32bit max Integer라는 검색어로 검색결과 2가지 결론을 도출할수 있었다. <code>2147483648</code> 또는 <code>Math.pow(2,32 - 1)</code>이 2가지의 값은 같다. pow연산자를 계산하면 2147483648라는 수가 나온다.    
좀 딥하게 들어가지는 못해도 32비트의 최대값은 2147483648 / 최소값은 - 2147483648 이라는 것이다. 

### 세번째 시도
~~~js
var reverse = function(x) {
    const numCheck = x > 0? 1 : -1;
    const arr = x.toString().split('').reverse().join('');
    if(parseInt(arr) >= Math.pow(2,32 - 1)) {
      return 0;
    }
    return numCheck * parseInt(arr);
};
~~~
나의 소스는 음수인지 양수인지는 나중에 곱해지니 알필요가 없기에 2147483648 크거나 같으면 0으로 return시켜준 후에 제출하니  **success** !!!   
easy는 easy인데 32비트 정수 최대/최소값은 너무 생소하다. 분명 언젠가는 쓰일일이 있겟지..?라고 생각하며 기억해둬야겠다.

## 결과
* Runtime: **76 ms**, faster than **81.68%** of JavaScript online submissions for Reverse Integer..
* Memory Usage: **35.9 MB**, less than **53.96%** of JavaScript online submissions for Reverse Integer..


76ms 나왔다. 이번에도 1등의 소스가 궁굼해졌다. 2,3등의 소스도 살펴봤지만 거의 비슷했었다. 궁굼하니까 이번에도 1등 소스 한번 분석해보자..

## 포스트 작성날 기준 1등의 소스
~~~js
var reverse = function(x) {
    const maxNum = Math.pow(2, 31);
    let ret = x < 0 ? '-' : '';
    
    x = Math.abs(x);
    
    do {
        ret += x % 10;
        x = Math.floor(x / 10);
    } while (x >= 1) {}	
    
    ret = parseInt(ret);

    return (ret >= -maxNum && ret <= (maxNum - 1)) ? ret : 0;
};
~~~

일단 기본적으로 <code>Array</code>로 제어 하는거 보단 <code>while</code>문을 써서 체크하는 방식이 1,2,3등의 방식이였다. 언제 모가 더 좋고 이런건 아직은 정확하게 모르지만. 이것도 기억해두자..   
1. x값을 10으로 나누면서 나머지들을 ret변수에다가 추가해준다.
2. 반면에 x는 10씩 나눠주면서 자리수를 하나씩 줄여나간다.
3. 결국 자리수대로 10씩 나눈 나머지값들이 숫자 반전이 되는셈이다.

## 마치면서

음.. 경험치라는게 이런상황을 두고 얘기하나보다. two sum 문제의 1등 소스를 분석할때도 느꼈지만, 스크립트의 세계는 아이디어 및 수학 싸움인듯 하다. 이런 생각들을 대체 어떻게 하는지.. 절로 벙찐다.. 경이롭고 존경스럽네. 분석하고 나면 아.. 이러지만 그때그때 접목할수 있는 날이 빨리 왔으면 좋겠다. 