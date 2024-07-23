---
title: Clickjacking
date: 2024-07-22 08:30:41
tags:
- 'JavaScript'
- 'Web Security'
category: 'Interview'
---

這題大概是我某次錯失一個理想職缺的關鍵，很明顯我的職涯發展沒有好好顧及到網路安全，印象中只處理過像是 XSS 或一些登入驗證、會員權限等實作。但不論如何，失敗就是失敗趁著記憶猶新先記錄起來吧。


### 問題

「我們在知名的串流影因網站看到這段 code，你覺得這是用來做什麼的？」
```html
<script>
  if (self.location === top.location) {
    const body = document.getElementsByTagName('body')[0];
    body.style.display = "block";
  } else {
    top.location = self.location;
  }
</script>
```

### Clickjacking

根據 Wiki 的 [Clickjacking 頁面](https://en.wikipedia.org/wiki/Clickjacking)，這是一種誤導使用者點擊網頁後觸發不預期行為的惡意技術。最常見的手法就是網路上的一些免費資源或影片，在使用者點擊播放或下載按鈕的頁面，其實整個畫面事先蓋了一層透明的互動元素或利用 `pointer-events: none` 讓點擊行為穿越元素，進一步把你導入別的廣告頁面等等根本不是你想要的方向。

常見的狀況在你點擊數次以後就會將蓋板移除，讓你獲得原本預期的資源，目前為止是算是比較有良心的。但再更壞一點的就會用 iframe 將信任度高的網站偷過來再覆蓋上自己的元素，透過使用者的信任去偷取填入表單的個人資訊或甚至是刪除帳號等惡意的行為。而前述的那段程式碼正是為了避免自己的網站被人放入到 iframe 中而設計的防禦手段。

### 問題解說

`window` 物件中有兩個屬性可以用來檢查自己是否被別人嵌入到 `iframe` 中

`window.top`: 瀏覽器最上層的 `window`。
`window.self`: 當前的 `window`，也就是說假設你被嵌入了，這層就會是你的 `window`

有了上述的知識後我們再一步一步理解最一開始的程式碼：
```html
<script>
  // 檢查自己是否被嵌入 iframe
  if (self === top) {
    // 否，將一開始藏好的元素展開
    const body = document.getElementsByTagName('body')[0];
    body.style.display = "block";
  } else {
    // 是，將上層重新導向回自己的網頁
    top.location = self.location;
  }
</script>
```

掌握知識以後這題就變得非常簡單了，真後悔當初沒能了解到本題的含義。

### 另一個防禦手段: X-Frame-Options
`X-Frame-Options` 是伺服器的一種 Header，可以用來直接告訴瀏覽器是否拒絕 `iframe`、`embed` 等等鑲嵌方式。


| 指令 | 用途 |
|----------|----------|
| DENY    | 無論如何頁面都無法被置於 frame   |
| SAMEORIGIN    | 所有 ancestor frames 都是同源的話，才能被放入 frame   |


前面 script 的方式來說仍有機會被繞過，而後者則是過時的瀏覽器會不支援，所以最好還是雙管齊下才會有比較好的保障。