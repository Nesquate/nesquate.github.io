---
title: MediaWiki 搭 Cloudflare ，load.php 出現 520 Error 的問題
tags: []
id: '111'
categories:
  - - uncategorized
---

根據 [Cloudflare 文章內對於 5xx Error 的說明](https://support.cloudflare.com/hc/zh-cn/articles/115003011431-Cloudflare-5XX-%E9%94%99%E8%AF%AF%E6%95%85%E9%9A%9C%E6%8E%92%E9%99%A4#520error) 以及 [MediaWiki 上對 Cloudflare 設定的說明](https://www.mediawiki.org/wiki/Manual:CloudFlare) 來看，應該只是單純的沒有設定好例外 IP。

MediaWiki 版本 >= 1.34 的有支援 CDN IP 排除的選項，只要在 `LocalSettings.php` 裡面加入這兩行：

```php
$wgUseCdn = true;
$wgCdnServerNoPurge = array("173.245.48.0/20","103.21.244.0/22","103.22.200.0/22","103.31.4.0/22","141.101.64.0/18","108.162.192.0/18","190.93.240.0/20","188.114.96.0/20","197.234.240.0/22","198.41.128.0/17","162.158.0.0/15","104.16.0.0/13","104.24.0.0/14","172.64.0.0/13","131.0.72.0/22","2400:cb00::/32","2606:4700::/32","2803:f800::/32","2405:b500::/32","2405:8100::/32","2a06:98c0::/29","2c0f:f248::/32");
```

其中 `$wgCdnServerNoPurge` 是參考 [Cloudflare IP Ranges](https://www.cloudflare.com/zh-tw/ips/) 頁面去添加的，所以要記得隨時檢查。