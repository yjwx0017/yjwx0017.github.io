title: Dear ImGui
categories: 日志
tags: [ImGui,Qt]
date: 2021-01-02 12:23:00
---
## Dear ImGui

[https://github.com/ocornut/imgui][1]

Dear ImGui 是一个即时渲染GUI组件。

即时渲染：游戏代码里有个游戏循环，每次循环就是一帧，一般玩游戏谈的每秒多少帧就是每秒执行了多少次游戏循环。在循环中可以处理用户输入、模型渲染、界面渲染、物理模拟等等。

而 Dear ImGui 就是一个UI渲染库。

GitHub上各路神仙上传的截图真的好炫酷。

贴一张官方的Demo截图：

![ImGui Demo截图][2]

## QtImGui

找到了一个能将ImGui集成到QtWidget上的项目。

[https://github.com/seanchas116/qtimgui][3]

![QtImGui 截图][4]


  [1]: https://github.com/ocornut/imgui
  [2]: /usr/uploads/2021/01/3650587755.png
  [3]: https://github.com/seanchas116/qtimgui
  [4]: /usr/uploads/2021/01/3845625907.jpg