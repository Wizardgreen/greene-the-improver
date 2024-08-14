---
title: React v18 - useSyncExternalStore
gitalk: React v18 - useSyncExternalStore
date: 2024-08-09 08:25:17
tags: react
category: React
---

> **useSyncExternalStore is intended to be used by libraries, not application code.**
> **useSyncExternalStore 主要設計給工具庫使用，而非應用層的程式碼**

---

### 概念
可以把這 React Hook 想像成一個小型的 redux，你可以透過他來訂閱外部的儲存庫。

```jsx
// 在你的元件最上層使用 useSyncExternalStore 來帶入外部資料
function component(params) {
  const snapshot = useSyncExternalStore(store.subscribe, store.getSnapshot)
  //...
}

```
hook 會回傳 store 的快照，然後它需要接收兩個參數：

1. subscribe: 接收一個訂閱 callback 的函式，這個 callback 應該要在每次 state 變動時觸發，而這個行為同時也會使元件重新渲染
2. getSnapshot: 也是一個函式，用來回傳 data 的快照。

---

### 應用
假設我需要額外記錄使用者上次滑鼠點擊的時間：

```js
// in store.js
export const store = {
  state: {
    lastActiveTime: null,
  },
  subscribe: (listener) => {
    window.addEventListener('click', () => {
      const date = new Date();
      store.state = {
        ...store.state,
        lastActiveTime: date.toLocalString(),
      }
      listener();
    })
    return () => {
      window.removeEventListener('click', listener)
    };
  },
  getSnapshot: () => store.state
}

// in component
import { useSyncExternalStore } from 'react';
import { store } from './store.js';

const Component = () => {
  const snapshot = useSyncExternalStore(store.subscribe, store.getSnapshot);

  return (
    <h1>最後活躍時間</h1>
    <p>{snapshot.lastActiveTime}</p>
  )
}
```

而當然，如果有需要的話，我們也可以像是 Redux 一樣定義 dispatch / action 來觸發更新。

```jsx
// in store.js
export const store = {
  state: {
    lastActiveTime: null,
  },
  listener: null,
  setLastActiveTime: (time) => {
    store.state = {
      ...store.state,
      lastActiveTime: time,
    }
    store.listener();
  }
  subscribe: (listener) => {
    window.addEventListener('click', () => {
      const date = new Date();
      store.state = {
        ...store.state,
        lastActiveTime: date.toLocalString(),
      }
      listener();
      store.listener = listener
    })
    return () => {
      window.removeEventListener('click', listener)
      store.listener = null,
    };
  },
  getSnapshot: () => store.state
  emitChange: () => {
    store.listener()
  }
}

// in component
import { useSyncExternalStore } from 'react';
import { store } from './store.js';

const Component = () => {
  const snapshot = useSyncExternalStore(store.subscribe, store.getSnapshot);

  const onClick = () => {
    const date = new Date();
    store.setLastActiveTime(date.toLocalString());
  }

  return (
    <h1>最後活躍時間</h1>
    <p>{snapshot.lastActiveTime}</p>
    <button onClick={onClick}>
  )
}

```

可想而知你還可以更近一步的去定義 Action type 或甚與 `context` 搭配使用，這就看大家怎麼發揮拉，但整體規模變大以後我認為還是比較適合直接用 Redux。