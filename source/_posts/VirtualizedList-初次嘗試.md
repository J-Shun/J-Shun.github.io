---
title: VirtualizedList-初次嘗試
date: 2024-12-05 11:18:22
tags:
---

## 重點

Virtualize List 可針對過於龐大的 List 進行渲染上的處理

## 說明

當碰到所謂的「Long List」（可能為數千筆資料的 List 或 Table）

在不做處理的情況下，瀏覽器會直接渲染出完整，造成以下幾個效能問題：

- 畫面載入時間過長
- 大量 DOM 節點造成滾動不流暢、頁面卡頓（與 Reflow 和 Repaint 有關）
- 佔用過多的記憶體，導致瀏覽器崩潰

為了因應上述問題，才會有 Virtualized List 技術誕生

其核心在於「只渲染當前可見範圍內的列表項目」，從而大幅減少 DOM 節點，降低開銷

此技術也被稱之為「Windowing」

## 原理

- 使用帶有 `relative` 的 DOM 容器（ul）作為子元素的容器
- 使用一個 DOM 元素作為滾動的視窗
- 將子元素在容器內設定 `absolute`，並設置 `top` 、 `left` 、 `width` 、 `height` 達成固定高度及位置
- 根據滑動的距離計算出要視窗內渲染的元素

## 限制及缺點

- 需要針對 HTML 結構進行較多的處理
- 依賴子元素的高度計算，因此動態高度可能會導致滾動或渲染錯誤
- 過快地滾動會超出渲染速度造成空白或閃爍，造成體驗不夠順暢
- 要處理與滾動相關的狀態
- SEO 會因為子項目僅在可見範圍內而有所限制

## 使用情境

資料量龐大，但無法透過分頁處理時可以考慮，以下幾種可以考慮

- 聊天應用的聊天記錄
- 社交媒體的用戶推文
- 大型數據表格（Excel）
- 低性能設備的電商商品列表、地圖應用
- 股票實時價格表（即時更新的大型數據）

## 套件處理

- react-window：更快更輕、較好的版本，但稍微有些限制
- react-virtualized：架構特殊時的選擇

## 範例測試

- 使用 vite 和 react-window 做一個處理
- 預設背景不特別處理
- 要注意 FixedSizeList 的 style 必須帶，會影響渲染
- 記得將 CSS 加入 `border-box` ，因為 FixedSizeList 會需要計算高度

**無 Virtualized List**

```jsx
import './App.css';
const items = Array.from({ length: 40000 }, (_, i) => `Item ${i + 1}`);

const NormalList = () => (
  <div style={{ height: '400px', overflowY: 'auto', border: '1px solid #ccc' }}>
    {items.map((item, index) => (
      <div
        key={index}
        style={{ padding: '10px', borderBottom: '1px solid #eee' }}
      >
        {item}
      </div>
    ))}
  </div>
);

const App = () => (
  <>
    <h1>一般列表</h1>
    <NormalList />
  </>
);

export default App;
```

**有 Virtualized List**

```jsx
import './App.css';
import { FixedSizeList } from 'react-window';

const items = Array.from({ length: 40000 }, (_, i) => `Item ${i + 1}`);

const VirtualizedList = () => (
  <FixedSizeList
    height={400}
    itemCount={items.length} // 列表的總項目數
    itemSize={40} // 每個項目的高度（固定）
    width='100%'
  >
    {({ index, style }) => (
      <div
        style={{ ...style, padding: '10px', borderBottom: '1px solid #eee' }}
      >
        {items[index]}
      </div>
    )}
  </FixedSizeList>
);

const App = () => (
  <>
    <h1>處理過的列表</h1>
    <VirtualizedList />
  </>
);

export default App;
```

可以注意到，光是頁面最初的載入就有時間差

另外，當你在 VirtualizedList 進行選取，倘若超過畫面，則超過的部分會自動消失

最後這裡的程式碼用了兩個按鈕，個別將兩種列表渲染上畫面當中

可以比對一下兩者出現的速度

```jsx
import './App.css';
import { useState } from 'react';
import { FixedSizeList } from 'react-window';

const items = Array.from({ length: 40000 }, (_, i) => `Item ${i + 1}`);

const NormalList = () => (
  <div style={{ height: '400px', overflowY: 'auto', border: '1px solid #ccc' }}>
    {items.map((item, index) => (
      <div
        key={index}
        style={{ padding: '10px', borderBottom: '1px solid #eee' }}
      >
        {item}
      </div>
    ))}
  </div>
);

const VirtualizedList = () => (
  <FixedSizeList
    height={400}
    itemCount={items.length} // 列表的總項目數
    itemSize={40} // 每個項目的高度（固定）
    width='100%'
  >
    {({ index, style }) => (
      <div
        style={{ ...style, padding: '10px', borderBottom: '1px solid #eee' }}
      >
        {items[index]}
      </div>
    )}
  </FixedSizeList>
);

const App = () => {
  const [isShowVirtualizedList, setIsShowVirtualizedList] = useState(false);
  const [isShowNormalList, setIsShowNormalList] = useState(false);

  const handleToggleVirtualizedList = () => {
    if (!isShowVirtualizedList) setIsShowNormalList(false);

    setIsShowVirtualizedList(!isShowVirtualizedList);
  };

  const handleToggleNormalList = () => {
    if (!isShowNormalList) setIsShowVirtualizedList(false);
    setIsShowNormalList(!isShowNormalList);
  };

  return (
    <>
      <h1>列表比對</h1>
      <div
        style={{
          display: 'flex',
          justifyContent: 'center',
          gap: '10px',
          marginBottom: '24px',
        }}
      >
        <button onClick={handleToggleNormalList}>Render Normal</button>
        <button onClick={handleToggleVirtualizedList}>
          Render Virtualized
        </button>
      </div>
      {isShowNormalList && <NormalList />}
      {isShowVirtualizedList && <VirtualizedList />}
    </>
  );
};

export default App;
```

## 參考資料

[React-window | 有效地呈現大型列表與表格](https://medium.com/%E6%89%8B%E5%AF%AB%E7%AD%86%E8%A8%98/virtualize-long-list-with-react-window-95bac3673a91)
[Day 13 - 為什麼要用 Virtualized List](https://ithelp.ithome.com.tw/articles/10299969)
