---
title: Implement Promise.all
tags:
  - javascript
  - promise
category: Interview
gitalk: Implement Promise.all
date: 2024-08-14 10:15:58
---

第一題：你覺得按照常理邏輯，最下方的 log 該是什麼結果？
第二題：該如何實作下方的 promiseAll?

```js
  function promise1() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve("1");
    }, 2000);
  });
}

function promise2() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve("2");
    }, 000);
  });
}

function promise3() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve("3");
    }, 1000);
  });
}

function promise4() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve("4");
    }, 1000);
  });
}

function PromiseAll(promises) {
  return new Promise((resolve, reject) => {})

}


PromiseAll([promise1, promise2, promise3,promise4]).then((values) => {
  console.log(values);
});

```

---

### 第一題答案

{% fold 解答 %}
```js
 ["1", "2", "3", "4"]
```
{% endfold %}

---

### 第二題答案

當初遇到這題的時候我幾乎是 0 概念的去應考的...
應該說我知道怎麼用 promise.all，但完全沒有看過原碼。

最早的思路：

```js
function PromiseAll(promises) {
  return new Promise((resolve, reject) => {
    const resultList = [] // 儲存 promise 結果
    promises.forEach((promise) => {
      promise()
        .then((result) => {
          resultList.push(result);
          if (resultList.length === promises.length) {
            resolve(resultList)
          }
        })
        .catch((error) => {
          reject(error)
        })
    })
  })
}

PromiseAll([promise1, promise2, promise3,promise4]).then((values) => {
  console.log(values);
  // ["2", "3", "4", "1"]
  // 是正常運作了，但傳回來的結果沒有按照擺放的順序...
});

// 於是我改成用 index 做紀錄
promises.forEach((promise , index) => {
  promise()
    .then((result) => {
      resultList[index] = result;
      if (resultList.length === promises.length) {
        resolve(resultList)
      }
    })
})

// 但我忽略了非同步的特性，導致結果得...
// [undefined, "2", "3", "4"]
```

當時我就解到這邊，剩下因為時間因素就跟公司方用口頭討論處理方式，
為了避免非同步的問題，其實只要補個計數器，直接上完整程式碼：

{% fold 解答 %}
```js
function PromiseAll(promises) {
  return new Promise((resolve, reject) => {
    const resultList = []
    let fulfillCounter = 0 // 計數器
    promises.forEach((promise, index) => {
      promise()
        .then((result) => {
        resultList[index] = result;
        fulfillCounter += 1 // 完成就 +1
        if (fulfillCounter === promises.length) { // 改變判斷條件
          resolve(resultList)
        }
      })
        .catch((error) => {
        reject(error)
      })
    })
  })
}

```
{% endfold %}


