---
layout: post
title: "openId登录流程"
date: 2016-09-10 13:50:19 +0800
comments: true
categories: 认证授权
---
网易基于openId认证登录机制


1.User访问App

2.App向OpenId Server发送关联请求，post请求，请求参数都是固定的

3.OpenId Server响应关联请求，包含assoc_handle和mac_key,assoc_handle用于认证请求,mac_key用于校验

4.App生成用户认证URL，这里设置认证成功跳转URL（访问OpenId Server）

5.App响应用户1中的请求，返回302，通知User跳转认证URL

6.User访问认证URL，即我们经常见到的那个授权页面

7.如果认证成功，OpenId Server返回302，通知User跳转App URL

8.User跳转App URL

9.App 用步骤3获得的mac_key进行校验，校验成功即认证成功。
