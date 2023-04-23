---
lang: zh
title: 4 月 18 号后在 Chrome 上使用 New Bing <br> Using New Bing on Chrome after April 18th
layout: post
---

4 月 18 号（好像是）之后，之前通过修改请求头的 `user-agent` 来在 Chrome 上使用 New Bing 的方法就失效了，会强行跳转到 Edge 上。

After April 18th (maybe), the previous method of using New Bing on Chrome by modifying the `user-agent` of the request header has been invalid, and it will forcibly jump to Edge.

通过对比 Edge 与 Chrome 上的请求头，发现其实是现在对请求头的判断多加了一项 `sec-ch-ua-full-version-list`。

By comparing the request headers on Edge and Chrome, it is actually that an additional `sec-ch-ua-full-version-list` is added to the judgment of the request header.

![request-head](/assets/reqhead.PNG)

因此只需在[之前修改请求头的方法](#之前的方法-previous-method)之上再加上 `sec-ch-ua-full-version-list` 修改即可。

修改值为 `"Chromium";v="112.0.5615.121", "Microsoft Edge";v="112.0.1722.48", "Not:A-Brand";v="99.0.0.0"`

Therefore, you only need to add `sec-ch-ua-full-version-list` modification on top of the [previous method of modifying the request header](#之前的方法-previous-method).

Modify the value to `"Chromium";v="112.0.5615.121", "Microsoft Edge";v="112.0.1722.48", "Not:A-Brand";v="99.0.0.0"`

![head-edit](/assets/head-edit.PNG)

为保稳健可把 `sec-ch-ua` 也加上，虽然目前不会根据该项进行判断，以防万一之后会把这项也加上。

修改值为 `"Chromium";v="112", "Microsoft Edge";v="112", "Not:A-Brand";v="99"`

You can also add `sec-ch-ua` for stability, although it will not be judged based on this item at present, just in case it will be added in the future.

Modify the value to `"Chromium";v="112", "Microsoft Edge";v="112", "Not:A-Brand";v="99"`

![head-edit](/assets/head-edit2.PNG)

## 之前的方法 Previous Method

下载并安装 [Header Editor](https://chrome.google.com/webstore/detail/header-editor/eningockdidmgiojffjmkdblpjocbhgh) 插件至 Chrome。安装好后点击插件图标，进入管理界面。

Download and install the [Header Editor](https://chrome.google.com/webstore/detail/header-editor/eningockdidmgiojffjmkdblpjocbhgh) plugin to Chrome. After installation, click the plugin icon and enter the management page.

![head-edit0](/assets/head-edit0.PNG)

点击右下角 + 图标，选择 修改请求头，正则表达式

匹配类型 `^http(s?)://(.*).bing\.com/(.*)`

头名称 `user-agent`

头内容 `Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/112.0.0.0 Safari/537.36 Edg/112.0.1722.48`

保存。

Click the + icon in the lower right corner, select Modify Request Header, Regular Expression

Match Type  `^http(s?)://(.*).bing\.com/(.*)`

Header Name `user-agent`

Header Content `Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/112.0.0.0 Safari/537.36 Edg/112.0.1722.48`

And save.

![head-edit1](/assets/head-edit1.PNG)

