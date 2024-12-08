---
title: infiniteScroll-初次嘗試
date: 2024-12-08 21:31:27
tags:
---

## 重點

infinite scroll 是讓頁面能夠無限載入新內容的技術，常見於社群軟體中（Meta、X 等）

## 說明

通常使用 infinite scroll 會有幾個目的：

- 提升使用者體驗
- 提升使用者的留存率
- 改善行動裝置上的操作
- 降低初始載入時間

其核心目的在於「降低操作負擔、提升沉浸感、提高性能、兼顧商業需求」

## 原理

### 使用事件監聽

- 準備一個 container 容器，用來放置內容
- 透過監聽 scroll 事件，監測容器畫面是否到達容器底部（或接近底部）
- 到達特定位置時呼叫 API，拿取後面的資料

### 使用 Intersection Observer API

- 在底部準備一個要用來觀察的觸發器（這裡簡單用 div 放 loading 來處理）
- 建立 IntersectionObserver，並在 callback 中設定成元素進入可視範圍時，呼叫 API 拿取後面內容
- 觀察目標物件（在這裡是 loading）
- 若不再使用，要清除觀察器

### 選擇

使用 Intersection Observer API 會相對有優勢，有幾個原因：

- 事件監聽會在滾動時不斷觸發，有效能上的影響
- Intersection Observer API 可以做出更精確的控制（例如從可視範圍的百分比決定觸發時機）

## 限制及缺點

- DOM 節點不斷增加會導致性能問題
- 較難定位之前瀏覽過的內容

## 範例測試

- 使用 vite 和 React 及 TypeScript
- 預設背景不特別處理

**事件監聽**

```jsx
import './App.css';
import { useEffect, useState, useRef } from 'react';

const item = Array.from({ length: 10 }, (_, index) => index + 1);

// 事件監聽
function App() {
  const containerRef = useRef() as React.MutableRefObject<HTMLDivElement>;

  const [data, setData] = useState(item);
  const [position, setPosition] = useState(1); // 通常會與 API 做搭配，用來記錄載入資料位置

  useEffect(() => {
    const handleScroll = () => {
      const { scrollTop, scrollHeight, clientHeight } = containerRef.current;
      // 這段判斷會決定資料載入的時機
      if (scrollTop + clientHeight >= scrollHeight) {
        // 模擬 API 請求的延遲
        setTimeout(() => {
          const newData = Array.from(
            { length: 10 },
            (_, index) => index + 1 + position * 10
          );
          setPosition((prev) => prev + 1);
          setData((prev) => [...prev, ...newData]);
        }, 500);
      }
    };

    const currentRef = containerRef.current;
    currentRef.addEventListener('scroll', handleScroll);
    return () => currentRef.removeEventListener('scroll', handleScroll);
  }, [position]);

  return (
    <>
      <h1>Infinite Scroll</h1>

      <div
        ref={containerRef}
        style={{
          width: '400px',
          height: '600px',
          overflowY: 'scroll',
        }}
      >
        {data.map((item, index) => (
          <div
            key={index}
            style={{
              display: 'flex',
              justifyContent: 'center',
              alignItems: 'center',
              height: '250px',
              border: '1px solid black',
              marginBottom: '24px',
              fontSize: '28px',
            }}
          >
            {item}
          </div>
        ))}
      </div>
    </>
  );
}

export default App;
```

**Intersection Observer**

```jsx
import './App.css';
import { useEffect, useState, useRef } from 'react';

const item = Array.from({ length: 10 }, (_, index) => index + 1);

// IntersectionObserver
function App() {
  const loadingRef = useRef() as React.MutableRefObject<HTMLDivElement>;

  const [data, setData] = useState(item);
  const [position, setPosition] = useState(1); // 通常會與 API 做搭配，用來記錄載入資料位置

  useEffect(() => {
    const observer = new IntersectionObserver((entries) => {
      // 當 sentinel 元素進入可視範圍時載入更多內容
      if (entries[0].isIntersecting) {
        // 模擬 API 請求的延遲
        setTimeout(() => {
          const newData = Array.from(
            { length: 10 },
            (_, index) => index + 1 + position * 10
          );
          setPosition((prev) => prev + 1);
          setData((prev) => [...prev, ...newData]);
        }, 500);
      }
    });

    const currentRef = loadingRef.current;

    // 開始觀察 sentinel 元素
    if (currentRef) {
      observer.observe(currentRef);
    }

    // 清理觀察器
    return () => {
      if (currentRef) {
        observer.unobserve(currentRef);
      }
    };
  });

  return (
    <>
      <h1>Infinite Scroll</h1>

      <div
        style={{
          width: '400px',
          height: '600px',
          overflowY: 'scroll',
        }}
      >
        {data.map((item, index) => (
          <div
            key={index}
            style={{
              display: 'flex',
              justifyContent: 'center',
              alignItems: 'center',
              height: '250px',
              border: '1px solid black',
              marginBottom: '24px',
              fontSize: '28px',
            }}
          >
            {item}
          </div>
        ))}
        <div ref={loadingRef}>Loading more...</div>
      </div>
    </>
  );
}

export default App;
```

兩種方式基本上很接近，比較明顯的部分在於，Intersection Observer 是在底部放置 Loading more 作為呼叫 API 用的觸發器
