---
title: intersectionObserver-初次嘗試
date: 2024-12-09 21:45:37
tags:
---

## 重點

用來監控目標元素（單個或多個）是否進入畫面的可視範圍，並觸發後續行為

主要解決了過去 scroll 相關的處理：

- 不需要不停監聽 scroll 事件，最佳化效能
- 可監控多個目標
- 設定方便，並且可做相較精細的操作

## 原理

- 創建 intersection observer 實例，並定義選項和 callback
- 使用 `observe` 方法觀察目標 DOM 元素
- 若不再需要，要使用 `unobserve` 方法清除特定觀察器、或用`disconnect` 方法停止所有觀察器，釋放空間

## 限制及缺點

- 舊版瀏覽器可能無法使用

## 使用情境

以下幾種狀況都可以考慮透過 intersection observer 處理

- 大量圖片的 Lazy Loading
- 大量內容的 Infinite Scroll
- 根據畫面位置處理動畫的滾動視差
- 影片的自動播放（Steam 遊戲影片的自動播放感覺有用到）
- 曝光率追蹤（GA + GTM 的追蹤曝光感覺有用到）

## 範例測試

- 使用 vite + React + TypeScrip
- 預設背景不特別處理
- 要注意 LazyImage 要確實帶入高度，避免最開始的狀態已經全部可見

**透過 intersection observer 處理圖片 lazy load**

```tsx
import './App.css';
import { useState, useEffect, useRef } from 'react';

const LazyImage = ({ src, alt }: { src: string; alt: string }) => {
  const imageRef = useRef<HTMLImageElement>(null);
  const [isLoaded, setIsLoaded] = useState(false);

  const options = {
    root: null, // 設定相對哪個範圍進行觀察，預設是 viewport
    rootMargin: '10px', // 設定觀察範圍的邊距，支援 px 和 %
    threshold: 0, // 當元素與範圍交集達到某比例時觸發回調
  };

  useEffect(() => {
    const observer = new IntersectionObserver((entries) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting && imageRef.current) {
          setIsLoaded(true);
          observer.unobserve(imageRef.current); // 停止觀察
        }
      });
    }, options);

    const image = imageRef.current;

    if (image) {
      observer.observe(image);
    }

    return () => {
      if (image) {
        observer.unobserve(image);
      }
    };
  }, [src]);

  return (
    <img
      ref={imageRef}
      src={isLoaded ? src : ''}
      alt={alt}
      style={{
        // 高度很重要，如果沒設高度，元素最初會全部可見，導致 Lazy Load 失去意義
        minHeight: '400px',
        width: '100%',
        objectFit: 'cover',
      }}
    />
  );
};

// 用 Lorem Picsum 處理圖片基礎設定
// 這裡高度設定成和 LazyImage 中一致，避免瀏覽器高度跳動
const imageUrl = 'https://picsum.photos/400/400';
const imageArray = Array.from({ length: 10 }, (_, index) => {
  return {
    src: `${imageUrl}?random=${index + 1}`,
    alt: `random image ${index + 1}`,
    id: index + 1,
  };
});

function App() {
  return (
    <>
      <h1>Intersecton Observer</h1>
      <div
        style={{
          display: 'flex',
          justifyContent: 'center',
        }}
      >
        <div
          style={{
            width: '400px',
            display: 'flex',
            flexDirection: 'column',
            alignItems: 'center',
            justifyContent: 'center',
            gap: '24px',
          }}
        >
          {imageArray.map((image) => (
            <LazyImage
              key={image.id}
              src={image.src}
              alt={image.alt}
            />
          ))}
        </div>
      </div>
    </>
  );
}

export default App;
```

option 如果不代入就會使用預設值

可以調整 option 內的參數觀看會產生什麼效果
