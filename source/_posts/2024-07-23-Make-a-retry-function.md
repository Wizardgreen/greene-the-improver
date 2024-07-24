---
title: Make a retry function
date: 2024-07-23 09:45:48
tags:
- 'JavaScript'
category: 'Interview'
---

### 問題

希望實做出下方函式：

`execute` 代表想要執行的 async api。
`times` 則是最高重試次數。

假設成功就直接中斷，失敗則反覆執行到次數上限或成功為止，如果最後都失敗就回傳最後的失敗結果。

```js
  const retry = async (execute, times) => { }
```

大概三年前入職前公司的時候設計過一次，後來蠻穩的就沒在維護，到現在已經完全忘記怎麼寫的了。我第一個想法是透過 `times` 建立出固定長度的陣列，然後利用 `Promise` 去判斷成功與否之後的邏輯。

但現在冷靜後想想，我把狀況弄的太複雜了這，是我一直以來都會有的缺點。
事實上簡單的 `while` 控制搭配 `async/await` 理論上就能有個不錯的版本。

### 答題

我想這會是一個蠻符合人腦思考順序也容易改動的版本。

```js
  const retry = async (execute, times) => {
    let result = await execute();

    // 我假定這個回傳結果都會包含代碼讓我判斷成功與否
    if (result.code === 200) {
      return result
    } else {
      // 首次執行後進入重試機制，從 1 開始
      let retryCount = 1;
      while (retryCount <= times) {
        result = await execute();
        if (result.code === 200 || retryCount === times) {
          return result;
        }
        retryCount ++
      }
    }
  }
```