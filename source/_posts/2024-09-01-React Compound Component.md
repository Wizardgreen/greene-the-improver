---
title: React Compound Component
gitalk: React Compound Component
date: 2024-09-1 12:14:00
tags:
 - react
 - react design pattern
category: react
---

Compound Component (複合元件) 是一種 React 的設計模式，是一個將多種小元件組合在一起使用的大型元件。通常單個無法自己運作，常用 React API 或 useContext 來傳遞彼此專屬的 props，將這些邏輯封裝在其中而不會干擾到外部。因為這樣實作的特性，複合元件很適合用在高彈性但邏輯不被別的元件使用的場景，例如表單容器與欄位、下拉選單的容器跟選項等等...。

這邊會用上述提到的兩種方式實作 Tab 元件，但為了省時間就不特別寫樣式了，如果這篇真的有讀者的話，再麻煩自行想像一下它的美麗外型。


### 方法 1 - React API

```jsx
import React, { useState } from 'react';

const TabItem = ({ children, onClick, selected }) => {
  return (
    <li
      onClick={onClick}
      style={{ background: selected ? 'grey' : 'white' }}
    >
      {children}
    </li>
  )
}

const TabWrapper = ({ children }) => {
  const [selectedIndex, setSelectedIndex] = useState(0);

  const handleClickClosure = (index) => () => {
    setSelectedIndex(index)
  }

  return (
    <ul>
      {
        React.Children.map(children, (child, index) => {
          const result = React.cloneElement(child, {
            onClick: handleClickClosure(index),
            selected: selectedIndex === index,
          });

          return result
        })
      }
    </ul>
  )
}

// 可有可無的額外封裝，
// 看團隊風格而定，可以用來純放對外互動的邏輯，回傳選項、接受動態資料等等。
const TabCompoundComponent = () => {
  return (
    <TabWrapper>
      <TabItem>首頁</TabItem>
      <TabItem>分頁1</TabItem>
      <TabItem>分頁2</TabItem>
    </TabWrapper>
  )
}


// 使用方式，
// 因為我們把父子溝通的邏輯都透過 react api 封裝在個別元件中了，
// 所以其他開發者也不需關注到他們的邏輯，
// 只要子元件的接口設計相同，就可以低成本的直接在使用層替換 JSX 結構。
export function App(props) {
  return (
    <div className='App'>
      <TabCompoundComponent />
       {/* or */}
      <TabWrapper>
        <TabItem>首頁</TabItem>
        <TabItem>分頁1</TabItem>
        <TabItem>分頁2</TabItem>
      </TabWrapper>
    </div>
  );
}

```



### 方法 2 - useContext

顧名思義是利用 hook 的實作方式，不同於前者提到**可有可無的封裝**，在這種方式通常都會把定義 context 的邏輯放在封裝中，更明確的切割純邏輯與介面。

```jsx
import React, { useState, createContext, useContext, useMemo } from 'react';

const TabContext = createContext();

const TabItem = ({ label }) => {
  const { selectedLabel, handleClick } = useContext(TabContext)

  const onClick = () => handleClick(label)
  return (
    <li
      onClick={onClick}
      style={{ background: selectedLabel === label ? 'grey' : 'white' }}
    >
      {label}
    </li>
  )
}

const TabWrapper = ({ children }) => {
  // 目前情境沒用到，但需要的話，在這邊取得 context，
  // 比如說改變容器的樣式什麼的。
  // const {} = useContext(TabContext)

  return (
    <ul>
      {children}
    </ul>
  )
}

const TabCompoundComponent = ({ children, defaultLabel = null }) => {
  const [selectedLabel, setSelectedLabel] = useState(defaultLabel);

  const contextValue = useMemo(() => ({
    selectedLabel,
    handleClick: (label) => {
      setSelectedLabel(label)
    },
  }), [selectedLabel])

  return (
    <TabContext.Provider value={contextValue}>
      {children}
    </TabContext.Provider>
  )
}


export function App(props) {
// 稍微改變了一下應用方式，
// 要用原本的 react.children.map 生產出明確 index 來做檢查也是沒問題的。

  return (
    <div className='App'>
      <TabCompoundComponent defaultLabel="首頁">
        <TabWrapper>
          <TabItem label="首頁" />
          <TabItem label="分頁1" />
          <TabItem label="分頁2" />
        </TabWrapper>
      </TabCompoundComponent>
    </div>
  );
}

```

### 關於封裝的題外話

由於這種複合元件幾乎都缺一不可，我喜歡讓封裝結構與命名有更直接的關聯：
TabItem -> Item
TabWrapper -> Wrapper
TabCompoundComponent -> Tab

Tab.Item = Item
Tab.Wrapper = Wrapper

```jsx
import Tab from '@CompoundComponent/Tab'

const Page = () => {
  return (
    <Tab defaultLabel="首頁">
      <Tab.Wrapper>
        <Tab.Item label="首頁" />
        <Tab.Item label="分頁1" />
        <Tab.Item label="分頁2" />
      </Tab.Wrapper>
    </Tab>
  )
}

```

<!-- 
```js




/*
* 使用 compound component pattern 實作 Select 元件
*
* 元件使用案例:
*
* function App () {
*    const [value, setValue] = useState(0);
*
*    return (
*       <div>
*         <h1>{value}</h1>
*         <Select
*            onChange={(selectedValue) => setValue(selectedValue)}
*          >
*            <Select.Option value={0}>0</Select.Option>
*            <Select.Option value={1}>1</Select.Option>
*            <Select.Option value={2}>2</Select.Option>
*            <Select.Option value={3}>3</Select.Option>
*         </Select>
*       </div>
*
*     )
*  }
*
*
*/

import { createContext, useState, useEffect, useContext } from 'react';

const SelectContext = createContext({});

function Select ({ children, onChange }) {
  return (
    <SelectContext.Provider value={onChange}>
      {children}
   </SelectContext.Provider>
  )
}


function Option ({ value, children }) {
  // 請實作 Option 元件
  const {onChange} = useContext(SelectContext)

  return <button onClick={() => onChange(value)}>{children}</button>
}
```
 -->

---

> **給自己：**
> **你明明就會，卻在考試中失憶，最好導致 offer 比預期少了 15 萬左右 (獵頭說的)。**
> **馬的，給我好好複習。**

---