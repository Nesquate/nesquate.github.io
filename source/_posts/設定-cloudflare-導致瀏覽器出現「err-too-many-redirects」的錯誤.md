---
title: 設定 Cloudflare 導致瀏覽器出現「ERR_TOO_MANY_REDIRECTS」的錯誤
tags:
  - Caddy
  - Cloudflare
id: '71'
categories:
  - - 網路維運
comments: false
date: 2022-05-15 20:55:34
---

![](/2022/05/image-1024x560.png)

預設 Cloudflare 的 SSL 加密策略為「彈性」，改成「完整」就解決了...

都自己用 VPS 架設 WordPress 部落格了，VPS 的流量也算是寸土寸金等級的 (如果流量爆掉的話)，遲早上一下 CDN 以避免流量爆炸。

只是很奇怪，在 Cloudflare 開啟後台 Proxy 之後，Chrome 直接噴「ERR\_TOO\_MANY\_REDIRECTS」後就不給我瀏覽，架站設定初期還不知道這是什麼原因，所以之前是直接關掉 CDN 的。

但總不能一直關掉然後搞到流量爆炸吧，所以稍微認真去爬文一下，才在[一篇 GitHub Gist 上](https://gist.github.com/lopezjurip/5314252970cc94970058320ac78f490a)看到原來需要調整 Cloudflare 的設定......。
<!-- more -->
爬了文後發現，以 Cloudflare 的預設模式 (彈性) 來說，會在伺服器端連線到 HTTP (Port 80)，而訪客瀏覽則是由 Cloudflare 提供的 HTTPS (Port 443) 連線，但是因為 Caddy Server 這邊早就對 HTTP 做重定向 (Redirect) 了，所以 Cloudflare 連線到伺服器的那方就會連不到 HTTP，就變成無限重定向了。

![](/2022/05/image-1.png)

如同[這篇文](https://caddy.community/t/cloudflare-infinite-redirect-err-too-many-redirects/14192/3)說到的結果，但當時我對於到底該怎麼設定還是沒概念XD

所以如果確定伺服器有提供完整的 HTTPS，那就要改成「完整」或是「完整 (嚴格)」。

在撰文的同時，我想起來其實以前在自架 WordPress + Cloudflare (那時候還是 Shared Hosting) 我明明有注意到這點，但怎麼在這時候忘記了呢？而且明明之前用 yinyin.dev 也沒出現過這種問......。

啊對欸，.dev 網域強制 HSTS，Cloudflare 那邊就算是彈性的策略也會受到 HSTS 影響，難怪之前沒遇到Orz