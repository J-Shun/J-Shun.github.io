---
title: Hexo 初次建立
date: 2024-11-04 21:19:27
tags:
---

## 安裝 Hexo

- 使用指令：`npm install hexo-cli -g`

安裝結束後可透過：`hexo version` 或 `hexo v` 這裡種指令查看是否安裝成功

## 創建 Hexo 專案

- 使用指令：`hexo init <專案名稱>`

例如：

```
hexo init shun-blog
```

這項指令會在目前位置建立一個新的專案資料夾

- 進入資料夾
- 輸入資料 `npm i`

## 新增文章

- 使用指令：`hexo new [layout] <title>`

這項指令會在 `source\posts` 中建立一個 md 檔

之後便可在檔案中撰寫文章，使用的格式是 **Markdown**

不熟悉 Markdown 語法可以參考：[Markdown 教程](https://markdown.com.cn/)

## 本地啟動

- 使用指令:`hexo server`

這項指令會預設使用 `http://localhost:4000/` 作為路徑，進入後便可看到完成的靜態網站

## 部屬到 GitHub

- 在 GitHub 上建立專案，名稱為：`<username>.github.io`
- 回到專案，使用指令：`npm install hexo-deployer-git --save` 下載需要的套件
- 修改 `-config.yml` 中的 Deployment，內容如下：

```
deploy:
  type: git
  repo: https://github.com/<username>/<username>.github.io
  branch: gh-pages
```

其中 branch 可依照分支做調整

- 將此次的更動 commit 並 push 上 GitHub
- 使用指令：`hexo clean && hexo deploy`

部署完後即可在 `https://<username>.github.io/` 確認部屬的靜態網站

## 參考資料

[【學習筆記】如何使用 Hexo + GitHub Pages 架設個人網誌](https://hackmd.io/@Heidi-Liu/note-hexo-github)
[Hexo GitHub Pages](https://hexo.io/docs/github-pages)