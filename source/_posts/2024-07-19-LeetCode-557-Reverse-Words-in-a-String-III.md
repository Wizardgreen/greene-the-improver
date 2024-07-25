---
title: LeetCode - 557. Reverse Words in a String III
tags:
  - Algorithm
  - LeetCode
  - 'LeetCode - Easy'
  - 'Two Pointers'
category: 'Interview'
date: 2024-07-19 08:52:20
---
題目連結：<https://leetcode.com/problems/reverse-words-in-a-string-iii/>

你會收到個一個字串，其中每個單詞的順序是正確的，但詞中的字母順序卻是顛倒的，在維持單詞順序與空白字元的情況下還原語句。這題曾出現在我被刷掉的技術面當中，雖然按照題意我不覺得自己有寫錯，但事後還是想了一點可以改進的版本。

```js
// 舊版，runtime 79.86%, memory 49.21%
const reverseWords = (s) => {
  const wordArray = s.split(' ');
  const resultArray = [];
  wordArray.forEach((word) => {
    let correctWord = ''
    for (let i = word.length - 1; i >= 0; i--) {
      correctWord += word[i]
    }
    resultArray.push(correctWord)
  })
  return resultArray.join(' ')
};
```


回頭來看舊版過於粗暴，應該要搭配 [344. Reverse String](https://leetcode.com/problems/reverse-string/) 提到 modifying in-place 的前提去做，避免多餘的記憶體與操作。但我在這邊又吃了一個鱉，因為在 344 裡面收到的資料是一個 char[]，而本題是一個 String，然而 String 在 JS 是 Array-like 卻同時又 immutable (我卻忘記了)，因此我用 `string[index]` 的方式修改字串撞了幾次牆才意識到這件事情：
```js
// 新版 (錯誤示範)，用 two point 的概念加上自製函式
// 然而卻換來 runtime 跟 memory 都只有 5% 的結果

const reverseWords = (s) => {
  const wordArray = s.split(' ');

  // 建立調整字串用的函式
  const setCharAt = (str, index, char) => {
    return str.substring(0, index) + char + str.substring(index + 1);
  }

  wordArray.forEach((word, index) => {
    let left = 0;
    let right = word.length - 1;
    let newWorld = word

    while (left < right) {
      if (word[left] !== word[right]) {
        newWorld = setCharAt(newWorld, left, word[right])
        newWorld = setCharAt(newWorld, right, word[left])
      }

      left ++;
      right --;
      wordArray[index] = newWorld
    }
  })

  return wordArray.join(' ')
};
```

最後回到了兩種哲學的對抗了，你是要快速解題還是套演算法找解答：
```js
// 1, 高速解題版, runtime 57.60%, memory 92.23%
const reverseWords = (string) => string.split(' ').map(word => word.split('').reverse().join('')).join(' ');

// 2, two pointers 認真面對版, runtime 9.01%, memory 5.12%
const reverseWords = (string) => {
  let chars = [...string];
  let start = 0;

  for (let i = 0; i < chars.length; i++) {
    if (chars[i] === ' ' || i === chars.length - 1) {

      // 找出每個詞的間隔，為避免下方 two pointers 不適用最後一個詞，要額外幫 end + 1
      let end = (i === chars.length - 1 && chars[i] !== ' ') ? i + 1 : i;

      // 對間隔前的文字進行 two pointers 並解構附值
      while (start < end) {
        [chars[start], chars[end - 1]] = [chars[end - 1], chars[start]];
        start++;
        end--;
      }

      // 交換完畢以後將 start 設為間隔的下一個字母，繼續跑下一輪
      start = i + 1;
    }
  }
  return chars.join('');
}
```

以結論來說高速解題版效能還是遠大於 two pointers 的版本，真是令我有點哭笑不得。