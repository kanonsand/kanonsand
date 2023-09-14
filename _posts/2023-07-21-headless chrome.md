---
layout: post
title: "headless chrome"
tags: ["bash","linux","chrome","screenshot"]
---

## 需求
使用脚本自动化流程时碰到一个需求，需要对某个网页截图然后上传到制定网页，需要解决在命令行中
进行网页截图的问题。

## 解决方案
### 命令行抓取截图
1、chrome有一个headless模式，可以在命令行中不打开chrome gui的情况下进行截图等操作，但是
这个模式无法使用cookie，最后找到了下面这个项目：
[github](https://github.com/NeverMendel/chrome-headless-screenshots)

> 该项目使用nodejs，使用前确保安装了nodejs14以上的版本。

下载项目后进入项目目录，执行 node index.js可以看到提示，按照提示配置参数即可。

#### 常见问题
1、chrome路径未找到
    如果chrome没有安装在默认路径下，程序无法自动识别到chrome路径，项目路径下创建.puppeteerrc.yml文件，
内容如下
```shell
executablePath:
  /opt/google/chrome/chrome
```
指定chrome具体路径
2、cookie格式不对
    cookie要使用json的格式，如下（项目下也有个cookie的示例文件）
```shell
[
  {
    "name": "CONSENT",
    "value": "YES+shp.gws-20210811-0-RC2.en+FX+896",
    "domain": ".google.com"
  }
]
```
3、需要使用代理
    没有提供命令行选项，可以修改index.js，找到如下内容
```shell
args: [
        '--no-sandbox',
        '--headless',
        '--disable-gpu',
        '--disable-dev-shm-usage',
        '--remote-debugging-port=9222',
        '--remote-debugging-address=0.0.0.0',
        '--proxy-server=http://10.200.19.51:646',
      ],
```
上面的--proxy-server就是自己添加的代理参数

### 命令行查看图片
抓取完截图后可能想要命令行查看下大概，只通过terminal查看的工具如下
[tiv](https://github.com/stefanhaustein/TerminalImageViewer.git)
clone后进入src/main/cpp然后make，make install即可，之后执行
```shell
tiv test.jpg
```
即可查看图片

#### 常见问题
1、格式无法识别
tiv需要一些解码库，安装如下软件
```shell
 sudo apt install imagemagick
```
会自动安装这些解码库