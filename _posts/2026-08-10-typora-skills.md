---
layout: post
title:  typora 技巧
date:   2026-08-10 19:15:03
categories: 计算机
tags: typora
---

* content
{:toc}

### 文章添加水印

``` html
<style>
  /* Typora 导出 PDF 时只渲染 #write 容器，
     给 #write 设置 SVG 平铺背景，Chromium 渲染每一页时都会重复绘制，
     因此水印会出现在 PDF 的每一页 */
  #write {
    background-image: url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='600' height='500'><text x='50%' y='50%' fill='gray' fill-opacity='0.12' font-size='60' font-weight='bold' text-anchor='middle' dominant-baseline='middle' transform='rotate(-45 300 250)'>机密文件，禁止外泄</text></svg>");
    background-repeat: repeat;
    background-size: 600px 500px;
    background-position: center center;
  }
  @media print {
    html,
    body,
    #write {
      -webkit-print-color-adjust: exact;
      print-color-adjust: exact;
    }
  }
</style>
```