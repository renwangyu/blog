---
title: 由html2canvas截图空白引出的img跨域和canvas“污染”
date: 2023-12-02 12:40:31
tags:
- 跨域
categories:
- 前端
top_img: /img/post/canvas-tainted.jpeg
cover: /img/post/canvas-tainted.jpeg
---

# 前言
工作中，有项目因为有截图上传的功能，用的 [html2canvas](https://www.npmjs.com/package/html2canvas) 这个库，一直没什么问题，直到最近测试环境迁移，oss也跟着做了迁移，域名什么也做了，发现测试环境上，截屏上传的竟然都是空白图。虽然生产是好的，但是总不能让测试环境变坏吧，因此做了一些调研。
在排除了功能代码改动、依赖版本不正确、环境问题，调用接口问题、代码运行错误等因素后，发现 canvas 对于非 img 元素，均可正确截图，只是嵌入 canvas 中的 img 标签，会导致空白（正好项目里又全是用 img 拼凑的截图），所以问题很自然落到了 img 的头上。

# img中的crossorigin
在 MDN 上遍寻 img 标签的属性，终于找到了 [crossorigin](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/img#crossorigin) 这个属性。

从 MDN 对其解释来看，这个枚举属性表明是否必须使用 CORS 完成相关图像的抓取。设置了表示启用 CORS，不设置表示不开启 CORS。其中该属性有两个有效取值：
+ anonymous: 开启 CORS，发送**忽略凭据**的跨源请求（即，不携带 cookie、X.509 证书 或 Authorization 请求标头）。
+ use-credentials: 开启 CORS，发送**携带凭据**的跨源请求（比如 cookie、X.509 证书和 Authorization 请求标头）。

在一般应用中，img 标签使用 src 属性来获取的内容，其 url 是可以跨域的，也是无需考虑同源，我们也能获取图片资源。但这种场景下，也是不会对 crossorigin 这一属性进行设置，也就意味着没有开启 CORS 的方式来获取图片资源。
常规情况下没什么影响，但是在 MDN 上有一个关键点：`启用 CORS 的图像可以在 <canvas> 元素中重复使用，而不会被标记为“污染（tainted）”`。这就牵扯到了 canvas 的污染和安全问题了。

# 被“污染”的 canvas
canvas 可以通过 drawImage 等 api 来使用其他来源的图片，
MDN 原话：`尽管不通过 CORS 就可以在 canvas 中使用其他来源的图片，但是这会污染画布，并且不再认为是安全的画布，这将可能在 canvas 检索数据过程中引发异常`。
很明显的表示，canvas 可以使用 src 获取的 img 图片，但是如果是没有开启 CORS 方式获取的img，则会被认为是“污染”了canvas，那么这个canvas 的后续一些导出的操作就会受到影响，在“被污染”的 canvas 中调用以下方法将会抛出安全错误：
+ 在 canvas 的上下文上调用 getImageData()
+ 在 canvas 元素本身调用 toBlob()、toDataURL() 或 captureStream()

也就是说，如果 img 不设置 crossorigin 属性，就会在 canvas 使用后造成画布污染，只有设置了 img 的 crossorigin 属性，以 CORS 的方式获取资源，才不会污染画布。至于设置为 anonymous 还是 use-credentials，需要根据服务端的实际情况，如果服务端没有支持资源的 CORS，则设置为 use-credentials 的方式后，会有 CORS 的错误。一般而言，如果仅仅处理简单的 canvas 污染问题，anonymous 足够了。

# 结尾
如上所述，正因为项目中的 img 没有设置 crossorigin 属性，致使 html2canvas 在从 canvas 转图片的过程中产生了错误，导致了后续的截图空白。在为 img 设置 crossorigin 为 anonymous 之后，问题迎刃而解。
