---

title: emacs完全补完计划（二）——简单配置和设置
date: 2016-02-01 23:34:00
tags: ["emacs", "IDE"]
---

<!-- toc -->

## 我们在谈论lisp的时候谈论什么？
> 很多emacs教程都想避开Emacs Lisp(下面简称elisp)，完全不可能，反而学习过程中会磕磕碰碰。其实lisp作为一个古老的语言，语法上确实相比现代语言看起来看起来怪异很多，但并不落后，反而有独到之处，简洁易懂。在函数式编程复兴的浪潮中，去了解函数编程的始祖显得意义非凡。之后的内容在配置过程中不断回顾elisp的语法和用法，先提三点：

<!--more-->
### 操作符前置

程序的本质是什么，无非是数据本身和操作数据过程。lisp把操作数据的“操作符”前置了，比如平常计算多个数加法的时候用 **1+2+3+4+5**，lisp这样做： **(+ 1 2 3 4 5)**，括号是lisp的特性，左括号的往右的第一个字符就是所谓的“操作符”，用空格来区别不同的参数。这样做有什么好处？现代语言因为更偏向自然语言，操作符的位置更加灵活，比如调用一个方法的时候：`object.method(args)`，方法需要一个对象支撑，自己变成了附庸品，后置了；几乎所有连接字符串都可以这么操作：`"HE"+"LLO"+"WORLD"`，操作符`+`位于中间；还有最后一种情况，常见的 `Assert statement` 或者内置函数调用 `ABS(num)`又把操作符前置了，在操作符使用过程中混乱不协调，容易迷惑然后出错，所以灵活也有它的弊端。括号和操作符前置两个特性让lisp独特而优雅，组成了为人称道的S-expression(S表达式)，以后还会提及它的好处。

### 数据结构
将操作符前置是高度统一又美好理想的理念(别给我扯汇编)，因为纯数据的存在导致不可能所有括号里面的最左边都是操作符。lisp其实叫做列表处理语言，这里就引出了lisp里面最基础的数据结构--list，还有它的组成元素atom(原子)。其实我们已经见过了list就是**(+ 1 2 3 4 5)**，只有在括号括起来的表达式就叫做一个list，里面的元素只要不是另外一个list就叫做原子(atom)，意为不能再分解的意思。那么纯数据该怎么表示？像这样`(1 2 3 4 5 )`，显然不符合操作符前置的原则，会报错说找不到名为`1`的函数，于是考虑再三就把操作符扩充到了括号之外，变成`'(1 2 3 4 5)`表示整个括号里面不执行，直接返回整个list。像**(+ 1 2 3 4 5)**这样的list会执行，返回值为15；而`'(+ 1 2 3 4 5)`会直接返回(+ 1 2 3 4 5)本身。

### 返回机制
上面提到了返回，在其他大部分的语言里需要一个关键字`return`来实现，而lisp只要遇到括号就代表自动返回值，可以执行括号内容后返回，也可以根据括号前面的`'`单引号只返回括号本身。这时候就有人想在返回的时候部分内容能够执行就好了，比如`(5 (* 2 5) 15 20)`，如果返回的时候里面的表达式`(* 2 5)`能直接执行的话整个表达式就能返回`(5 10 15 20)`这样的数列。于是就有人想出了用外层的括号来控制内层的括号，变成**`(5 ,(* 2 5) 15 20)**，外层加个反引号内层加个逗号就能返回以5为倍数的数列了。

## 配置文件位置
在linux 下，`sudo touch ~/emacs.d/init.el`，其中**init.el**文件是emacs 配置文件，用elisp写的文件都以el为拓展名。
在window下新建一个文件夹`C:\Users\username\AppData\Roaming\.emacs.d`，其中username为本机的名字，在这个文件夹下新建一个init.txt，把它改为init.el即可。

## 常见配置方法

> 在这节只会提及一些elisp常见的关键字，这些关键字经常会困扰着新手，首先要知道elisp用`-`来连接声明不同功用的变量名和函数名的。

### `nil` 和 `t`
nil和t在elisp里面分别代表着false和true，用单个字母`t`来代表true让人措手不及。和其他语言一样，正整数也表示true，负整数表示false。

### `require`
顾名思义它的作用就是引用包，后面一般加着包名，至于具体的设置和用法会在下一篇文章详细分析，只要知道它和其他语言中的import，require一样把其他模块或者包的上下文空间引入当前文件下。

### `setq`
setq作用是赋值，一般这么用(setq variable value)，比如`(setq kill-ring-max 200)`让emacs剪切板存放的条目数上限为200。

### `add-to-list`
添加一个元素到某个list里面，这样用`(add-to-list LIST ELEMENT)`。也可以看出除了用单个变量来控制某种配置外，emacs里面还存在各种list这种容器式数据结构，用来映射或者存放一组变量。

### `add-hook`
添加钩子，比如(add-hook 'window-setup-hook 'toggle-frame-maximized)，作用是当window启动的时候frame最大化。在添加钩子的时候要查看是否有以`-hook`结尾的函数。

### `global-set-key`、`define-key` 以及`kbd`
`kbd`是一个宏，至于什么是宏，我们先暂按不说，它的存在就是为简化定义一系列按键输入（键序列），比如(kbd "C-x C-f")会返回control+c 然后control+f的输入操作。`global-set-key`用来定义按键绑定的函数，之前提到过emacs里面的任何操作都是由一个或者多个函数组成，结合上面的`kbd`宏，emacs这样定义的一个按键的功能——`(global-set-key (kbd "C-x C-f") 'find-file)`。那么`define-key`又是什么？global-set-key显而易见是用来定义全局的，针对于整个emacs而言的，污染性太大，define-key则用来定义针对某个mode才有的按键，也就是说当我们进入到某个mode的时候这个按键才生效。常见用法`(define-key keymap function)`，keymap保管了key和function的映射表，一般每个mode都会提供了一个keymap用于用户自定义。
如果不知道自己输入的键序列在`kbd`里面是怎么写的，可以使用`C-h k`来查看有没有此键序列的绑定，如果没有绑定任何的函数会返回这个键序列本身，就是写入`kbd`里面的内容。

## 一份通用配置说明
> 这份配置几乎可以说是最基本的配置，没有安装其他的package，仅仅是内置和外观方面的设置。

```lisp
;;禁止菜单栏
(menu-bar-mode -1)
;;禁止显示工具栏
(when (fboundp 'tool-bar-mode) (tool-bar-mode -1))
;;禁止显示滚动条
(when (fboundp 'scroll-bar-mode) (scroll-bar-mode -1))
;;禁止emacs一个劲的叫
(setq visible-bell t);
;;在console，不想看到屏幕不停的闪
(setq ring-bell-function (lambda () t))
;;关闭启动是的那个“开机画面”
(setq inhibit-startup-message t)
;;设置剪切板最大条目数为200
(setq kill-ring-max 200)
;;一行显示最多显示80列
(setq default-fill-column 80)
;;缺省的major-mode为text-mode
(setq default-major-mode 'text-mode)
;;括号匹配时显示另外一边的括号
(show-paren-mode t)
(setq show-paren-style 'parentheses)
;;显示语法高亮
(global-font-lock-mode t)
;;设置编码
(set-buffer-file-coding-system 'utf-8)
(set-terminal-coding-system 'utf-8)
(set-keyboard-coding-system 'utf-8)
(set-selection-coding-system 'utf-16-le)
(set-default-coding-systems 'utf-8)
(set-clipboard-coding-system 'utf-8)
(set-language-environment "UTF-8")
(prefer-coding-system 'utf-8)  
(set-file-name-coding-system 'gb18030)
;;内部有个自动补全功能，根据当前buffer的内容、文件名、剪切板等自动补全
(setq hippie-expand-try-functions-list
      '(
        try-expand-dabbrev
        try-expand-dabbrev-visible
        try-expand-dabbrev-all-buffers
        try-expand-dabbrev-from-kill
        try-complete-file-name-partially
        try-complete-file-name
        try-expand-all-abbrevs
        try-expand-list
        try-expand-line
        try-complete-lisp-symbol-partially
        try-complete-lisp-symbol))
;;按ALT+/ 键进行补全
(global-set-key (kbd "M-/") 'hippie-expand)
;;用ibuffer替换默认的buffer管理器
(global-set-key (kbd "C-x C-b") 'ibuffer)
;;用正则搜索替换默认搜索
(global-set-key (kbd "C-s") 'isearch-forward-regexp)
(global-set-key (kbd "C-r") 'isearch-backward-regexp)
;;一个好用的minibuffer插件ido，许多插件都基于它。
(ido-mode t)
(setq ido-enable-flex-matching t)
```
可以看出除了数字、nil和t之外所有的值都需要加`'`来传值，如果不加表示执行这个值，加了表示只需要返回它本身的字符。

## 内置的自定义方法
显然虽然可以通过package说明来定义配置，但每个package包含了大量的自定义变量或者函数，使用`M-x customize`则会出现一个buffer列出了可自定义的类别，进入每个类别当中，又有各个package的名字，再随便选择一个package的名字就会展开这个package可定义的变量或者函数，每个条目都包含了此条目的说明以及现有默认的值。

![package](../images/packages.png)

比如`M-x customize`，进入 **Faces**这个用来配置emacs外观的配置组，再深入到**Basic Faces**，就可以看到一系列的变量和函数，接着选择 **Cursor face**(用来配置光标外观的)，在最左边按下enter键，就会展开一个**Choose**按钮，点击则会列出各种颜色，随便选一个，选完点最上面用`apply`或`apply and save`两个按钮，光标的颜色就变成了刚刚选取的颜色。
其实打开**init.el**文件也可以看到文件末尾增加了两个函数，也就是说在emacs内置的自定义操作最后还是以文字的形式保存到了init.el里。增加的函数如下：
```lisp
(custom-set-variables
 ;; custom-set-variables was added by Custom.
 ;; If you edit it by hand, you could mess it up, so be careful.
 ;; Your init file should contain only one such instance.
 ;; If there is more than one, they won't work right.
 )
(custom-set-faces
 ;; custom-set-faces was added by Custom.
 ;; If you edit it by hand, you could mess it up, so be careful.
 ;; Your init file should contain only one such instance.
 ;; If there is more than one, they won't work right.
 '(cursor ((t (:background "DarkOliveGreen1")) (t nil))))
```
## 本文参考
1. [emacs-tutor](http://tuhdo.github.io/emacs-tutor.html)
2. [我的emacs配置](http://emacser.com/dea.htm)
