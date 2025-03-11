---
title: React 發票掃描處理研究
date: 2025-03-11 15:34:03
tags:
---

## 重點

為了處理公司專案的電子發票掃描，在研究一些可能的方法和套件後，決定透過手刻的方式進行處理

本文對考量過了哪些套件、特點、碰到哪些問題做了一個紀錄

## 緣由

最近因為工作的關係，處理了電子發票的掃描功能

原先以為是個簡單的任務，沒想到卻在研究功能的這段期間踩了不少坑，因此做個簡單的紀錄

現有條件如下：

- 需要在 2 週內新增電子發票的 QRCode 掃描功能
- 希望掃描的相機畫面能夠配合設計稿做處理，並且 iOS Android 都要能使用
- 掃描電子發票時，發票上有左右兩個 QRCode，但一起掃到的時候只對左方的 QRCode 有反應
- 專案本身使用 CRA （Create React App）建立，並且使用 LINE LIFF

鑑於上述條件，我開始針對了幾種方式做測試和比較

## 掃瞄方式比較

### scanCodeV2

`scanCodeV2` 是 LINE LIFF 提供的相機掃描方式

實際說明可以看[官網](https://developers.line.biz/en/reference/liff/#scan-code-v2)

範例使用官網的寫法，只需要 `liff.scanCodeV2()` 即可，但相對受限較大

```js
liff
  .scanCodeV2()
  .then((result) => {
    // 邏輯處理
  })
  .catch((error) => {
    console.log('error', error);
  });
```

實測下來有下列影響：

- 會因為瀏覽器、作業系統不同，造成開出來的相機畫面不一樣（滿版、bottom sheet、打不開）
  但如果使用 LIFF 瀏覽器一律是 bottom sheet，雖然統一，但與畫面要求不符
- 掃描靈敏，但如果掃到非目標 QRCode，則會導致相機關閉，使用者體驗糟糕
- 畫面無法進一步客製化

基於上述原因，雖然 `scanCodeV2` 用起來方便，但無法滿足專案需求

### react-qr-barcode-scanner

這是一個外部套件

基本上我是直接遵照[官方 GitHub](https://github.com/jamenamcinteer/react-qr-barcode-scanner)的範例進行測試和研究

範例如下：

```jsx
<BarcodeScannerComponent
  width={500}
  height={500}
  onUpdate={(err, result) => {
    // 邏輯處理
  }}
/>
```

使用下來套件的問題在於：

- 掃描太難成功

套件本身容易使用，但在掃描測試時，針對電子發票怎麼掃都很難產生反應

要想掃描成功，需要針對其中一個 QRCode 做遮擋，並且將另外一個 QRCode 蓋滿幾乎整個鏡頭維持一小段時間才能夠成功

官網上也沒找到任何能針對這點改善的參數或處理，因此這個套件也無法滿足需求

### @yudiel/react-qr-scanner

第三個方式一樣是外部套件，詳細內容可以看[官方 GitHub](https://github.com/yudielcurbelo/react-qr-scanner)

GitHub 上除了簡單的範例之外，也有提供 [Demo](https://yudielcurbelo.github.io/react-qr-scanner/?path=/story/scanner--scanner) 讓使用者直觀理解它能辦到的效果，並且深入研究也能找到其他設定方法

套件本身使用起來簡單，掃描起來也很精準，也能夠設定成針對複數 QRCode 時有反應

上述幾點都基本上滿足了這次任務的條件

不過，套件本身也是有幾項問題點：

- 掃描靈敏度有些詭異，放在空無一物的桌字上掃不過，但拿在手、放在有雜物的地板上、黑色背景的地方卻可以
- 官網範例太簡潔，如果不看 Demo 會不知道它有一些行為可以設定來增加使用者體驗，並且不深入 GitHub 更不會知道如何去做設定
- 在 CRA 情況下引入套件時，報錯顯示無法解析套件
- 套件本身在放專案跑時，被 CSP 擋下來，發現套件有另外再加載外部的 wasm 檔，來源是 `https://fastly.jsdelivr.net`

第 1 點勉強可以因為是環境問題而忽視，2, 3 點後續也是有找到處理的方法

但第 4 點本身就有點致命了，在與後端討論過後，基於安全性的理由，最後放棄使用這個套件

但研究都研究了，所以我還是會將 2, 3 點研究時發現的東西寫上來

#### 套件詳細設定

如果有使用 Demo，可以發現它的處理方式能讓使用者在掃到目標時冒出字幕、或出現外框，還有一些功能設定

但如果單看 GitHub 的 Usage 和下面的介紹欄位，會發現它沒有提供範例

我先是看到一個日本網站的[範例用法](https://dev.classmethod.jp/articles/customizing-scan-ui-with-iscannercomponents-in-yudiel-react-qr-scanner/)後，回頭往 GitHub 深入調查才知道那些效果是如何做出來的

套件本身有提供 `components` props 讓使用者去做一些調整，這部分 GitHub 也有，而具體的使用方式如下：

```jsx
<Scanner
  onScan={handleScan}
  formats={[
    'qr_code'
    'micro_qr_code',
  ]}
  allowMultiple={true}
  components={{
    tracker: customTracker, // 掃描時的客製化提示
    audio: true, // 音效
    onOff: true, // 開關
    zoom: true, // 縮放
    finder: false, // finder
    torch: true, // 閃光燈
  }}
/>
```

除了使用方式之外，重點在於 `tracker` 的處理

從 GitHub 中的 [overlays.ts](https://github.com/yudielcurbelo/react-qr-scanner/blob/main/src/misc/overlays.ts) 其實可以看到它在 Demo 中所做的處理

只要使用它所寫好的這些 function，就能模擬出一樣的效果

例如我想把它的外框線拿來用，就可以做一個簡單的修改：

```jsx
/* 掃描時，對到 QRCode 會冒出的方形黃線 */
function boundingBox(detectedCodes, ctx) {
  detectedCodes.forEach(({ boundingBox: { x, y, width, height } }) => {
    ctx.lineWidth = 2;
    ctx.strokeStyle = 'yellow';
    ctx.strokeRect(x, y, width, height);
  });
}

// ...some code

<Scanner
  onScan={handleScan}
  formats={['qr_code', 'micro_qr_code']}
  components={{
    audio: false,
    onOff: false,
    torch: false,
    zoom: false,
    tracker: boundingBox,
  }}
  allowMultiple
  scanDelay={300}
  paused={isPauseCamera}
/>;
```

像這樣，當掃描成功時，就會在畫面上看到相機會在 QRCode 上冒出邊框進行標示

它的處理法是使用 canvas 的 ctx，因此稍微懂一些 canvas 的人應該可以做出其他客製化的修改

#### CRA 下引入套件時，報錯顯示無法解析

這個問題是我當初測試時沒注意到的

因為需求本身是讓手機掃描 QRCode，因此需要實際部署後使用手機相機先進行測試，所以我當初沒有直接開公司專案測試

但這造成了一個問題，那就是在我的專案上沒問題，但在公司專案上只要引入就開始報錯

錯誤內容很長，但內容差不多，大致如下：

```
RROR in ./node_modules/@yudiel/react-qr-scanner/dist/index.esm.mjs 2:0-80
Module not found: Error: Can't resolve 'webrtc-adapter/dist/chrome/getusermedia' in '/Users/user/Desktop/liff/node_modules/@yudiel/react-qr-scanner/dist'
Did you mean 'getusermedia.js'?
BREAKING CHANGE: The request 'webrtc-adapter/dist/chrome/getusermedia' failed to resolve only because it was resolved as fully specified
(probably because the origin is strict EcmaScript Module, e. g. a module with javascript mimetype, a '*.mjs' file, or a '*.js' file where the package.json contains '"type": "module"').
The extension in the request is mandatory for it to be fully specified.
Add the extension to the request.
```

會產生這個錯誤，主要是模組沒有正確解析檔名，導致找不到內容

我在測試當下使用的 Vite 會自動處理這些問題，所以根本不會注意到

要在 CRA 的專案中解決這個問題，就需要修改 Webpack 讓它能夠正確讀到檔案

但接下來就更有趣了

雖然知道 Webpack 可以處理，但是最靠北的卻是在專案裡面找不到 Webpack 的設定檔

那是因為在 CRA 開啟來的專案中，Webpack 一開始是被隱藏的，需要手動做設定（eject）才可以開始更改

並且此動作不可逆，使用之後 Webpack 的處理就變自訂了

幸運的是，公司專案的前人有使用 `react-app-rewired` 這套工具，只要用了它就可以在不更改現有條件下，針對配置去進行改寫

安裝和使用方式我就不多說了，可以看看[這篇技術文章說明](https://ithelp.ithome.com.tw/articles/10293213)

我來直接講解如何處理掉我目前的問題：

```js config-overrides.js
module.exports = function override(config, env) {
  // 參數中的config 就是預設的 webpack config

  // 確保找到 babel-loader 實例
  const babelLoader = config.module.rules
    .find((rule) => rule.oneOf)
    .oneOf.find((r) => r.loader && r.loader.includes('babel-loader'));

  if (babelLoader) {
    // 確保 babel-loader 處理 .mjs 檔案
    babelLoader.test = /\.(js|mjs|jsx|ts|tsx)$/;
  }

  // 新增 .mjs 支援
  config.resolve.extensions.push('.mjs');

  // 添加 Webpack 規則，確保 .mjs 檔案正確解析
  config.module.rules.push({
    test: /\.mjs$/,
    include: /node_modules/,
    type: 'javascript/auto', // 避免 Webpack 將 .mjs 解析成 ES Module
  });
  config.resolve.alias = {
    ...config.resolve.alias,
    'webrtc-adapter': path.resolve(__dirname, 'node_modules/webrtc-adapter'),
  };
  config.resolve.fullySpecified = false;

  return config;
};
```

我是透過上面的設定讓專案能夠正確解析 mjs 檔案，才成功讓套件能夠運行在公司專案上

雖然最後因為 CSP 問題最後還是沒有使用就是了 QQ

### qr-scanner

一樣是外部套件，但經歷 `@yudiel/react-qr-scanner` 的問題之後，我這次直接用一樣的環境進行測試

實際測試結果，它和 `@yudiel/react-qr-scanner` 一樣有呼叫外部腳本，會被 CSP 檔下來，因此直接淘汰

## 手刻掃瞄器

上面看過了那麼多套件，發現問題真的太多了

居然都沒辦法處理，那就只好~強迫上頭更改需求~手刻一個了

雖然說是手刻，但範圍就到開啟掃瞄器並獲取資訊，解析 QRCode 還是需要套件協助

這方面我交給 `jsQR` 處理，詳細可看 [GitHub](https://github.com/cozmo/jsQR)

只要將 QRCode 的圖片傳給它，就能夠解析出相關資訊

接下來，要在前端中開啟相機進行掃描，我們會需要 `<video>` 和 `<canvas>` 這兩個 HTML tag

`<video>` 會用來開啟相機，`<canvas>` 則是會將 canvas 拿到的畫面丟給 `jsQR` 解析出 QRCode 資訊

有了 `<video>` ，還需要透過 `navigator.mediaDevices.getUserMedia` 這個方法配合來去處理

`navigator.mediaDevices.getUserMedia` 會徵求使用者相機權限，並在允許後返回相關的物件資料

整體來說流程如下，以 react 為例：

- 準備好 `<video>` 和 `canvas` ，用 useRef 好抓到 DOM
- 點擊開啟相機按鈕時時，透過 `navigator.mediaDevices.getUserMedia` 從使用者要求特定權限，獲取返回的資料
- 將獲取的資料丟給 `<video>` 的 srcObject（在沒有支援的情況下，要使用 `localVideo.src = window.URL.createObjectURL(localStream)` 處理）
- 透過 `play()` 方法讓相機運作（需要這個動作是因為掃描後有使用 `pause()` 方法暫停，如果沒有 play() 也會自行運作）

實際使用下來如下，使用 react 處理：

```jsx
import { useState, useEffect, useRef } from 'react';

export default function ScanRegister({ setPageLoading }) {
  const [isShowCamera, setIsShowCamera] = useState(false);
  const videoRef = useRef(null);
  const canvasRef = useRef(null);
  const intervalRef = useRef(null);
  const streamRef = useRef(null);

  const handleCameraOpen = async () => {
    setIsShowCamera(true);
    const deviceStream = await navigator.mediaDevices.getUserMedia({
      // 指定使用畫面，並且用後置鏡頭
      video: {
        facingMode: 'environment',
      },
      audio: false,
    });
    streamRef.current = deviceStream;
    videoRef.current.srcObject = deviceStream;
    videoRef.current.play(); // 可能不需要

    // 初始化 canvas，並設定 willReadFrequently 幫助提升讀取的效能
    const canvas = canvasRef.current;
    const ctx = canvas.getContext('2d', { willReadFrequently: true });

    // 使用 interval 持續性處理資料，以 0.1 秒為單位。頻率太頻繁網頁會爆
    intervalRef.current = setInterval(() => {
      // 設定 canvas 和 video 同大小，並將 video 拿到的畫面丟上去
      canvas.width = videoRef.current.videoWidth;
      canvas.height = videoRef.current.videoHeight;
      ctx.drawImage(videoRef.current, 0, 0);

      // 將 canvas 畫面轉為圖片，並提供給 jsQR 做解析
      const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
      const code = jsQR(imageData.data, imageData.width, imageData.height);

      if (code) {
        // ...some code，針對解析出來的資料做處理

        // 清除資料、暫停相機
        clearInterval(intervalRef.current);
        intervalRef.current = null;
      }
    }, 100);
  };

  const handleCameraClose = () => {
    setIsShowCamera(false);
    if (intervalRef.current) {
      // 關掉 interval 並清除 ref
      clearInterval(intervalRef.current);
      intervalRef.current = null;
    }
    if (streamRef.current) {
      // 把串流暫停，並清除 ref
      streamRef.current.getTracks().forEach((track) => track.stop());
      streamRef.current = null;
    }
  };

  // 卸載元件
  useEffect(() => {
    return () => {
      handleCameraClose();
    };
  }, []);

  return (
    <>
      <video
        ref={videoRef}
        autoPlay
        playsInline
        style={{ display: `${isShowCamera ? 'block' : 'none'}` }}
      >
        <track kind='captions' />
      </video>
      <canvas ref={canvasRef} style={{ display: 'none' }} />
      <button onClick={handleCameraOpen}>開啟相機</button>
      <button onClick={handleCameraClose}>關閉相機</button>
    </>
  );
}
```

透過上面的方法，就可以正常開啟手機相機開始進行掃描了

用這種方式處理，目前畫面上出現的會是 `<video>` 的畫面

如果想要針對掃描後的行為進行加工，例如掃到 QRCode 就出現提示用的外框線，就要從渲染 `<video>` 改為渲染 `<canvas>`

之後透過 ctx 去處理畫面

範例如下：

```jsx
import { useState, useEffect, useRef } from 'react';

export default function ScanRegister({ setPageLoading }) {
  const [isShowCamera, setIsShowCamera] = useState(false);
  const videoRef = useRef(null);
  const canvasRef = useRef(null);
  const intervalRef = useRef(null);
  const streamRef = useRef(null);

  const handleCameraOpen = async () => {
    setIsShowCamera(true);
    const deviceStream = await navigator.mediaDevices.getUserMedia({
      video: {
        facingMode: 'environment',
      },
      audio: false,
    });
    streamRef.current = deviceStream;
    videoRef.current.srcObject = deviceStream;

    const canvas = canvasRef.current;
    const ctx = canvas.getContext('2d', { willReadFrequently: true });

    intervalRef.current = setInterval(() => {
      canvas.width = videoRef.current.videoWidth;
      canvas.height = videoRef.current.videoHeight;
      ctx.drawImage(videoRef.current, 0, 0);

      const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
      const code = jsQR(imageData.data, imageData.width, imageData.height);

      if (code) {
        const { location } = code;

        if (!location) return;

        // 畫出 QRCode 的邊框
        ctx.strokeStyle = '#FAAD14';
        ctx.lineWidth = 5;
        ctx.beginPath();
        ctx.moveTo(location.topLeftCorner.x, location.topLeftCorner.y);
        ctx.lineTo(location.topRightCorner.x, location.topRightCorner.y);
        ctx.lineTo(location.bottomRightCorner.x, location.bottomRightCorner.y);
        ctx.lineTo(location.bottomLeftCorner.x, location.bottomLeftCorner.y);
        ctx.closePath();
        ctx.stroke();

        // ...some code，針對解析出來的資料做處理

        clearInterval(intervalRef.current);
        intervalRef.current = null;
      }
    }, 100);
  };

  const handleCameraClose = () => {
    setIsShowCamera(false);
    if (intervalRef.current) {
      clearInterval(intervalRef.current);
      intervalRef.current = null;
    }
    if (streamRef.current) {
      streamRef.current.getTracks().forEach((track) => track.stop());
      streamRef.current = null;
    }
  };

  useEffect(() => {
    return () => {
      handleCameraClose();
    };
  }, []);

  return (
    <>
      <video ref={videoRef} autoPlay playsInline style={{ display: 'none' }}>
        <track kind='captions' />
      </video>
      <canvas
        ref={canvasRef}
        style={{ display: `${isShowCamera ? 'block' : 'none'}` }}
      />
      <button onClick={handleCameraOpen}>開啟相機</button>
      <button onClick={handleCameraClose}>關閉相機</button>
    </>
  );
}
```

如此一來，每當掃描到目標 QRCode 時，就會出現一個方形外框提示，提升使用者體驗

但使用 `<canvas>` 替代 `<video>` 渲染有個小缺點

那就是畫面是每 0.1 秒從 `<video>` 傳到 `<canvas>` 渲染出來的，流暢度相比單純使用 `<video>` 還低了一些

不過關鍵的靈敏度不減，是否需要這層處理就看狀況了

[Demo](https://yudielcurbelo.github.io/react-qr-scanner/?path=/story/scanner--scanner)
[參考範例](https://dev.classmethod.jp/articles/customizing-scan-ui-with-iscannercomponents-in-yudiel-react-qr-scanner/)
[參考用法](https://github.com/yudielcurbelo/react-qr-scanner/blob/main/stories/Scanner.stories.tsx)
[參考設定](https://github.com/yudielcurbelo/react-qr-scanner/blob/main/src/misc/overlays.ts)
