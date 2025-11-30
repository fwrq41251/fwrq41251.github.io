
---
title:  waydroid on omachy
date: 2025-11-30
tags:

- arch
- waydroid

slug:  waydroid-on-omachy
---

这两天新安装了omachy，试图在上面安装waydroid，我使用的是Vanilla版本（没有任何 Google 组件），但是碰到了没有网络的问题。

```
# 允许 Waydroid 流量
sudo ufw allow in on waydroid0
sudo ufw allow out on waydroid0
# 允许路由转发（关键）
sudo ufw default allow routed
# 重启 UFW
sudo ufw reload
```

然后玩明日方舟还需要让鼠标拖拽能模拟触摸屏的拖拽。

```
waydroid prop set persist.waydroid.fake_touch "*"
```
