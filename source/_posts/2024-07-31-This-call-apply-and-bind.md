---
title: This, call, apply and bind
gitalk: This, call, apply and bind
date: 2024-07-31 12:04:12
tags:
  - javascript
category: 'Interview'
---

最近兩間博弈公司的面試讓我又開始重新回味 `this` 這個新手必考的概念，
廢話不多說，直接上題目吧。

---

### This

這兩個 `console.log` 會印出什麼資訊？

```js
var age = 30
var name = 'peter'

var user = {
  name: 'peter',
  age: '18',
  props: {
    getName: function() {
      return this.name
    },
  },
  getAge: () => {
    return this.age
  }
}

console.log(user.props.getName());
console.log(user.getAge());
```

{% fold 解答 %}
undefined
30
{% endfold %}

解釋 `this` 的文章已經有超多大大寫過了，想要 "詳解" 上網搜尋都很多，
我自己的話則是很推這篇 [JavaScript深入之从ECMAScript规范解读this](https://github.com/mqyqingfeng/Blog/issues/7) 。

但以比較粗略跟方便解釋的角度，我自己有個說法：

1. **傳統函式：** `this` 就是呼叫的那一層。
2. **箭頭函式：** `this` 就是頂層，在建立函式時就已經固定住了。

箭頭函式應該就不解釋了，寫幾個範例來解說為什麼我會這樣解釋傳統函式中的 `this`。

```js
var name = 'andy'
var user = {
  name: 'peter',
  props: {
    getName: function() {
      return this.name
    },
  },
  getName: function() {
      return this.name
  },
}

const getNameOutsideA = user.props.getName;
const getNameOutsideB = user.getName;


console.log(user.props.getName()) // undefined, 互叫層是 props 同一層沒有 name
console.log(user.getName()) // peter, 呼叫層是 user
console.log(getNameOutsideA()) // andy, 定義在 props，但在 global 呼叫，所以 this 是 global
console.log(getNameOutsideB()) // andy, 定義在 user，同上


const button = document.createElement('button')
button.addEventListener('click', function() { console.log(this) })
button.click() // <button></button>
```

像這樣隨著不同的使用方式導致 `this` 不一樣，對程式開發來說是有一定淺在風險的，
開發者可能無意間就改變了 `this` 的對象，更不用說共同來開發的時候也可能變為一場災難。

但這只是粗略大方向，實際情況還會變化，這段落最後留個有趣的事情：
```js
class Car  {
  owner = 'Greene';
  showOwner() {
    console.log(this.owner);
  };
};

const myCar = new Car();

myCar.showOwner(); // Greene

var outsideShowOwner = myCar.showOwner;

outsideShowOwner(); // TypeError: Cannot read properties of undefined
window.outsideShowOwner(); // undefined


// normal
(() => {console.log(this)})() // window (global)

// in strict mode
(() => {console.log(this)})() // undefined
```

---

### Call, Apply And Bind

也就是因為 `this` 這麼俏皮動感的變數，所以我們才需要用這三個函式來重新定義它，
三者是 `Function.prototype` 底下的 method, 目的都一樣，但用法有些許不同。

> **注意：這不適用於箭頭函式，箭頭函式在建立時就已經固定了。**

#### Call

將傳入參數作為新的 `this` 且立即執行函式。

```js
const car = {
  speed: 80
}

function getSpeed() {
  return this.speed
};

getSpeed() // undefined

getSpeed.call(car) // 80

// 然後 call 也有多個參數可以作為原本函式的參數帶入

function tellThemHowQuickIam(msg) {
  return `${msg} ${this.speed}`
}

tellThemHowQuickIam.call(car, 'my speed is') // my speed is 80
```

#### Apply

跟 `call` 可以說是一樣的，唯一的差異在於參數帶入的方式不同。

```js
const data = {
  value: 10,
}

function factory(times, plus) {
  return (this.value * times) + plus;
}

factory.apply(data, [2,3]) // 23，如果是 call 就會搖寫好幾個傳入參數
```

#### Bind

跟前兩者稍微不同，這是回傳一個封裝好的新函式，看自己什麼時機拿來使用。
```js

const car = {
  speed: 80
}

function getSpeed() {
  return this.speed
};

getSpeed() // undefined

const boundGetSpeed = getSpeed.bind(car)
boundGetSpeed() // 80


// 同樣的，boundFunction 也可以傳入參數
function tellThemHowQuickIam(msg) {
  return `${msg} ${this.speed}`
}

const boundTellThemHowQuickIam = tellThemHowQuickIam.bind(car, 'my speed is')
boundTellThemHowQuickIam() // my speed is 80
```