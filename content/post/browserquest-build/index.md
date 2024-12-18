---
title: 搭建rpg小游戏 browserquest
description: 搭建rpg小游戏 browserquest
date: 2024-11-21 20:02:40+0000
categories:
  - linux
tags:
  - docker
---

## 搭建rpg小游戏 browserquest

```bash
docker run -d \
-p 8000:8000 \
-p 8787:8787 \
--name rpg \
-e HOST_IP=your-ip \
registry.cn-guangzhou.aliyuncs.com/welldene/games:rpg_game
```

浏览器输入`your-ip:8787`


