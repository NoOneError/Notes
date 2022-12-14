# [Typora快捷键](https://www.cnblogs.com/hongdada/p/9776547.html)

## [windows快捷键:](https://www.cnblogs.com/hongdada/p/9776547.html#windows%E5%BF%AB%E6%8D%B7%E9%94%AE%EF%BC%9A)

- 无序列表：输入-之后输入空格 

  - 
  - 123

- ==有序列表：输入数字+“.”之后输入空格==

  1. 123
  2. 123
  3. 123

- 任务列表：-[空格]空格 文字

- ==标题：ctrl+数字==

- ==表格：ctrl+t==

- ==生成目录：`[TOC]`按回车==

- 选中一整行：ctrl+l

- 选中单词：ctrl+d

- 选中相同格式的文字：ctrl+e

- 跳转到文章开头：ctrl+home

- 跳转到文章结尾：ctrl+end

- 搜索：ctrl+f

- 替换：ctrl+h

- 引用：输入>之后输入空格

- ==代码块：ctrl+alt+f==

- **加粗**：ctrl+b

- *倾斜*：ctrl+i

- <u>下划线</u>：ctrl+u

- ~~删除线~~：alt+shift+5

- ==插入图片：直接拖动到指定位置即可或者ctrl+shift+i==

- ==插入链接：ctrl + k==

- <audio>
      <source src="C:\Users\57010\Music\兔裹煎蛋卷 - 忆似故人曲.mp3" type="audio/mp3">
  </audio>

- 字体颜色 
  
  - 在句首
    - <font color='orange'>反例</font> 
    - <font color='red'>强制</font>
    - <font color='green'>正例</font>
    - <font color='yellow'>推荐</font>
    - <font color='brown'>说明</font>
  
  - 在句中
    - <font color='cornflowerblue'>正文说明---------一般重要</font>
    - **加粗--------重要**
    - ==高亮---------非常重要==
    - `小标题`
    - *~~倾斜~~*--------不建议使用
  
- 转pdf标题自动分页：

  1. 在需要分页的前面，添加:

     ```css
     <div style="page-break-after: always;"></div>
     ```

  2. 在css文件中添加:

  ```css
  @media print {
  	h2 {
  		page-break-before: always;
  	}
  	h2:first-of-type {
  		page-break-before: avoid;
  	}
  }
  ```

## 给代码块设置快捷键:[#](https://www.cnblogs.com/hongdada/p/9776547.html#给代码块设置快捷键)

偏好设置->打开高级设置->conf.user.json文件

```json
  "keyBinding": {
    // for example: 
    // "Always on Top": "Ctrl+Shift+P"
	"Always on Top": "Ctrl+Shift+P",  
    "Code Fences": "Ctrl+Shift+F",  
    "Ordered List":"Ctrl+Alt+o",  
    "Unordered List": "Ctrl+Alt+u"  
  },
```

Code Fences 代码块

Ordered List 数字有序列表

Unordered List 无序列表

## 下标[#](https://www.cnblogs.com/hongdada/p/9776547.html#下标)

想要使用这个功能，需要在设置面板的 `Markdown` 栏启动它，之后使用`~`来修饰下标文本。栗如：

`H~2~O` 和`X~long\ text~` 显示为 H2O 和Xlong text 。

\#### 13.上标

想要使用这个功能，需要在设置面板的 `Markdown` 栏启动它，之后使用`^`来修饰下标文本。栗如：

`X^2^` 显示为 X2 。

## 高亮[#](https://www.cnblogs.com/hongdada/p/9776547.html#高亮)

想要使用这个功能，需要在设置面板的`Markdown` 栏启动它，之后使用`==`来修饰高亮文本，栗如：

`==highlight==` 显示为 highlight 。

## 图片：[#](https://www.cnblogs.com/hongdada/p/9776547.html#图片：)

[![img](https://img2018.cnblogs.com/blog/443934/201810/443934-20181012170159282-378811511.png)](https://img2018.cnblogs.com/blog/443934/201810/443934-20181012170159282-378811511.png)
[![img](https://img2018.cnblogs.com/blog/443934/201810/443934-20181012170211920-1988294604.png)](https://img2018.cnblogs.com/blog/443934/201810/443934-20181012170211920-1988294604.png)

## 表情[#](https://www.cnblogs.com/hongdada/p/9776547.html#表情)

输出表情需要借助 `：`符号。

栗子：`:smile` 显示为 😄,记住是左右两边都要冒号。

使用者可以通过使用`ESC`键触发表情建议补全功能，也可在功能面板启用后自动触发此功能。同时，直接从菜单栏`Edit` -> `Emoji & Symbols`插入UTF8表情符号也是可以的。

或者使用下面的方法

访问网站 https://emojikeyboard.org/，找到需要的符号，鼠标左键单击，然后粘贴到需要的地方就行了！🆗

:sweat_smile:

## 数学公式[#](https://www.cnblogs.com/hongdada/p/9776547.html#数学公式)

你可以通过使用**MathJax**来实现*LaTeX*的数学符号的表达。

输入`$$`，然后按下`Enter`键就会弹出一个支持TeX/LaTeX语法的输入框，下面是一个栗子：

V1×V2=∣∣ijk ∂X∂u∂Y∂u0 ∂X∂v∂Y∂v0 ∣∣V1×V2=|ijk ∂X∂u∂Y∂u0 ∂X∂v∂Y∂v0 |

在Markdown源文件中，数学的公式块是通过利用`$$`标记借用*LaTeX*语言来实现的：

```text
$$
\mathbf{V}_1 \times \mathbf{V}_2 =  \begin{vmatrix} 
\mathbf{i} & \mathbf{j} & \mathbf{k} \\
\frac{\partial X}{\partial u} &  \frac{\partial Y}{\partial u} & 0 \\
\frac{\partial X}{\partial v} &  \frac{\partial Y}{\partial v} & 0 \\
\end{vmatrix}
$$
```

## HTML[#](https://www.cnblogs.com/hongdada/p/9776547.html#html)

Typora不能使用HTML元素，但是Typora可以解析和编译非常有限的HTML元素，作为Markdown功能的补充，这些有限的功能包括：

- 下划线： `<u>underline</u>`
- 图片：`<img src="http://www.w3.org/html/logo/img/mark-word-icon.png" width="200px" />`（HTML标签中的`width`, `height` 以及属于样式的`width`, `height`, `zoom`样式可以被识别和应用。）
- 评论：`<!-- This is some comments -->`
- 超链接： `<a href="http://typora.io" target="_blank">link</a>` 。

大多数这些属性、样式或分类会被忽略。对其他的标签，Typora会将它们以HTML片段的形式表达。

## 行内嵌数学符号[#](https://www.cnblogs.com/hongdada/p/9776547.html#行内嵌数学符号)

想要使用这个功能，需要在设置面板的 `Markdown`栏启用它。然后使用`$`来启动TeX命令，栗如：`$\lim_{x \to \infty} \exp(-x) = 0$` 会以LaTeX的命令形式表达出来。

为了触发行内内嵌数学符号的实时编译你需要：输入`$`然后按下`ESC`键之后输入TeX命令，之后就会弹出一个如图所示的工具提示栏：

[![img](https://pic3.zhimg.com/v2-4033508b043cad96c59ec4edbca92f36_b.gif)](https://pic3.zhimg.com/v2-4033508b043cad96c59ec4edbca92f36_b.gif)



##  自定义css样式



 [案例样式](typora样式\gitee.css)

