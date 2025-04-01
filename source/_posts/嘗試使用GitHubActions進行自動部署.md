---
title: 嘗試使用 GitHub Actions 進行自動部署
date: 2025-04-01 11:20:36
tags:
---

## 說明

這裡沒有打算針對 GitHub Actions 進行介紹，只是想記下如何透過這項工具幫助我自動把網站部署到 GitHub 上

如果有興趣可以參考[這篇文章](https://ithelp.ithome.com.tw/articles/10262377)，內容挺不錯的

## 重點

我是使用 vite 進行處理的，流程如下

- 在 GitHub 專案中，選擇 `Settings` -> `Pages` ，並從 `Source` 中選擇 `GitHub Actions`
- 在專案的 `vite.config.ts` 中，加入 `base: '/專案路徑名稱/'` ，例如我在 GitHub 上弄的專案叫「 good-machine 」，就新增 `base: '/good-machine/'`
- 在專案 `src` 中新增 `.github` 資料夾，並在裡面新增 `workflows` 資料夾，再新增 `deploy.yml` 檔
- 在 yml 檔中新增指令內容，讓 main 分支合併後就會自動部署，內容大致如下（內容都是複製過來的）：

```yml
name: Deploy Vite to GitHub Pages

on: # 觸發 action 的時機。整體就是 main 被 push 後自動觸發
  push:
    branches:
      - main

permissions: # 設定權限，允許讀取程式碼、寫入 Github Pages 和 id-token
  contents: read
  pages: write
  id-token: write

jobs:
  build: # 建構
    runs-on: ubuntu-latest

    steps: # 步驟，功能如名稱所示
      - name: 檢出程式碼
        uses: actions/checkout@v4

      - name: 設定 Node.js 環境
        uses: actions/setup-node@v4
        with:
          node-version: 18

      - name: 安裝依賴與建置
        run: |
          npm install
          npm run build
          touch dist/.nojekyll

      - name: 上傳 artifact
        uses: actions/upload-pages-artifact@v3
        with:
          name: github-pages # 確保名稱為 "github-pages"
          path: dist

  deploy: # 部署，指定在 Ubuntu 最新版本環境中執行
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: 部署到 GitHub Pages
        uses: actions/deploy-pages@v4
```

yml 擋內容都有透過 chat 標上註解

處理完後，後續 main 分支一有變動， GitHub Actions 就會自動進行更新

可以從 `Settings` -> `Pages` 看到網站提供的網址

## 遇到的問題

主要是在處理時，碰上了版本上的問題

例如：

- 上傳 artifact 這段，我看到的內容是寫 `actions/upload-pages-artifact@v2` ，但實際上需要使用 `actions/upload-pages-artifact@v3`
- 部署到 GitHub Pages 這段，我看到使用 `actions/deploy-pages@v2` ，但實際上需要使用 `actions/deploy-pages@v4`

這些問題應該是因為後續有更新，並且彼此不相容造成的

這部分原先有請 ChatGPT 協助，但發現它不斷給錯誤的版本和名稱，導致我如何試錯都無法成功

是後續在 stack overflow 找到線索後才成功的（太依賴 AI 了，要檢討一下）

## 總結

透過 GitHub Actions 自動部署挺方便的，而且它本身還不只這項功能，之後應該會再找時間玩玩看

這篇主要是讓自己知道如何快速設定，並提醒自己當初是怎麼被一個簡單的東西搞死的

畢竟版本問題未來還是有可能發生，有個印象在至少後續排錯就輕鬆許多了
