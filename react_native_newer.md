---
title: reactNative 尝试
date: 2016-04-30 15:31:00
categories: 技术
tags: [react]
description: reactNative 尝试
---



## 安装及初始化工作

REF: reactnative.cn/docs/0.31/getting-started.html

### 项目初始化

- Q: 下载太慢
- A: 完整打包:
github: https://github.com/CellChen/react-native-init
速度慢的话使用
coding: https://coding.net/u/chenhui/p/react-native-init
- REF:
https://segmentfault.com/q/1010000003767098?_ea=363563

### 项目运行

- Q: react-native run-ios

```
node_modules/react-native/local-cli/cli.js:56
  const setupEnvScript = /^win/.test(process.platform)
  ^^^^^
```

- A: nginx 版本过低，用 nvm 管理 nodejs 版本

- REF: 
	- https://segmentfault.com/q/1010000003969608
	- http://www.tuicool.com/articles/Vzquy2

- Q:Command `run-ios` unrecognized. Did you mean to run this inside a react-native project?
- A:
	- npm install -g react-native-cli
	- npm install --save react-native@latest