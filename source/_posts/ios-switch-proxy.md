---
title: iOS快速设置代理
date: 2019-09-27 22:30:52
tags:
- 代理
- iOS
---

## 问题
手动设置wifi的步骤非常长：
1. 点击“设置”
2. 点击“无线局域网”
3. 点击已连接的wifi
4. 点击“配置代理”
5. 点击“手动”
6. 定位服务器输入框，输入ip
7. 定位端口输入框，输入port
8. 点击“存储”

## 解决
用iOS上的Shadowrocket和Mac上的Charles来快速设置代理
iOS和Mac在同一wifi

1. 启动Charles，假设代理地址为：192.168.0.100:8888
2. Shadowrocket设置`全局路由`为`代理`
3. Shadowrocket添加`HTTP`类型的节点
4. Shadowrocket打开连接

**添加`HTTP`类型的节点:**
![添加节点](http://upload-images.jianshu.io/upload_images/281540-3a3a4df908ac1248.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**结果:**
![结果](http://upload-images.jianshu.io/upload_images/281540-c94d3b52048e63e7.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)