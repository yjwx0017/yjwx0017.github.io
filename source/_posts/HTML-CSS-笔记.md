title: HTML CSS 笔记
categories: 日志
tags: [CSS,HTML]
date: 2023-10-27 13:08:00
---
[尚硅谷前端html+css零基础教程，2023最新前端开发html5+css3视频][1]

## HTML4

### 网页基本结构

``` html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title></title>
  </head>
  <body>
  </body>
</html>
```

### 安装 Live Server 插件

VSCode 安装 Live Server 插件。代码出现改动，页面可以自动刷新。

### 注释

``` html
<!-- 注释 -->
```

### 文档声明

作用是告诉浏览器当前网页的版本。

``` html
<!DOCTYPE html>
```

<!--more-->

### 字符编码

``` html
<head>
  <meta charset="UTF-8"/>
</head>
```

### 设置语言

主要作用：

1. 让浏览器显示对应的翻译提示。
2. 有利于搜索引擎优化

``` html
<html lang="zh-CN">
```

第一种写法（ 语言-国家/地区 ）：

- zh-CN ：中文-中国大陆（简体中文）
- zh-TW ：中文-中国台湾（繁体中文）
- zh ：中文
- en-US ：英语-美国
- en-GB ：英语-英国

第二种写法（ 语言—具体种类）已不推荐使用：

- zh-Hans ：中文—简体
- zh-Hant ：中文—繁体

### 网站图标

在网站根目录存放一个 favicon.ico 图片，可配置网站图标。

### 开发者文档

- W3C官网： www.w3c.org
- W3School： www.w3school.com.cn
- MDN： developer.mozilla.org —— 平时用的最多。

### 排版标签

- h1 ~ h6 标题
- p 段落
- div 没有任何含义，用于整体布局

### 语义化标签

用特定的标签，去表达特定的含义。

原则：标签的默认效果不重要（后期可以通过 CSS 随便控制效果），语义最重要！

### 块级元素与行内元素

块级元素：独占一行（排版标签都是块级元素）。

行内元素：不独占一行。

块级元素中能写行内元素和块级元素（简单记：块级元素中几乎什么都能写）。

行内元素中能写行内元素，但不能写块级元素。

一些特殊的规则：

- h1~h6 不能互相嵌套。
- p 中不要写块级元素。

### 文本标签

常用：

- em 要着重阅读的内容
- strong 十分重要的内容（语气比em要强）
- span 没有语义，用于包裹短语的通用容器

不常用：

- cite 作品标题（书籍、歌曲、电影、电视节目、绘画、雕塑）
- dfn 特殊术语 ，或专属名词
- del 与 ins 删除的文本 【与】 插入的文本
- sub 与 sup 下标文字 【与】 上标文字
- code 一段代码
- samp 从正常的上下文中，将某些内容提取出来，例如：标识设备输出
- kbd 键盘文本，表示文本是通过键盘输入的，经常用在与计算机相关的手册中
- abbr 缩写，最好配合上 title 属性
- bdo 更改文本方向，要配合 dir 属性，可选值: ltr （默认值）、rtl
- var 标记变量，可以与 code 标签一起使用
- small 附属细则，例如：包括版权、法律文本。—— 很少使用
- b 摘要中的关键字、评论中的产品名称。—— 很少使用
- i 本意是：人物的思想活动、所说的话等等。现在多用于：呈现字体图标（后面要讲的内容）。
- u 与正常内容有反差文本，例如：错的单词、不合适的描述等。——很少使用
- q 短引用 —— 很少使用
- blockquote 长引用，块级元素 —— 很少使用
- address 地址信息，块级元素

HTML标签太多了！记住那些：重要的、语义感强的标签即可；截止目前，有这些：

h1~h6 、 p 、 div 、 em 、 strong 、 span

### 图片标签

``` html
<img src="" alt="" />
```

常用属性：

- src ：图片路径（又称：图片地址）—— 图片的具体位置
- alt ：图片描述
- width ：图片宽度，单位是像素，例如： 200px 或 200
- height ：图片高度， 单位是像素，例如： 200px 或 200

### 超链接

``` html
<!-- 跳转其他网页 -->
<a href="https://www.jd.com/" target="_blank">去京东</a>
<!-- 跳转本地网页 -->
<a href="./10_HTML排版标签.html" target="_self">去看排版标签</a>
<!-- 浏览器能直接打开的文件 -->
<a href="./resource/自拍.jpg">看自拍</a>
<a href="./resource/小电影.mp4">看小电影</a>
<a href="./resource/小姐姐.gif">看小姐姐</a>
<a href="./resource/如何一夜暴富.pdf">点我一夜暴富</a>
<!-- 浏览器不能打开的文件，会自动触发下载 -->
<a href="./resource/内部资源.zip">内部资源</a>
<!-- 强制触发下载 -->
<a href="./resource/小电影.mp4" download="电影片段.mp4">下载电影</a>
```

若想强制触发下载，请使用 download 属性，属性值即为下载文件的名称。

常用属性：

- href ： 指定要跳转到的具体目标。
- target ： 控制跳转时如何打开页面，常用值如下: _self ：在本窗口打开。 _blank ：在新窗口打开。
- id ： 元素的唯一 标识，可用于设置锚点。
- name ： 元素的名字，写在 a 标签中，也能设置锚点。

### 锚点

设置锚点：

``` html
<!-- 第一种方式：a标签配合name属性 -->
<a name="test1"></a>
<!-- 第二种方式：其他标签配合id属性 -->
<h2 id="test2">我是一个位置</h2>
```

跳转锚点：

``` html
<!-- 跳转到test1锚点-->
<a href="#test1">去test1锚点</a>
<!-- 跳到本页面顶部 -->
<a href="#">回到顶部</a>
<!-- 跳转到其他页面锚点 -->
<a href="demo.html#test1">去demo.html页面的test1锚点</a>
<!-- 刷新本页面 -->
<a href="">刷新本页面</a>
<!-- 执行一段js,如果还不知道执行什么，可以留空，javascript:; -->
<a href="javascript:alert(1);">点我弹窗</a>
```

### 唤起指定应用

``` html
<!-- 唤起设备拨号 -->
<a href="tel:10010">电话联系</a>
<!-- 唤起设备发送邮件 -->
<a href="mailto:10010@qq.com">邮件联系</a>
<!-- 唤起设备发送短信 -->
<a href="sms:10086">短信联系</a>
```

### 列表

有序列表：

``` html
<h2>要把大象放冰箱总共分几步</h2>
<ol>
  <li>把冰箱门打开</li>
  <li>把大象放进去</li>
  <li>把冰箱门关上</li>
</ol>
```

无序列表：

``` html
<h2>我想去的几个城市</h2>
<ul>
  <li>成都</li>
  <li>上海</li>
  <li>西安</li>
  <li>武汉</li>
</ul>
```

自定义列表：

一个 dl 就是一个自定义列表，一个 dt 就是一个术语名称，一个 dd 就是术语描述（可以有多个）。

``` html
<h2>如何高效的学习？</h2>
<dl>
  <dt>做好笔记</dt>
  <dd>笔记是我们以后复习的一个抓手</dd>
  <dd>笔记可以是电子版，也可以是纸质版</dd>
  <dt>多加练习</dt>
  <dd>只有敲出来的代码，才是自己的</dd>
  <dt>别怕出错</dt>
  <dd>错很正常，改正后并记住，就是经验</dd>
</dl>
```

### 表格

- table ：表格
- caption ：表格标题
- thead ：表格头部
- tbody ：表格主体
- tfoot ：表格注脚
- tr ：每一行
- th 、 td ：每一个单元格（备注：表格头部中用 th ，表格主体、表格脚注中用： td ）

``` html
<table border="1">
  <!-- 表格标题 -->
  <caption>学生信息</caption>
  <!-- 表格头部 -->
  <thead>
    <tr>
      <th>姓名</th>
      <th>性别</th>
      <th>年龄</th>
      <th>民族</th>
      <th>政治面貌</th>
    </tr>
  </thead>
  <!-- 表格主体 -->
  <tbody>
    <tr>
      <td>张三</td>
      <td>男</td>
      <td>18</td>
      <td>汉族</td>
      <td>团员</td>
    </tr>
    <tr>
      <td>李四</td>
      <td>女</td>
      <td>20</td>
      <td>满族</td>
      <td>群众</td>
    </tr>
    <tr>
      <td>王五</td>
      <td>男</td>
      <td>20</td>
      <td>回族</td>
      <td>党员</td>
    </tr>
    <tr>
      <td>赵六</td>
      <td>女</td>
      <td>21</td>
      <td>壮族</td>
      <td>团员</td>
    </tr>
  </tbody>
  <!-- 表格脚注 -->
  <tfoot>
    <tr>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td>共计：4人</td>
    </tr>
  </tfoot>
</table>
```

table 常用属性：

- width ：设置表格宽度。
- height ：设置表格最小高度，表格最终高度可能比设置的值大。
- border ：设置表格边框宽度。
- cellspacing ： 设置单元格之间的间距。

thead tbody tr tfoot 常用属性：

- height ：设置表格头部高度。
- align ： 设置单元格的水平对齐方式，可选值如下： left ：左对齐 center ：中间对齐 right ：右对齐
- valign ：设置单元格的垂直对齐方式，可选值如下： top ：顶部对齐 middle ：中间对齐 bottom ：底部对齐

td th 常用属性：

- width ：设置单元格的宽度，同列所有单元格全都受影响。
- heigth ：设置单元格的高度，同行所有单元格全都受影响。
- align ：设置单元格的水平对齐方式。
- valign ：设置单元格的垂直对齐方式。
- rowspan ：指定要跨的行数。
- colspan ：指定要跨的列数。

跨行跨列：

- rowspan ：指定要跨的行数
- colspan ：指定要跨的列数

### 常用标签

- br 换行
- hr 分隔
- pre 按原文显示（一般用于在页面中嵌入大段代码）

不要用 <br> 来增加文本之间的行间隔，应使用 <p> 元素，或后面即将学到的 CSS margin 属性。

<hr> 的语义是分隔，如果不想要语义，只是想画一条水平线，那么应当使用 CSS 完成。

### 表单

form 常用属性：

- action ：用于指定表单的提交地址（需要与后端人员沟通后确定）。
- target ：用于控制表单提交后，如何打开页面，常用值如下：_self ：在本窗口打开 _blank ：在新窗口打开
- method ：用于控制表单的提交方式，暂时只需了解，在后面 Ajax 的课程中，会详细讲解。

input 常用属性：

- type ：设置输入框的类型，目前用到的值是 text ，表示普通文本。
- name ：用于指定提交数据的名字，（需要与后端人员沟通后确定）。

``` html
<form action="https://www.baidu.com/s" target="_blank" method="get">
  <input type="text" name="wd">
  <button>去百度搜索</button>
</form>
```

文本输入框：

``` html
<input type="text">
```

- name 属性：数据的名称。
- value 属性：输入框的默认输入值。
- maxlength 属性：输入框最大可输入长度。

密码输入框：

``` html
<input type="password">
```

- name 属性：数据的名称。
- value 属性：输入框的默认输入值（一般不用，无意义）。
- maxlength 属性：输入框最大可输入长度。

单选框：

``` html
<input type="radio" name="sex" value="female">女
<input type="radio" name="sex" value="male">男
```

- name 属性：数据的名称，注意：想要单选效果，多个 radio 的 name 属性值要保持一致。
- value 属性：提交的数据值。
- checked 属性：让该单选按钮默认选中。

复选框：

``` html
<input type="checkbox" name="hobby" value="smoke">抽烟
<input type="checkbox" name="hobby" value="drink">喝酒
<input type="checkbox" name="hobby" value="perm">烫头
```

- name 属性：数据的名称。
- value 属性：提交的数据值。
- checked 属性：让该复选框默认选中。

隐藏域：

``` html
<input type="hidden" name="tag" value="100">
```

用户不可见的一个输入区域，作用是： 提交表单的时候，携带一些固定的数据。

- name 属性：指定数据的名称。
- value 属性：指定的是真正提交的数据。

提交按钮：

``` html
<input type="submit" value="点我提交表单">
<button>点我提交表单</button>
```

- button 标签 type 属性的默认值是 submit 。
- button 不要指定 name 属性
- input 标签编写的按钮，使用 value 属性指定按钮文字。

重置按钮：

``` html
<input type="reset" value="点我重置">
<button type="reset">点我重置</button>
```

普通按钮：

``` html
<input type="button" value="普通按钮">
<button type="button">普通按钮</button>
```

普通按钮的 type 值为 button ，若不写 type 值是 submit 会引起表单的提交。

文本域：

``` html
<textarea name="msg" rows="22" cols="3">我是文本域</textarea>
```

- rows 属性：指定默认显示的行数，会影响文本域的高度。
- cols 属性：指定默认显示的列数，会影响文本域的宽度。
- 不能编写 type 属性，其他属性，与普通文本输入框一致。

下拉框：

``` html
<select name="from">
  <option value="黑">黑龙江</option>
  <option value="辽">辽宁</option>
  <option value="吉">吉林</option>
  <option value="粤" selected>广东</option>
</select>
```

- name 属性：指定数据的名称。
- option 标签设置 value 属性， 如果没有 value 属性，提交的数据是 option 中间的文字；如果设置了 value 属性，提交的数据就是 value 的值（建议设置 value 属性）
- option 标签设置了 selected 属性，表示默认选中。

给表单控件的标签设置 disabled 既可禁用表单控件。

label 标签可与表单控件相关联，关联之后点击文字，与之对应的表单控件就会获取焦点。

两种与 label 关联方式如下：

1. 让 label 标签的 for 属性的值等于表单控件的 id 。
2. 把表单控件套在 label 标签的里面。

fieldset 与 legend 的使用：

fieldset 可以为表单控件分组、 legend 标签是分组的标题。

### iframe 标签

- name ：框架名字，可以与 target 属性配合。
- width ： 框架的宽。
- height ： 框架的高度。
- frameborder ：是否显示边框，值：0或者1。

iframe 标签的实际应用：

1. 在网页中嵌入广告。
2. 与超链接或表单的 target 配合，展示不同的内容。

### HTML 实体

常见字符实体：

| | 描述 | 实体名称 | 实体编号 |
| --- | --- | --- | --- |
| | 空格 | `&nbsp;` | `&#160;` |
| < | 小于号 | `&lt;` | `&#60;` |
| > | 大于号 | `&gt;` | `&#62;` |
| & | 和号 | `&amp;` | `&#38;` |
| " | 引号 | `&quot;` | `&#34;` |
| ´ | 反引号 | `&acute;` | `&#180;` |
| ￠ | 分（cent） | `&cent;` | `&#162;` |
| £ | 镑（pound） | `&pound;` | `&#163;` |
| ¥ | 元（yen） | `&yen;` | `&#165;` |
| € | 欧元（euro） | `&euro;` | `&#8364;` |
| © | 版权（copyright） | `&copy;` | `&#169;` |
| ® | 注册商标 | `&reg;` | `&#174;` |
| ™ | 商标 | `&trade;` | `&#8482;` |
| × | 乘号 | `&times` | `&#215;` |
| ÷ | 除号 | `&divide;` | `&#247;` |

### 常用的全局属性

- id 给标签指定唯一标识，注意： id 是不能重复的。作用：可以让 label 标签与表单控件相关联；也可以与 CSS 、 JavaScript 配合使用，。
- class 给标签指定类名，随后通过 CSS 就可以给标签设置样式。
- style 给标签设置 CSS 样式。
- dir 内容的方向，值: ltr 、 rtl
- title 给标签设置一个文字提示，一般超链接和图片用得比较多。
- lang 给标签指定语言

### meta 元信息

``` html
<!-- 配置字符编码 -->
<meta charset="utf-8">
<!-- 针对 IE 浏览器的兼容性配置 -->
<meta http-equiv="X-UA-Compatible" content="IE=edge">
<!-- 针对移动端的配置（移动端课程中会详细讲解） -->
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<!-- 配置网页关键字 -->
<meta name="keywords" content="8-12个以英文逗号隔开的单词/词语">
<!-- 配置网页描述信息 -->
<meta name="description" content="80字以内的一段话，与网站内容相关">
<!-- 针对搜索引擎爬虫配置 -->
<meta name="robots" content="此处可选值见下表">
<!--
index 允许搜索爬虫索引此页面。
noindex 要求搜索爬虫不索引此页面。
follow 允许搜索爬虫跟随此页面上的链接。
nofollow 要求搜索爬虫不跟随此页面上的链接。
all 与 index, follow 等价
none 与 noindex, nofollow 等价
noarchive 要求搜索引擎不缓存页面内容。
nocache noarchive 的替代名称。 
-->
<!-- 配置网页作者 -->
<meta name="author" content="tony">
<!-- 配置网页生成工具 -->
<meta name="generator" content="Visual Studio Code">
<!-- 配置定义网页版权信息 -->
<meta name="copyright" content="2023-2027©版权所有">
<!-- 配置网页自动刷新 -->
<meta http-equiv="refresh" content="10;url=http://www.baidu.com">
```

## HTML5

### 新增布局标签

- header 整个页面，或部分区域的头部
- footer 整个页面，或部分区域的底部
- nav 导航
- article 文章、帖子、杂志、新闻、博客、评论等。
- section 页面中的某段文字，或文章中的某段文字（里面文字通常里面会包含标题）。
- aside 侧边栏
- main 文档的主要内容 ( WHATWG 没有语义， IE 不支持)，几乎不用。
- hgroup 包裹连续的标题，如文章主标题、副标题的组合 （ W3C 将其删除）

关于 article 和 section ：

1. artical 里面可以有多个 section 。
2. section 强调的是分段或分块，如果你想将一块内容分成几段的时候，可使用 section 元素。
3. article 比 section 更强调独立性，一块内容如果比较独立、比较完整，应该使用article 元素。

### meter 标签

常用属性：

- high 数值 规定高值
- low 数值 规定低值
- max 数值 规定最大值
- min 数值 规定最小值
- optimum 数值 规定最优值
- value 数值 规定当前值

### progress 标签

常用属性：

- max 数值 规定目标值。
- value 数值 规定当前值。

### 新增列表标签

- datalist 用于搜索框的关键字提示
- details 用于展示问题和答案，或对专有名词进行解释
- summary 写在 details 的里面，用于指定问题或专有名词

``` html
<input type="text" list="mydata">
<datalist id="mydata">
  <option value="周冬雨">周冬雨</option>
  <option value="周杰伦">周杰伦</option>
  <option value="温兆伦">温兆伦</option>
  <option value="马冬梅">马冬梅</option>
</datalist>
```

``` html
<details>
  <summary>如何走上人生巅峰？</summary>
  <p>一步一步走呗</p>
</details>
```

### 新增文本标签

- ruby 包裹需要注音的文字
- rt 写注音， rt 标签写在 ruby 的里面
- mark 标记，W3C 建议 mark 用于标记搜索结果中的关键字。

``` html
<ruby>
  <span>魑魅魍魉</span>
  <rt>chī mèi wǎng liǎng </rt>
</ruby>
```

### 表单控件新增属性

- placeholder 提示文字（注意：不是默认值， value 是默认值），适用于文字输入类的表单控件。
- required 表示该输入项必填， 适用于除按钮外其他表单控件。
- autofocus 自动获取焦点，适用于所有表单控件。
- autocomplete 自动完成，可以设置为 on 或 off ，适用于文字输入类的表单控件。注意：密码输入框、多行输入框不可用。
- pattern 填写正则表达式，适用于文本输入类表单控件。注意：多行输入不可用，且空的输入框不会验证，往往与 required 配合。

### input 新增属性值

- email 邮箱类型的输入框，表单提交时会验证格式，输入为空则不验证格式。
- url url 类型的输入框，表单提交时会验证格式，输入为空则不验证格式。
- number 数字类型的输入框，表单提交时会验证格式，输入为空则不验证格式。
- search 搜索类型的输入框，表单提交时不会验证格式。
- tel 电话类型的输入框，表单提交时不会验证格式，在移动端使用时，会唤起数字键盘。
- range 范围选择框，默认值为 50 ，表单提交时不会验证格式。
- color 颜色选择框，默认值为黑色，表单提交时不会验证格式。
- date 日期选择框，默认值为空，表单提交时不会验证格式。
- month 月份选择框，默认值为空，表单提交时不会验证格式。
- week 周选择框，默认值为空，表单提交时不会验证格式。
- time 时间选择框，默认值为空，表单提交时不会验证格式。
- datetime-local 日期+时间选择框，默认值为空，表单提交时不会验证格式。

### form 新增属性

- novalidate 如果给 form 标签设置了该属性，表单提交的时候不再进行验证。

### video 标签

常用属性：

- src URL地址 视频地址
- width 像素值 设置视频播放器的宽度
- height 像素值 设置视频播放器的高度
- controls - 向用户显示视频控件（比如播放/暂停按钮）
- muted - 视频静音
- autoplay - 视频自动播放
- loop - 循环播放
- poster URL地址 视频封面
- preload auto / metadata / none 视频预加载，如果使用 autoplay ，则忽略该属性。
- src URL地址 音频地址
- controls - 向用户显示音频控件（比如播放/暂停按钮）
- autoplay - 音频自动播放
- muted - 音频静音
- loop - 循环播放
- preload auto / metadata / none 音频预加载，如果使用 autoplay ，则忽略该属性。

- none : 不预加载视频。
- metadata : 仅预先获取视频的元数据（例如长度）。
- auto : 可以下载整个视频文件，即使用户不希望使用它。

### audio 标签

常用属性：

- src URL地址 音频地址
- controls - 向用户显示音频控件（比如播放/暂停按钮）
- autoplay - 音频自动播放
- muted - 音频静音
- loop - 循环播放
- preload auto / metadata / none 音频预加载，如果使用 autoplay ，则忽略该属性。

- none : 不预加载视频。
- metadata : 仅预先获取视频的元数据（例如长度）。
- auto : 可以下载整个视频文件，即使用户不希望使用它。

### 新增全局属性

- contenteditable 表示元素是否可被用户编辑，可选值如下：true ：可编辑 false ：不可编辑
- draggable 表示元素可以被拖动，可选值如下：true ：可拖动 false ：不可拖动 
- hidden 隐藏元素
- spellcheck 规定是否对元素进行拼写和语法检查，可选值如下：true ：检查 false ：不检查
- contextmenu 规定元素的上下文菜单，在用户鼠标右键点击元素时显示。
- data-* 用于存储页面的私有定制数据。

### HTML5 兼容性处理

添加元信息，让浏览器处于最优渲染模式：

``` html
<!--设置IE总是使用最新的文档模式进行渲染-->
<meta http-equiv="X-UA-Compatible" content="IE=Edge">
<!--优先使用 webkit ( Chromium ) 内核进行渲染, 针对360等壳浏览器-->
<meta name="renderer" content="webkit">
```

使用 html5shiv 让低版本浏览器认识 H5 的语义化标签：

``` html
<!--[if lt ie 9]>
<script src="../sources/js/html5shiv.js"></script>
<![endif]-->
```

- lt 小于
- lte 小于等于
- gt 大于
- gte 大于等于
- ! 逻辑非

``` html
<!--[if IE 8]>仅IE8可见<![endif]-->
<!--[if gt IE 8]>仅IE8以上可见<![endif]-->
<!--[if lt IE 8]>仅IE8以下可见<![endif]-->
<!--[if gte IE 8]>IE8及以上可见<![endif]-->
<!--[if lte IE 8]>IE8及以下可见<![endif]-->
<!--[if !IE 8]>非IE8的IE可见<![endif]-->
```

  [1]: https://www.bilibili.com/video/BV1p84y1P7Z5
