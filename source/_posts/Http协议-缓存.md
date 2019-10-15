---
title: Http协议--缓存
date: 2019-09-24 22:56:16
tags:
---

!['Web缓存流程图'](/images/Web缓存流程图.png)

### If-None-Match/Etag

Etag 为服务器响应时所携带，是此资源的唯一标识，当资源做过修改时，其值也会修改
If-None-Match 为上次请求服务器响应时的 Etag 值，与服务器新的 Etag 做比较。

### If-Modified-Since/Last-Modified

Last-Modified 为服务器响应时所携带，是此资源的上一次修改时间
If-Modified-Since 为上次请求服务器响应时的 Last-Modified 值，与服务器新的 Last-Modified 做比较。
