---
title: 如何修改npm包中的代码
date: 2024-08-13 15:15:30
tags: 
    - npm 
    - git
---
在开发过程中总是避免不了npm包源码不适配我们的需求，通常我们都是提pr或者fork本地改造。但是，有没有能在本地修改源码的方法呢？答案是有的 -- 本地补丁

首先在项目根目录新建patches文件夹之后输入
<!--more-->
```bash
git diff > patches/third-party-package.patch
```
这个时候我们就创建了一个空补丁包，但是为了之后可能存在的重新npm install或者yarn刷掉了所有的本地包，我们需要使用自动化地在每次安装依赖时应用这个补丁。
首先我们要下载两个包 postinstall-postinstall 和 patch-package

```bash
npm install patch-package postinstall-postinstall --save-dev
```
postinstall-postinstall 旨在解决 postinstall 脚本在 monorepo 或者多包（multi-package）项目中的重复执行问题。
patch-package 持久化依赖包的修改

在package.json中加入配置
```json
"scripts": {
  "postinstall": "patch-package"
}
```
现在假设我们修改node_modules里的debug包
![alt text](https://showpenz.github.io/images/QQ_1723534472265.png)
然后执行
```bash 
npx patch-package debug

# 以下是执行结果
# patch-package 8.0.0
# • Creating temporary folder
# • Installing debug@4.3.6 with yarn
# • Diffing your files with clean files
# ✔ Created file patches/debug+4.3.6.patch

# 💡 debug is on GitHub! To draft an issue based on your patch run

#     yarn patch-package debug --create-issue
```
patches目录下也更新了
![alt text](https://showpenz.github.io/images/QQ_1723534567893.png)
好的，现在我们删除node_modules 测试我们的补丁是否生效然后执行
```bash
yarn
```
在打开node_modules发现debug的代码已经注入了补丁![alt text](https://showpenz.github.io/images/QQ_1723534673042.png)
上述即完成了补丁持久化的操作