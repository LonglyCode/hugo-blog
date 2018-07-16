---

title: emacs完全补完计划（三）——package的引入和管理
date: 2016-02-02 23:30:39
tags: ["emacs", "IDE"]
---
<!-- toc -->

## package说明
> emacs中package（包）和其他编辑器一样，把一些功能或者特性包装成为一个package。

### package存放在哪里？
有package那么肯定需要知道存放在哪里，现在公认有两个稳定的仓库分别是ELPA和MELPA，当然也可以去emacswiki或者github 上直接下载。下面的程序在emacs24以上的版本下有用，将它加入到init.el文件里面去：
```lisp
;;内置的package管理，emacs24以上的版本有用
(require 'package)
;;将melpa仓库地址加入到package-archives这个列表变量里面
(add-to-list 'package-archives
    '("melpa" . "http://melpa.org/packages/") t)
;;这个是国内一个elpa仓库镜像，速度更快一些，你懂得
(add-to-list 'package-archives 
    '("popkit" . "http://elpa.popkit.org/packages/") t)
;;初始化package
(package-initialize)
;;更新仓库里面的package
(when (not package-archive-contents)
  (package-refresh-contents))
```

<!--more-->
### 如何下载和安装？
在emacs里面运行`M-x package-list-packages`，就进入package的下载界面。按 **n** 表示next，按**i**表示标记需要安装的包，按**u**表示取消标记，按**x**表示执行刚刚已经标记的包。
比如想安装emacs里面的VIM模拟器evil，可以先用**C-s**向下搜索evil，找到后按**i**再按**x**就下载到本地了。然后打开emacs的配置文件init.el，在里面添加下面：
```lisp
(require 'evil)
(evil-mode t)
```
重启emacs就算安装完成了。每个包都会说明如何在init.el里面添加配置，各有不同，可以到网上查找它的说明，不过一般都是`(xxx-mode)`或者`(xxx-mode t)`。
可以看到init.el路径下面多出了一个**elpa**文件夹，里面存放着我们刚刚下载过的package。

## package加载机制的前世今生
> 在前面两节完全可以安装和配置package了，但是会遗留许多问题，第一就是不够灵活，如果这个package不在已有仓库上面或者我们自己编写了一个`*.el`文件如何加到emacs配置里面？如果有很多个`*.el`文件分布到各个文件夹里面如何管理？如果有几个包是相互引用的，如何确定它们的加载次序？每次启动emacs的时候都全部加载会严重拖慢启动速度，怎么解决？怎么将一个自己写的某个函数挂载在某个package上面？要解决这些，我们需要了解emacs中引入package的内部机理。spacemacs有一篇写的很不错的emacs加载机制的[文章](https://github.com/syl20bnr/spacemacs/blob/develop/doc/LAYERS.org)，下面的内容相当于翻译加上自己的一些理解。

### load-file
最简单粗暴的加载方式：
```lisp
(load-file "~/elisp/foo.el")
```
这是最为原始的方式，填写的路径必须是绝对路径，这个路径也不会加入到emacs中load-path里（后面会提到）。它也不会优先寻找编译过`.elc`文件（显然编译过文件的会更快些）。这种方式已经被抛弃，仅作为历史提一下。
### Features
Features(特性)是emacs默认的另外一种常见的加载方式。相当于在`xxx.el`文件里面先注册，后在init.el里面引入，需要双方互相约定的协议。比如我编写了一个名为`my-feature.el`的文件，提供某些功能想加入到emacs里面，先在这个文件最底下写：
```lisp
;; Your code goes here ...
(provide 'my-feature)
```
其中provide（提供）就是所谓注册，后面跟的参数一定要和当前文件名一样。
然后再init.el文件里面加上：
```lisp
;; Your code goes here ...
(require 'my-feature)
```
那么在启动emacs之后会从一个叫features的列表里检查my-feature这个feature有没有存在，没有会查找叫做my-feature.el或者my-feature.elc文件，加载它，如果没有找到会提示错误。实际上这个机制可以让各个package根据`require` 出现在**init.el**文件里的顺序来加载的，一定程度上解决了package之间的相互依赖的关系。
在这里还有个问题，features机制提到会自动查找`.el`和`.elc`文件，那么它会在哪里查找，总不能搜寻硬盘里面的所有位置吧。所以需要一个列表来管理这些位置，名叫load-path，跟电脑里面的环境变量相似。
```lisp
(push "/some/path" load-path)
```
相当把某个路径添加到load-path里面了。可以用来管理分布在多个不同路径下的`.el`和`.elc`文件。之前从elpa或者melpa下载安装package其实已经自动把下载的package路径加入到了**load-path**里面，所以可以直接使用`require`来配置包。
### Auto-loading
使用require机制可能比粗暴load-file高明那么一丢丢，它解决了package加载顺序的问题，还有管理不同地方的elisp文件。但是它的加载方式也是比较暴力，每次emacs启动会全部一次性加载能查找到的所有文件，这将导致emacs的启动速度大大减慢。
auto-loading就是为了解决这种情况而诞生。你可以向emacs中注册一个函数，只有当调用或者使用这个函数的时候，包含这个函数的文件才会加载。这么用：
```lisp
(autoload 'some-function "some-file")
```
当调用some-function 时，加载some-file.el，再执行这个函数。autoload完整参数：`(autoload FUNCTION FILE &optional DOCSTRING INTERACTIVE TYPE)`，可以看出它除了提供加载文件地址外，还可以编写说明文档，在不加载文件时也能够查看它的用法。
autoload这种方法可以写入的到emacs配置文件里面，但明显不好管理，为什么不在函数定义处就指明它是一个autoload形式的函数呢？事实上可以用所谓的"魔术"注释来装饰一个函数的开头，让它autoload。
```lisp
;;;###autoload
(defun my-function ()
  ;; Source code...
  )
```
**;;;###autoload**是一种神奇的注释。当然autoload这种机制不局限于函数，可以用在一切可以定义的东西上面，比如宏、主模式、次模式等等。
### Eval after load
当我们加载一个package的时候想配置它，比如绑定一个自定义的函数等等。为了能够让我们的自定义的代码也autoload，使用with-eval-after-load。
```lisp
(with-eval-after-load 'helm
     ;; Some-Code
     )
```
当helm这个package加载后，some-code将接着执行，不管是使用features机制还是autoload机制。在emacs24.3方能使用with-eval-after-load，之前版本使用eval-after-load。
### Use-package
Use-package是一个第三方的package，使用前需要安装。Use-package可以说是为了解决之前提到的**所有**问题而出现的。Use-package是一个宏，来看看它是怎么使用的：
```lisp
(use-package helm)
```
最简单的加载，等同于(require 'helm)。

```lisp
(use-package helm
    :defer t)
```
`:defer`是个关键字，如果为t，则表示helm的里面的凡是被autoload的函数、宏、模式等都成立。
```lisp
(use-package helm
  :defer t
  :init
  ;; Code to execute before Helm is loaded
  :config
  ;; Code to execute after Helm is loaded
  )
```
增加了两个关键字:init和:config，顾名思义，针对package加载之前和package加载之后添加一些自己的配置。在:init中可以加载一些入口设定，比如定义一些按键调用这个package里面的函数。:config关键字后面的代码和with-eval-after-load功用是一样的，都是package实际被加载之后才会执行。
use-package还有很多关键字，比如指定某些条件下才会加载package，可以指定某些特定文件才加载这个package，可以进[官方文档](https://github.com/jwiegley/use-package)自行查看。
### org-babel-load-file 
org是和markdown一样是一种格式标记文本形式，可以用来编写文档的，但它比markdown厉害得多，提供功能更齐全，其中一种功能是将org里面编写的程序放到org-babel（org文件的某个块区域）里面，和markdown仅仅用来展示程序不同，org-babel里面的程序可以用`org-babel-load-file`来执行。
```lisp
(org-babel-load-file  "~/.emacs.d/config.org")
```
意味着你可以在一个org文件里面写出emacs的配置和相应的说明，是一种高效管理emacs配置的方式，初次听说还蛮刷三观的。不过因为加载速度的原因没能完全流行起来，不过还是有一些emacser坚持使用这种方式。
## 小结
use-package渐渐一统以前古老的配置方式，许多知名的emacser用这个包将自己的配置又重新写了一遍。因为它既包含了现有的所有定义方式又提供了许多关键字，让配置一个package简单易行，而且可以将所有的包都变成了延时加载，只有使用的时候再加载，大大加速了emacs的启动速度。
懂得以上所有的emacs加载机制后，基本上自己动手配置一个emacs配置不成问题。下篇文章会提到emacs能提供给我们的常用API。

## 附录

几个对新手友好的包

1. [better-defaults](https://github.com/technomancy/better-defaults):正如其名，是更好的默认emacs小设置 。
2. [which-key](https://github.com/justbur/emacs-which-key):按键提醒，当你按一个按键的时候，会弹出之后所有可能的按键以及说明。
3. [swiper](https://github.com/abo-abo/swiper):增强搜索和minibuffer的交互，包括一些模糊搜索。
4. evil-mode和evil-leader：vim党专用，emacs上的vim模拟器，功能十分完整，请自行搜索配置。

## 本文参考
1. [Spacemacs-layers](https://github.com/syl20bnr/spacemacs/blob/develop/doc/LAYERS.org)
2. [use-package](https://github.com/jwiegley/use-package)

