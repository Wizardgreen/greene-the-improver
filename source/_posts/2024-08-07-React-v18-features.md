---
title: React v18 features
gitalk: React v18 features
date: 2024-08-07 09:42:41
tags: react
category: React
---

### Auto Batching

我們通常進行一次 `setState` 就會促使 react 重新渲染一次畫面，這個功能是將多個 `setState` 集中在單次 re-render 中一起更新。過往情況只有 React 自己封裝的事件才會一起更新，而現在 18 版後，不論 `Promise` 或其他原生事件處理器，函式中的多個 `setState` 都會被一併處理。

```js
setTimeout(() => {
  // 多個 setState，一並更新
  setCount();
  setVisible();
  setIsUpdated();
}, 3000)

```

---

### Transitions

這是一個用來區分 state 更新行為緊急與否的全新概念：

- 緊急 (Urgent)：使用者與介面有直接相連的行為，例如打字、點擊等等。
- 不緊急 (Non-urgent)：例如前者互動後要將新資料更新至大型的陣列或物件中。

一般來說前述兩者是連在一起發生的行為。舉例來說，使用者輸入了某筆資料，而它就會被寫入陣列，最後再跟新列表。但如果今天 UI 或資料過於複雜就會導致使用者體驗不佳。在過往我們可能會利用 {% post_link "Debounce" %} 或 visualize 等盡可能減少運算的量，而現在我們有了一個新方案 - `startTransition`。

```jsx
import { startTransition } from 'react';

const onClick = (e) => {
  const { value } = e.target;

  setInputValue(e)

  startTransition(() => {
    setSearchQuery(e); // 非緊急更新
  })
}
```

像上方被 `startTransition` 包起來的行為就會被標示為非緊急的更新，而這種類型的差別就是在於他們會被其他緊急更新中斷。假若非緊急更新被中斷，React 就會捨棄原本的運算過程，直接使用最新的結果去渲染畫面。

---

### Suspense

這是一個新的元件，可以讓使用者在上層就直接設定子元件還沒準備好顯示時的 fallback。
```jsx
import { Suspense } from 'react';

<Suspense fallback={<LoadingSpinner />}>
  <Page />
</Suspense>

```

---


### New Hooks
#### useId

這是一個能產生出獨立且穩定 id 的 hook，作用很單純，也可以拿來避免 SSR 可能會遇到的 Hydration Mismatch Error。
但像這樣運算的行為不適合用在列表的 `key`，官網也有特地強調。

```jsx
import { useId } from 'react';

const App = () => {
  const id = useId();

  return <div className="App">
    <h3> id is {id}</h3>
  </div>;
}
```

#### useTransition

就像前面的 `startTransition` 可以讓你標記非緊急的 state 更新，但除了改成 hook style 以外，他還多了一個回傳變數用來辨別是否正在進行 transition。

```jsx
import { useTransition } from 'react';

const [isPending, startTransition] = useTransition();
startTransition(()=>{
 // doSomething
})

```
#### useDeferredValue

同樣是 transition 概念的延伸，但這次只做用在單一變數上。
但要切記，接收 deferredValue 的元件要先用 `useMemo` 或 `memo` 包起來，避免其餘變音。

```jsx
import { useState, useDeferredValue } from 'react';
import SlowList from './SlowList.js';

export default function App() {
  const [text, setText] = useState('');
  const deferredText = useDeferredValue(text);
  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      // wrapped by memo
      <SlowList text={deferredText} />
    </>
  );
}
```

#### useSyncExternalStore & useInsertionEffect

兩個都是建議用於開發 lib 的工具，不建議在應用端使用。
由於我自己也不熟，而且看起來主題偏龐大，我再另外獨立寫兩篇連結過來好了。