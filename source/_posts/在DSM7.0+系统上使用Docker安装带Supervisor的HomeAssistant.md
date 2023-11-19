---
title: 在DSM7.0+系统上使用Docker安装带Supervisor的HomeAssistant
date: 2022-04-03 19:13:38
tags: ["HomeAssisstant", "Synology", "Docker", "DSM7.0"]
---

![homeassisstant](/images/20220403192539.png)

![homeassisstant with supervisor on dsm 7.0](/images/20220403191505.png)

<!--more-->

## 操作步骤

首先创建一个文件夹用于存放 homeassisstant 的文件，并赋予 Everyone 完全控制的权限

![homeassisstant folder](/images/20220403191827.png)

然后在[HASSIO (Home Assistant Supervised) for Synology DSM 7 (Unsupported Installation but allows updates and fully functional)](https://gist.github.com/maeneak/851e883eca7cddd7114f7eaed201ca9d)下载提供的 docker compose 配置文件，并修改路径。

![docker compose](/images/20220403192248.png)

启动即可

```bash
docker-compose up --detach
```

![observer](/images/20220403192455.png)

## 额外问题

重新进行`docker-compose down`和`docker-compose up --detach`后，遇到了没有由`qemux86-64-homeassistant`镜像启动的名称为`homeassistant`容器的问题。
![homeassistant container](/images/20220501125803.png)
通过查看`hassio_superviosr`日志，类比其他容器的启动日志片段，尝试删除`ghcr.io/home-assistant/qemux86-64-homeassistant`，再次执行`docker-compose up --detach`，问题解决。
问题日志：
![superviosr log](/images/20220501130305.png)
删除镜像后再次启动的日志：
![superviosr log](/images/20220501130704.png)
![superviosr log](/images/20220501130735.png)