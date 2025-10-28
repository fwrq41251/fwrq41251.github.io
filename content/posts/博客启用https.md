---
title: 博客启用https(支持hexo)
date: 2015-10-09
categories:
  - 互联网
tags:
  - https
  - hexo
---

1. [CloudFlare](https://www.cloudflare.com/)是目前唯一一家提供免费ssl的cdn,注册完设置好nameserver之后大约等一个多小时,
网站就可以使用https访问了.
[ssllabs](https://www.ssllabs.com)的测试结果:
![](/img/56179c7581b1d.jpg "result")
从图中可以看到,浏览器必须支持sni.
2. 然后就是让用户的http请求跳转到https.
在 `layout/_scripts` 目录下加一个 script,然后在 `layout/_layout.swig` 中 include 进去.

```
<script type="text/javascript">
    var host = "yoursite.com";
    if ((host == window.location.host) && (window.location.protocol != "https:"))
        window.location.protocol = "https";
</script>
```

3. 设置网站的url,让google等搜索引擎知道你正在使用ssl.
在`_config.yml`中增加:

```
url: https://www.yoursite.com   # with the https protocol
enforce_ssl: www.yoursite.com   # without any protocol
```

参考:
[Set Up SSL on Github Pages With Custom Domains for Free](https://sheharyar.me/blog/free-ssl-for-github-pages-with-custom-domains/)
[HTTPS下点击页面自动跳转HTTP](https://github.com/iissnan/hexo-theme-next/issues/123)
