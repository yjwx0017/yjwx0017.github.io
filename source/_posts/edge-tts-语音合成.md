title: edge-tts 语音合成
categories: 日志
tags: [Python,Qt,语音合成,edge-tts]
date: 2024-03-01 15:38:00
---
## edge-tts 简介

edge-tts 是一个 Python 模块，允许使用微软 Edge 的在线文本转语音服务，可使用 Python 代码或 edge-tts 命令行工具执行语音合成。

## 安装

```
pip install edge-tts
```

## 使用

查看 edge-tts 支持的语音列表：

```
edge-tts --list-voices
```

语音合成：

```
edge-tts --voice zh-CN-YunxiNeural --text "你好，我是微软语音合成小助手。" --write-media hello.mp3
```

## 为命令行工具编写一个 UI 界面

使用 Qt 实现。

源码：[https://gitee.com/yjwx0017/edge-tts-ui][1]

![截图][2]


  [1]: https://gitee.com/yjwx0017/edge-tts-ui
  [2]: /usr/uploads/2024/03/1111535442.png