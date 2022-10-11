---
title: 從 WordPress 到 Hexo 的搬遷紀錄
date: 2022-10-11 15:09:26
tags: [WordPress, Hexo]
categories: 紀錄
---

這邊記錄一下我是怎麼從 WordPress 搬到 Hexo，並搭配 GitHub Actions 以及 GitHub Pages 作為我目前的託管方案。

之所以要搬遷的原因是，最近也沒什麼用途會需要用到 VPS，所以就搬家了，也順便省錢。

### hexo-migrator-wordpress

這次一樣使用 [hexo-migrator-wordpress](https://github.com/hexojs/hexo-migrator-wordpress)，但我這次才知道有 `--import-image` 這個參數可以用......。

這下工作量就減少不少，只要修改圖片的超連結就好。用完之後基本上文章、頁面、分類與標籤都順利搬遷過來了。

不過自 WordPress 5.x/6.x (不太確定) 引入的新版主題編輯器好像會讓 Hexo 匯入之後產出一些垃圾空白頁面，這部分除了手動移除之外好像別無他法。

### NexT 主題

這是我第二次使用 [NexT 主題](https://theme-next.js.org/)，這部分就依照自身的喜好修改。

詳細設定可以在[我的部落格的 GitHub Repo](https://github.com/Nesquate/nesquate.github.io)上看到，這部分應該沒有什麼好說明的。

目前我的部落格還沒決定好要用哪種留言系統，所以不開放留言。

### 設定 GitHub Actions

這部分參考[Hexo 官方教學](https://hexo.io/docs/github-pages)所填寫。

其實可以等於照抄了XD，但我可能要再研究一下看`peaceiris/actions-gh-pages@v3`能不能不要用我的名字做 Commit......。

### 結尾

好久沒用 Hexo 了，這次回來用 Hexo 相關操作有點遲鈍，不過很快就抓到感覺了。

把部落格方案轉移成 Hexo 之後，VPS 就可以直接銷毀了，至於 Vultr 剩下的餘額，可能看看之後能不能建立一個 VPN 起來用吧 (但好像很多服務都早就封鎖 VPS 服務的 IP 了)。