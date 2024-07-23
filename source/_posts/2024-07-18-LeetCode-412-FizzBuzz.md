---
title: LeetCode - 412.FizzBuzz
date: 2024-07-18 16:04:56
tags:
  - Algorithm
  - LeetCode
category: 'Interview'
---

題目連結：<https://leetcode.com/problems/fizz-buzz/>

菜鳥題目了，只是無意間重讀當初寫的版本後，發現能夠更好，就有了記錄起來的想法。

原本的寫法很單純，按照題目需求將 3 與 5 依序過濾，但要先過濾掉兩者的最小公倍數 15。

```js
// 舊版
const fizzBuzz = function(n) {
  const result = [];
  for (let i = 1; i <= n; i++) {
    if (i % 15 === 0) {
      result.push('FizzBuzz');
    } else if (i % 3 === 0) {
      result.push('Fizz');
    } else if (i % 5 === 0) {
      result.push('Buzz');
    } else {
      result.push(String(i));
    }
  }
  return result;
}
```


隔了幾年看到這題才突然意識到只要順序符合，這邊其實可以用字串累加的方式去處理。
```js
// 新版
const fizzBuzz = function(n) {
  const result = [];
  for (i = 1; i <= n; i++) {
    let string = ''
    if (i % 3 === 0) string += 'Fizz';
    if (i % 5 === 0) string += 'Buzz';

    if (string.length > 0) {
      result.push(string)
    } else {
      result.push(`${i}`)
    }
  }
  return result
};
```
除了更工整以外，程式碼傳達的涵義跟題目描述更接近，而且換作是商業邏輯的話，只要核心概念不變，繼續增加不同數字的判斷也不影響未來的閱讀。但使用舊版寫法就會考慮到各種最小公倍數的組合，判斷條件就會倍增許多。


