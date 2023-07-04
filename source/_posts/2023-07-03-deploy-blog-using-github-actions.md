---
layout: post
title: 用GitHub Actions 自動化部署blog
date: 2023-07-03 22:58:05
categories:
  - Development
tags:
  - github
---

網路上有很多怎麼用hexo, hugo 等等架blog 的教學，這邊紀錄一下如何自動化部署到 GitHub page 。

# 概念

這需要一點點基本CICD 的概念。在偵測到有新的commit 時，自動開啟一台虛擬機器，執行預先寫好的script，完成部署。這也就是GitOps。

# 如何部署

官方教學：https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site。

在repo頁面上找到settings/pages，看到有兩種方式：

1. GitHub Actions (beta)
2. Deploy from a branch

## GitHub Actions (beta)

第一個是自己寫GitHub Actions 將靜態檔案打包好，上傳到GitHub page。比較進階一點，適合想自己寫所有步驟的人。想自己搞的人應該都有許多經驗了，可以參考：https://github.com/adityatelange/hugo-PaperMod/blob/4a924cef54081b61530a30bd69d442ae95f16561/.github/workflows/gh-pages.yml#L66-L80。

這邊需要用到2個actions：

- `actions/upload-pages-artifact`
- `actions/deploy-pages`

串接在build 後面，執行打包上傳跟發佈頁面。

## Deploy from a branch

第二個是把靜態檔案推上repo，可以放在另一個branch 或是在某個子資料夾裡，這個選項GitHub會幫你執行第一個選項裡的步驟。比較簡單，而且這個方法還可以切換branch 去檢查靜態檔案，方便debug，推薦。

你也可以在本地build 後，手動把靜態檔案推上GitHub。這邊的目標是從build 到deploy 都用GitHub actions 完成。

這個blog 的repo：https://github.com/cgjosephlee/blog 。Source code 放在 `main` branch，靜態檔案放在`ph-pages` branch。

以hexo 為例，在 `hexo generate` 後所產生的 `./public` 裡面就是網頁的靜態檔案，我想把這裡面的內容推到 `gh-pages` branch。之後我只要有更新post markdown file，GitHub actions 就會自動build and deploy。

因為是在虛擬機上執行，想像在乾淨的機器上，步驟寫成script 大概是：

```bash
git clone repo
cd repo
npm ci  # install dependencies
npm run build  # == hexo generate
cd public
# git clone ...
# git checkout gh-pages
# git add ...
# git commit ...
# git push ...
```

最後要推到 `gh-pages` 的步驟比較麻煩，還好有人已經寫成action 可以直接套用了。

- https://github.com/peaceiris/actions-gh-pages

寫成GitHub actions yaml：

```yaml
name: Deploy hexo pages to gh-pages branch

# 偵測main上面的改動，可以忽略特定檔案的改動
# 這裡忽略了drafts, images
on:
  push:
    branches:
      - main
    paths-ignore:
      - 'source/_drafts/**'
      - 'source/images/**'

jobs:
  pages:  # job name
    runs-on: ubuntu-latest  # 虛擬機image
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v3  # git clone
      - uses: actions/setup-node@v3  # setup node env
        with:
          node-version: 18
          cache: 'npm'
      - run: npm ci  # install dependencies
      - run: npm run build  # == hexo generate
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3  # push ./public to gh-pages branch
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}  # 不用修改，會自動取得token
          publish_dir: ./public
```

設定細節可以看 https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions。

把檔案放在repo 裡的 `.github/workflows/deploy.yaml`（檔名隨意），推上GitHub 後，就完成GitHub Actions 的設定了。之後偵測到改動，GitHub 就會自動執行yaml 裡的動作，然後推到 `gh-pages` branch。在Actions tab 裡面可以看到執行的結果。

部署完成後，就可以到GitHub page 看網頁囉。
