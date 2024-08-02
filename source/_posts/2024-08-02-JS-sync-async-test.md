---
title: JS sync & async test
gitalk: JS sync & async test
date: 2024-08-02 12:38:00
tags:
  - javascript
  - promise
category: 'Interview'
---

> **本篇宗旨在於紀錄我近期面試遇到的相關考題與解題，不會花太多篇幅講解原理，**
> **未來還有遇到類似的題目都會再更新回這篇。**

---

### 第一題

**問：程式執行完畢後，會顯示什麼？**

```js
for (var i = 0; i <= 3; i++) {
  setTimeout(() => {
    console.log(i)
  }, 0);
}
```
{% fold 解答 %}
4 4 4 4
{% endfold %}

如果專注在 JS 的同步異步問題，這邊就是要讓讀者理解到 JS 的 [Event loop](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Event_loop) 概念，
像 `setTimeout` 這種異步函式即便帶入的延遲時間是 0 ，也會被丟到一個 queue 當中，直到同步事件處理完畢後，才會按照順序把 queue 中的異步函式處理掉。

還有個小陷阱是 `var`，但要詳細 `var` 與替代方案的 `let` 就會是另一個正篇主題了，所以這邊蜻蜓點水就好。由於 `var` 並不是 `block scope` 的關係，會導致四次 `setTimeout` 都會作用在同一個 `var`，所以這段程式碼如果要符合人腦直覺運作的話，其實很單純的把 `var` 換成 `let` 就可以囉。

---

### 第二題

```js
function runAsync(x) {
  const p = new Promise(resolve => setTimeout(resolve(x, console.log(x)),1000))
  return p;
}

Promise.all([runAsync(1), runAsync(2), runAsync(3)]).then(res => console.log(res))
```

{% fold 解答 %}
1 -> 2 -> 3 -> [1, 2, 3]
{% endfold %}

這題....
不知道是不是在虛晃一招嚇嚇受試者，其實就是很單純的考 `Promise` 基礎用法，
除了寫得特別雜以，外應該沒什麼特別之處。但也有可能是我才疏學淺，希望有不同想法的人可以給我一點指教。

---

### 第三題


**問：請問五個字母出現的順序**

```js
setTimeout(() => console.log('A'), 100);

new Promise(resolve => {
  console.log('B')
  resolve()
}).then(() => console.log('C'));

setTimeout(() => console.log('D'), 0);

async function main() {
  await Promise.resolve()
  console.log('E')
}

main();

console.log('F')
```
{% fold 解答 %}
B -> F -> C -> E -> D -> A
{% endfold %}

B, F 因為是同步觸發的所以直接跳過

但為什麼同樣都是異步，先排入的 `setTimeout` 會比 `promise` 晚？
可以查查關鍵字 -> macro and micro tasks in javascript.
