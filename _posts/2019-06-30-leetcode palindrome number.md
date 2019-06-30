---
layout: post
title: Leet Code 9. Palindrome Number (easy)
categories: leetcode
tags: leetcode
comments: true
---

> Palindrome Number
{:.lead}
* list
{:toc}

## 문제

**Example 1:**
~~~bash
Input: 121
Output: true
~~~
**Example 2:**
~~~bash
Input: -121
Output: false
Explanation: From left to right, it reads -121. From right to left, it becomes 121-. Therefore it is not a palindrome.
~~~
**Example 3:**
~~~bash
Input: 10
Output: false
Explanation: Reads 01 from right to left. Therefore it is not a palindrome.
~~~

**NOTE**: Determine whether an integer is a palindrome. An integer is a palindrome when it reads the same backward as forward.
{:.message}

**Follow Up** Coud you solve it without converting the integer to a string?

easy 3번째 문제다. NOTE에 보면 Number 형식을 String으로 변환하지말고 풀어보라고 권장한다. 흠.. 한번 해보자.

Palindrome Number란 숫자의 역순이 원본이랑 같으면 True를, 음수이거나 원본이랑 다르면 False를 출력하는 문제다.

~~~js
/**
 * @param {number} x
 * @return {boolean}
 */
var isPalindrome = function(x) {
    
};
~~~

### 첫번째 시도

~~~js
var isPalindrome = function(x) {
  const origin = x;

  if(x < 0) { return false};

  let rev = 0;

  while(x > 0) {
    rev *= 10;
    rev += x % 10;
    x = Math.floor(x / 10);
  }

  if( rev === origin) {
    return true;
  } else { 
    return false;
  }
};
~~~

호. 첫번째 시도에 답을 맞춘건 처음이네... **success** !!!    
easy난이도 2번째 문제였던 reverse integer에서 풀었던 방식이 Number를 String으로 변환시켜서 풀어봤었다. 그 당시 속도1위 문제에서 파악했을때 while문을 써서 숫자 반전을 출력했던 것이 기억나 그래도 이용해보고자 했다. 어차피 음수는 전부 <code>false</code>출력이니 0보다 작으면 바로 <code>return false</code>시켜버리고 while을 이용해 반전을 시킨뒤 최초 숫자를 대입시켜놨던 origin변수랑 대조해서 같으면 <code>true</code>로 반환 시켰다.   
역시 1등 소스를 파악한게 좀 도움이 됬나보다. 

## 결과
* Runtime: **184 ms**, faster than **89.60%** of  JavaScript online submissions for Palindrome Number.
* Memory Usage: **44.7 MB**, less than **98.46%** of JavaScript online submissions for Palindrome Number.


## 포스트 작성날 기준 1등의 소스 (140ms)
~~~js
/**
 * @param {number} x
 * @return {boolean}
 */
var isPalindrome = function(x) {
    if (x < 0) return false;
    return reverseInt(x) == x;
};

var reverseInt = function(int) {
    let rev = 0;
    while(int != 0) {
        let pop = int % 10;
        int = Math.floor(int/10);
        rev = rev * 10 + pop;
    }
    return rev;
}
~~~

## 마치면서

function으로 하나 더 뺀거 말고는 음.. 그렇게 큰 차이가 없는거같은데 저정도의 속도 차이가 나는 듯 하다. 한가지 인상적인 부분은 나는 x의값이 계속 변하기때문에 origin이라는 변수에 초기 x값을 저장해뒀지만, 1등의 소스는 함수와 변수를 비교함으로써 origin이라는 함수 자체가 필요없는 변수라는걸 깨달았다. 게다가 <code>if/else</code>구문을 return시켜 버림으로 그냥 한줄로 쓰는게 가능하다. 이런걸 보면 아직은 내가 멀은 느낌이다.. 다음에 나오는 문제들은 한번 객체지향으로 나눠서 풀어봐야겠따. 
