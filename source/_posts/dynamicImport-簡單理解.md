---
title: dynamicImport-簡單理解
date: 2024-12-10 16:58:17
tags:
---

## 重點

dynamic import 可以讓頁面在「需要時」才針對目標進行加載

## 說明

通常使用 dynamic import 會有幾個目的：

- 加快初始畫面的呈現
- 僅加載需要的模組，提升性能
- 依據條件加載不同模組
- 分割代碼，降低單一 bundle size

核心目的在於「依照需求才載入需要的模組，減少多餘的請求」

## 原理

- 在程式碼針對要動態載入的目標使用 `import()`，它本身會返回 `Promise`
- 若有找到模組，會下載並執行
- 最後會返回模組的內容
- 有需要可透過 `then` 或 `await` 獲取內容

## 限制及缺點

- 額外的 HTTP 請求，增加網路開銷
- 開發複雜度提升，會有處理非同步錯誤的需要
- 需要時才請求模組，會造成畫面延遲

雖然能提升效能，但必須評估使用時機，否則使用者體驗會有影響

## 使用情境

- 依照使用者狀態，加載不同版型
- 條件載入語言檔案
- 在大型應用中提高效能，避免多餘的載入

## 範例測試

- 使用 vite 和 React 及 TypeScript
- 預設背景不特別處理

### 動態引入元件

要注意 LargeList 不能跟主元件放在一起

**用來引入的元件**

```tsx LargeList.tsx
const items = Array.from({ length: 80000 }, (_, i) => `Item ${i + 1}`);

const LargeList = () => {
  // 計算加載時間
  useEffect(() => {
    onLoad();
  }, [onLoad]);

  return (
    <div
      style={{ height: '400px', overflowY: 'auto', border: '1px solid #ccc' }}
    >
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
};

export default LargeList;
```

**直接引入元件**

```tsx App.tsx
import './App.css';
import { useEffect } from 'react';
import LargeList from './LargeList';

function App() {
  const staticLoadStart = performance.now();

  // 計算載入時間
  const calcTime = (start: number, end: number) => {
    const passTime = end - start;
    const time = (passTime / 1000).toFixed(3);
    return time;
  };

  // 計算靜態內容加載時間
  useEffect(() => {
    const time = calcTime(staticLoadStart, performance.now());
    console.log(`靜態內容加載時間: ${time} 秒`);
  });

  return (
    <>
      <h1>Dynamic Import</h1>
      <div
        style={{
          display: 'flex',
          justifyContent: 'center',
          gap: '10px',
          marginBottom: '24px',
        }}
      >
        <LargeList
          onLoad={() => {
            const time = calcTime(staticLoadStart, performance.now());
            console.log(`動態內容加載時間: ${time} 秒`);
          }}
        />
      </div>
    </>
  );
}

export default App;
```

**使用 Dynamic Import 引入元件**

在 React 動態加載元件時，要配合 `Suspense` 和 `lazy` 來處理元件尚未被加載時的狀態

其中 `fallback` 綁定的讀取元件會一直到載入後才會切換過來

```tsx App.tsx
import './App.css';
import React, { useEffect } from 'react';

const LazyLargeList = React.lazy(() => import('./LargeList'));

function App() {
  const staticLoadStart = performance.now();

  // 計算載入時間
  const calcTime = (start: number, end: number) => {
    const passTime = end - start;
    const time = (passTime / 1000).toFixed(3);
    return time;
  };

  // 計算靜態內容加載時間
  useEffect(() => {
    const time = calcTime(staticLoadStart, performance.now());
    console.log(`靜態內容加載時間: ${time} 秒`);
  });

  return (
    <>
      <h1>Dynamic Import</h1>
      <div
        style={{
          display: 'flex',
          justifyContent: 'center',
          gap: '10px',
          marginBottom: '24px',
        }}
      >
        <React.Suspense fallback={<div>加載中...</div>}>
          <LazyLargeList
            onLoad={() => {
              const time = calcTime(staticLoadStart, performance.now());
              console.log(`動態內容加載時間: ${time} 秒`);
            }}
          />
        </React.Suspense>
      </div>
    </>
  );
}

export default App;
```

可以打開「開發人員工具」觀察兩種方式的渲染時間

這裡實際比對下來的時間如下：

**normal import**

- 完整內容加載時間: 1.020 ~ 1.073 秒

**dynamic import**

- 靜態內容加載時間: 0.002 - 0.006 秒
- 動態內容加載時間: 1.085 - 1.099 秒

可以觀察到，在使用 dynamic import 的情況下，靜態內容的載入相當迅速

這種方式可以在載入大量複雜資料時，先讓使用者看到一部分資料增加使用者體驗

至於動態內容可以看到，因為並非一開始就一併處理，因此花的時間就會比一般方式渲染還慢

總結來說，要在什麼樣的情況使用它增進使用者體驗，會需要看網站關注的重點（內容、效能、使用方式），評估後再來進行取捨

## 參考資料

[【Day.24】React 效能 - 用 lazy 和 Suspense 來動態載入元件](https://ithelp.ithome.com.tw/articles/10251342)
[Dynamic Import](https://www.patterns.dev/vanilla/dynamic-import)
