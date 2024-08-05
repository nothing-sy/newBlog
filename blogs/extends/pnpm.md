---
title: pnpm的版本管理（超时问题）
date: 2024-08-05
categories:
 - 扩展
---

问题： 当使用pnpm进行node版本管理的时候，突然发现无法切换版本了，总是提示 `Get "https://nodejs.org/dist/index.json": dial tcp 104.20.23.46:443: i/o timeout`


尝试解决问题：
1、开了代理后访问 `https://nodejs.org/dist/index.json` 是没有问题的。但是仍然无法访问。
2、尝试使用nvm进行版本管理，也是同样的提示。问题还是出在代理上

最终解决：

如果是nvm ，请设置代理 nvm proxy 127.0.0.1:7890  // 对应自己的代理端口即可正常管理
如果是npm,请设置代理  pnpm config set proxy http://127.0.0.1:7890 // http协议是必不可少的，如果不带协议仍然无法访问。

取消代理：
nvm proxy none
pnpm config set proxy