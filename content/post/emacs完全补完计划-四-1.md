---

title: emacs完全补完计划(四)--API大考古
date: 2016-02-19 23:36:37
tags: ["emacs", "API"]
---
<!-- toc -->

> 查找和学会使用API最好的方法是阅读官方文档，这篇文章着重讲一些emacs常用的概念和API惯用方法，之前提到的buffer、mode等都是从构架层次上来讲的，这次要具体的分析，所有编辑器的刚需都差不多，所以概念的东西都一样，重要的是如何使用。

<!--more-->

## 如何查看内部文档？
emacs内部文档完整且条理清晰，这也是我投入到emacs的重要原因之一。使用**M-x info**或者默认绑定键**C-h i**，可以按分类阅读内置文档。或者按关键字索引文档，这么用**M-x info-apropos**，非常有用的命令，建议绑定一个常用快键键。进入文档后，以下是阅读文档的正确姿势：

| 按键      |   功能                                       |
|-------|------------------------------------------|
|  SPC | 下翻一页，可越过章节                          |
|  BACKSAPCE | 向上翻一页，可越过章节                |
|  b  | 跳到本章节开始处 
|  n | 下个同级章节                         |
|  p | 上个同级章节                       |
|  ] | 下个章节 |
|  [ | 上个章节|
|  u | 父章节|
|  q | 退出 |
|  ? |帮助列表|


## 怎么测试运行elisp代码？
![图片](../images/ielm1.png)
上图包含了elisp运行的方法，结合两个buffer，一个名为scratch，可以临时书写代码并运行查看结果；另一个为IELM，可按**M-x ielm**打开，和其他脚本语言一样用来REPL(read eval print loop)。
下图演示了两者的关联。
![图片](../images/ielm2.png)

其实这两个命令分别绑定 `eval-print-last-sexp`和`eval-last-sexp`，**eval**就是用来执行elisp代码的，以下是相关的函数：

| 函数                 | 功能                                                              |
|----------------------|-------------------------------------------------------------------|
| eval-last-sexp       | 执行上一个elisp表达式                                             |
| eval-print-last-sexp | 执行并打印上一个elisp表达式                                       |
| eval-defun           | 执行定义的函数（其实就是把定义的函数加入到当前emacs执行环境当中） |
| eval-region          | 执行选中的区域代码                                                  |
| eval-current-buffer  | 执行当前buffer所有代码                                                    |


## Emacs 中的术语
Emacs中有些术语过于古老（好魔幻的感觉），下图是对应现代编辑器的概念：

|  EMACS术语 |现代概念 |说明                                                   |
|-------------------|---------------------|-------------------------------------|
| Point             |Point | 光标位置|
| Buffer            | Buffer| 和其他编辑器的概念一样|
| Mark              | select| 选择一段文字|
| Region            | selection | 被选中的区域，Mark的作用结果|
| Windows           | Windows| 操作窗口 |
| Yank              | Copy                | 复制|
| Kill              | Cut                 | 剪切|
| Kill Ring         | Clipboard           | 剪切板|
| fringe         | gutter/margin          |  左右边栏|
| Font Locking      | Syntax highlight     | 语法高亮控制|
| face | style | 样式、装饰，涉及Font、color等|
| imenu |function list/symbol list | 显示函数和变量的列表|

## Buffer 
> buffer是编辑器最基本的构成，其他部件都可以说是依附其上。又因为编辑器管理的是多个buffer，所以会有个buffer list，也有相应的列表函数，对buffer操作的函数应该最多。

### buffer对象本身
有些函数能够望文生义，这里仅做记录，不做说明，如要使用参考自带文档即可：

| 函数             | 功能 |
|------------------|-------------|
| buffer-list      |  省略    |
| buffer-name      |  省略    |
| get-buffer       |  省略    |
| kill-buffer      |  省略    |
| erase-buffer     |  省略    |
| save-buffer      |  省略    |
| rename-buffer    |  省略    |
| set-buffer       |  省略    |
| buffer-file-name |  省略    |
| previous-buffer  |  省略    |
| next-buffer      |  省略    |

1. current-buffer 和 with-current-buffer
  `(current-buffer)`直接返回当前的对象，**with-current-buffer**则在当前buffer下执行代码，相当于把代码包含到当前buffer上下文内：`(with-current-buffer buffer对象或者buffer名称 代码)`

2. generate-new-buffer：新建buffer对象，若已存在也会新建，则在同名的buffer基础加上递增的数字后缀。 

3. window-buffer:获得窗口对应的buffer，`(window-buffer &optional WINDOW)`，输入的是window对象。

### buffer文本操作
操作buffer里面的文本内容相关的函数：
1. buffer-string: 获得整个buffer的文本内容
2. buffer-substring 和 buffer-substring-no-properties:获得buffer某些区间的文本内容，后面返回纯文本内容，前者则包含了一些text专属的properties。
3. insert、insert-char、insert-string、insert-buffer-substring-no-properties:插入操作，根据后缀可以看出它们的不同用途。所有的插入，是根据当前buffer光标位置而言的。
4. insert-file-contens:插入某个文件的内容。

### buffer中查找和替换
查找和替换难免会涉及到正则表达式，以后会详细说明，现在提有用到的函数：
1. string-match
  搜索文本，`(string-match REGEXP STRING &optional START)`后面接着一个正则表达式和需要搜索的内容，最后参数可选择搜索的起始位置。
2. match-string以及match-beginning/match-end/match-data
  `(match-string N)` 返回上次正则查询的第N个分组的内容。`(match-beginning N)`获得上次正则查询第N个分组的起始字符，match-end同上。match-data以列表的形式返回上次查找的内容。
3. replace-match
  替换操作，需要与其他的search类函数配合，替代search匹配的文本。
4. replace-regexp
  以正则方式查找并替换。
5. looking-at
  `(looking-at REGEXP)` 一般用作判断文本是否存在。

### save-current-buffer保存现场
   `(save-current-buffer 表达式)`保存当前buffer，执行其中的表达式，完成后又回到原来的buffer。这个是常用的伎俩，还有两个类似的函数：
   1. `(save-excursion 表达式)`执行完又回到原来的光标位置，随处可见。
   2. `(save-window-excursion 表达式)`针对window的。

## 编辑操作

> 编辑操作基本上都是基于buffer的，这一小节完全可以归并到buffer里面讲，但涉及到的细节比较多，故而单独拿出来讲。

### Point
光标位置的重要性不言而喻。

#### 获得Point

| 函数                               | 说明                                                                 |
|------------------------------------|----------------------------------------------------------------------|
| (point)                            | 返回当前的光标位置                                                   |
| (point-min)                        | buffer的最开始位置                                                   |
| (point-max)                        | buffer的最末尾位置                                                   |
| (line-beginning-position)          | 一行最开始的位置                                                     |
| (line-end-position)                | 一行最末尾的位置                                                     |
| (region-beginning)                 | 被选中区域的开始位置                                                 |
| (region-end)                       | 被选中区域的末尾位置                                                 |
1. bobp(beginning of buffer predicate)用来判断是否在buffer头部，eobp(end of buffer predicate)用来判断是否在buffer尾部。
2. bolp(beginning of line predicate)用来判断是否在行首，eolp(end of line predicate)用来判断是否在行尾。

#### 光标移动
1. goto-char:跳转到某个光标位置，`(goto-char POSITION)`，输入位置参数。
2. forward-char/backward-char/forward-word/backward-word/forward-line/backward-line:按单个字符向前或者向后移动；按一个单词向前或者向后移动；按行向前或者向后移动。
3. search-forward/search-backward:向前或者向后搜索字符串`(search-forward Str)`光标移动到Str末尾位置。
4. (re-search-forward Regex)/(search-forward-regexp Regex):同上，不过是用的正则查询。同理有：(re-search-backward myRegex) / (search-backward-regexp myRegex)
5. beginning-of-buffer/end-of-buffer/beginning-of-line/end-of-line:移动到buffer的起始位置或者末尾位置；移动到一行开始或者结束位置。

#### thing-at-point
* **thing-at-point**是非常智能的识别当前光标所在位置的字符串类型，并返回这个字符串。比如：`(thing-at-point 'email)`，如果光标位于形如 xxx@xxx.com的任何位置，都将返回整个"xxx@xxx.com"字符串。
* 可识别的字符串类型：`symbol', `list', `sexp', `defun', `filename',`url', `email', `word', `sentence', `whitespace', `line', `number', `page'。
* `bounds-of-thing-at-point`:以点对的形式返回可识别字符串两头的位置。

### Region/Mark
region是选择后的区域，以下都是选择区域后的操作。
1. region-beginning / region-end:区域的开始位置和末尾的位置。
2. comment-region:注释region。
3. delete-region/kill-region:删除region。
4. indent-region:缩进region。
5. region-active-p:判断region是否活动。

### Marker
Marker是标记，中文直接意译为**马克**，按自己定义的名称在某个地方标记一下，用来快速跳转不同位置：
1. make-marker:创建空的标记。
2. set-marker:设置标记的位置和缓冲区`(set-marker temp (point))`。
3. point-marker:得到point处的标记。
4. copy-marker:复制标记
5. maker-position / marker-buffer:得到一个marker的内容。

## window
窗口对象，和操作buffer的函数相似。
1. selected-window:得到当前光标所在的窗口
2. window-list:当前frame的window列表
3. next-window/window-previo:windo-list里面向前一个window和向后一个window
4. delete-window/delete-other-windows:删除当前window/删除当前其他window
5. split-window:分割window
6. current-window-configuration:所谓的window-configuration是当前frame里面的window状态，有几个window，分割成什么样。
7. set-window-configuration:设置当前窗口配置信息。
8. walk-windows:遍历窗口
9. set-window-buffer:让某个窗口显示指定的buffer。
10. select-window:选择某个窗口。
11. window-live-p:判断窗口对象是否存在。
12. window-height/window-width

## file、directory
文件和路径操作和buffer一样特别多，而且两者操作互相关联。

### file和directory

| 函数             | 功能                         |
|------------------|------------------------------|
| find-file        | 查找文件                     |
| rename-file      | 重命名文件                   |
| copy-file        | 拷贝                         |
| delete-file      | 删除                         |
| copy-directory   | 直接拷贝目录                 |
| delete-directory | 删除目录                     |
| make-directory   | 创建目录                     |
| write-file       | 将buffer内容写入文件         |
| file-exists-p    | 判断file在某个路径下是否存在 |
| file-readable-p  | 文件是否可读                 |
| file-attributes  | 返回文件相关的属性           |
| set-file-time    | 设置文件的修改时间           |
| set-file-modes   | 设置文件模式                 |

从上面可以看出**-p**结尾的函数一般用来判断，p是predicate(谓词)的缩写，去掉**-**这规则也成立:

``` lisp
(numberp 45)   ;;==> t
(stringp "hello") ;;==>t
(symbolp 'buffer-name) ;;==t
```

### file-name和directory 

1. file-name-directory/file-name-nodirectory/file-name-extension:

    ``` lisp
    (file-name-directory "~/wtx/tempfile.txt") ;; => "~/wts/"
    (file-name-nondirectory "~/wtx/tempfile.txt") ;; => "tempfile.txt"
    (file-name-extension "~/wtx/tempfile.txt") ;; => "txt"
    ```
2. (expand-file-name filename):得到绝对路径。
3. (file-relative-name FilePath dirPath):把绝对路径转换成相对路径，第二参数为相对的路径。
4. directory-file-name:获得目录名。
5. directory-files / directory-files-attributes:得到某个目录的全部或者符合正则表达式的文件列表directory-files-attributes返回文件列表里面的属性。

## OS API(操作系统接口)

> emacs可以与操作系统交互。

### Commands 和 Apps
1. shell-command/async-shell-command:执行shell命令，等待shell命令结束或者异步。
2. shell-command-to-string:执行命令得带shell命令结束，并获取命令的输出:
  ``` lisp
  (shell-command-to-string "pwd")  ;;==>"/home/wtx\n"
  ```
3. shell-command-on-region:使用外部命令对所选择的区域进行处理。
4. call-process/start-process:调用外部程序；开始外部程序。前者同步的，后者异步的。 
5. call-process-shell-command/start-process-shell-command:使用shell命令同步或者异步打开外部程序。

### Environment Variables(环境变量)
1. getenv:获得当前系统的环境变量值
   ``` lisp
   (getenv "HOME") ;;==>"/home/wtx"
   ```
2. setenv:设置环境变量的值
   ``` lisp
   (getenv "JAVA_HOME" "/usr/local/java") ;;==>"/usr/local/java"
   (setenv "LANG" "en_US.UTF8")  ;;==>"en_US.UTF8"
   ```
3. system-type:变量，指向当前系统类型。
    ``` lisp
    system-type ;;==>gnu/linux
    ```
4. process-environment:当前系统所有环境参数集合，是一个列表，遍历便可以单个变量的名称和相应的值。
