# Linux终端git无法联网


## 问题描述

电脑使用了`clash`代理工具，配好环境之后浏览器可以访问`谷歌`和`github`，但是终端无法推送代码到github

## 解决方法
打开终端执行如下命令：
```
git config --global http.proxy 'sockets5://127.0.0.1:7891'
git config --global https.proxy 'sockets5://127.0.0.1:7891'

# 或者
git config --global http.proxy 'http://127.0.0.1:7890'
git config --global https.proxy 'https://127.0.0.1:7890'
```

具体看使用啥代理方式


