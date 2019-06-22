---
layout: post
title: Leet Code 1. two Sum (easy)
categories: leetcode
tags: leetcode
comments: true
---

> Two sum
{:.lead}
* list
{:toc}

## 문제

**NOTE**: Given an array of integers, return indices of the two numbers such that they add up to a specific target.  
You may assume that each input would have exactly one solution, and you may not use the same element twice.
{:.message}

nums라는 배열과 target number 2가지 인자를 받아서 nums의 배열안에 2개의 값을 합산했을때의 target값이 나온다면 2개의 index를 리턴하라 모 이런 내용입니다.  
단, 같은 엘리먼트를 2번 사용하지 말라는군요

~~~js
/**
 * @param {number[]} nums
 * @param {number} target
 * @return {number[]}
 */

Given nums = [2, 7, 11, 15], target = 9,

Because nums[0] + nums[1] = 2 + 7 = 9,
return [0, 1].
~~~

### 첫번째 시도

~~~js
const twoSum = function(nums, target) {
   for(let i = 0; i < nums.length; i ++) {
        let current = target - nums[i];
        let idx = nums.findIndex((item, idx) => {
            return item === current;
        });
        return [i, idx];
   }
};

console.log(twoSum([2,7,11,15], 9));
~~~

for문을 이용하여 nums 루프를 돌리고 target값에서 nums[i]의 값을 뺀값을 nums배열안에서 찾아보려고 했었다. 결과를 제출했으나 **Fail**..  
이유인 즉슨 <code>towsum([3,2,3], 6)</code>인경우에 output이 [0,0]이 나와버린다.  
같은 엘리먼트를 2번 쓰지 말라는 부분에 위배된다.

### 두번째 시도
~~~js
const twoSum = function(nums, target) {
   for(let i = 0; i < nums.length; i ++) {
        let current = target - nums[i];
        let idx = nums.findIndex((item, idx) => {
            return item === current;
        });
        if ( i !== idx){
            return [i, idx];
        }
   }
};

console.log(twoSum([3,2,3], 6));
~~~
위의 엘리먼트 2번쓰지 말라는 제약조건을 피하기 위해 현재index와 idx로 반환되는 값이 같지 않을때 배열로 리턴해서 제출했다. 결과는  **Fail**  
아 왜!!!!!! 다시한번 디버깅해보니 조건때문에 findIndex 메소드에 True로 떨어지는 경우가 없어서 -1 을 반환해버린다.

### 세번째 시도
~~~js
const twoSum = function(nums, target) {
   for(let i = 0; i < nums.length; i ++) {
        let current = target - nums[i];
        let otherNumber = nums.indexOf(current);
        if(i !== otherNumber && otherNumber !== -1){
            return [i, otherNumber];
        }
   }
};

console.log(twoSum([3,2,3], 6));
~~~
indexof 메소드로 변환하고 조건을 하나 더 추가해줬다. 이렇게 수정하고 나서 제출!!! **success**  
결과값은 [2,0]으로 나오고 문제의 expect는 [0,2]인데 Accept가 되어서 기분이 좀 찝찝하다.  
29가지의 Testcase들을 통과했다는 메시지와 모가됬든 첨으로 문제를 해결해서 뿌듯하다. 

## 결과
* Runtime: **148 ms**, faster than **24.40%** of JavaScript online submissions for Two Sum.
* Memory Usage: **33.8 MB**, less than **98.73%** of JavaScript online submissions for Two Sum.


148ms 나왔다. 음..이정도면 만족하지 모.. 라고 생각 하던 찰나 그래프에 제일 빠르게 작성한 케이스가 **32ms**라구요..?
어차피 1초도 안되는 시간 가지고 대수롭지 않게 생각할수 있겠지만, 궁굼하다.. 한번 보고싶어졌다.

## 포스트 작성날 기준 1등의 소스
~~~js
const twoSum = function(nums, target) {
  const comp = {};
  for (let i = 0; i < nums.length; i++) {

    if (comp[nums[i]] >= 0) {
      return [comp[nums[i]], i];
    }

    comp[target - nums[i]] = i;

  }
};

twoSum([1, 2, 3,4,5], 7);
~~~

Object를 쓸 생각은 1도 못했다.. 게다가 잘 이해가 안되는 부분들이 있어서 삽질하면서 파악을 좀 해봤다.  
1. typeof === 'undefined' 이런방식으로 체크하는건 많이 봤어도 이걸 일부러 False로 떨궈서 일종의 Falg역활을 하게 만드는 소스는 처음봤다. 
2. 빈 Object를 변수에 할당후 Undefined로 if문을 False로 통과시키면서 변수 comp의 Obejct에 key(target과 nums[i]의 차이), Value에 i를 반환하여 차곡 차곡 쌓는다.
3. 루프를 돌면서 nums[i]값이 comp 변수 key값과 같은 값으로 있는 부분을 찾게되면 Undefined의 값이 아니라 comp[nums[i]]값을 반환한후 [comp[nums[i]], i]으로 return 시키면서 break;

## 마치면서

음.. 역시 스크립트의 세계는 경이롭다. Leet code 사이트에서도 1번 문제이자 Easy 난이도의 문제고 프론트엔드로 전향한지 얼마 안되서 스킬이 딸리지만, 이렇게 다른사람의 소스만 몇개 봐도
얻는것이 많은거 같다. UI개발자로 있으면서 확실히 array와 object를 쓸일이 많이 없었기에 가뜩이나 더 어렵게 느껴지고 당장에 내가 이런 비슷한 유형의 문제를 만났을때 이걸 활용할 정도는 안되겠지만, 참고해두고 이런방식도 있다는것만 알고 넘어가야겠다고 다짐하며.. 이 포스트를 마친다..