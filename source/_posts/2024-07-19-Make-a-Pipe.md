---
title: Make a Pipe
date: 2024-07-19 16:14:35
tags: 'JavaScript'
category: 'Interview'
---

Pipe 在 functional programming 是不可或缺的一環，一般來說都是直接從 Ramda 或 Lodash 中引用，猜這考題是為了想了解受試者是不是單純會用工具而完全不懂其原理的工程師，苦笑。

說來可笑，我是第二次遇到這個考題了，上次遇到的時候因為非常緊張而整個人都是接近當機的狀況，那次說不定是我有史以來最慘烈的面試呢。第二次是前陣子終於一間有興趣的公司，雖然不確定這題自己到底有沒有寫對，但整個技術關還是沒過的，陣著記憶猶新先記錄起來吧。


需求：請實作下方的 pipe function
```js
  const add1 = (n) => n + 1;
  const double = (n) => n * 2;

  const factory = pipe(add1, double);

  const result = factory(10);
  // result === 22
  // 10+1 -> 11*2 -> 22
```

寫到這裡突然想起，在第二次考試的當下我也很緊張(我通常被人檢視的時候都會特別慌)，糊裡糊塗的竟然打算用 forEach 來處理接收的陣列變數，真是犯了天大的錯誤...


```js
  // 解答
  // 其實整個過程很單純，透過 reduce 能夠累加的功能反覆加工變數而已
  const pipe = (...fns) => (x) => fns.reduce((acc, fn) => fn(acc), x);
```

寫到這邊讓我心中冒出一個疑問 「pipe 跟 compose 有什麼不同？」。

...
..
.

還真的沒什麼大差別，只是加工的順序不一樣。
Pipe: 由左到右
Compose: 由右到左

參考文章: <https://dev.to/vivek96_/understanding-the-difference-between-compose-and-pipe-in-javascript-11ji>