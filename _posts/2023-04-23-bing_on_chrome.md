---
lang: zh
title: 4 月 18 号后在 Chrome 上使用 New Bing \n Using New Bing in Chrome after April 18th
layout: post
---

4 月 18 号（好像是）之后，之前通过修改请求头的 `user-agent` 来在 Chrome 上使用 New Bing 的方法就失效了，会强行跳转到 Edge 上。

通过对比 Edge 与 Chrome 上的请求头，发现其实是现在对请求头的判断多加了一项 `sec-ch-ua-full-version-list`。

![request-head](/assets/reqhead.PNG)

因此只需在之前修改请求头的方法之上再加上 `sec-ch-ua-full-version-list` 修改即可。

修改值为 `"Chromium";v="112.0.5615.121", "Microsoft Edge";v="112.0.1722.48", "Not:A-Brand";v="99.0.0.0"`

![head-edit](/assets/head-edit.PNG)

为保稳健可把 `sec-ch-ua` 也加上，虽然目前不会根据该项进行判断，以防万一之后会把这项也加上。

修改值为 `"Chromium";v="112", "Microsoft Edge";v="112", "Not:A-Brand";v="99"`

![head-edit](/assets/head-edit2.PNG)
