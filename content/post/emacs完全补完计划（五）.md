---

title: emacs完全补完计划（五）——歌剧魅影Elisp
date: 2016-02-28 20:18:06
tags: ["emacs", "tutorial"]

---
<!-- toc -->
## 闲话elisp
> 如果说emacs是一出庞大的歌剧，那么elisp则是撑起全场的主角，它犹如一道魅影，神秘、黑暗、引人入胜，在剧初以无以伦比的优雅身姿和独一无二的音色试图唤起观众的灵魂共鸣；当有人尝试接近它的时候，或许期望越大失望越大，藏在另一半面具下是人性特有的丑陋，是受世人侮辱唾弃后难掩的愤怒和敏感；而最后它高尚的选择和坚持让作为学习者的我们不断反思，什么叫做美？不是世俗的外在的可视的，是来源内在的人性之光，能够让灵魂起舞。事实上扯远了，不过大概总结了学习elisp的三个阶段，首先被洗脑认为lisp系语言如同武林传说中的秘籍邪典，迷迷糊糊建立了莫名的崇拜，后来深入学习才发现lisp系语言有固有的缺陷，致使它们在现代背景下难有立足之地，最后在完全了解它们了，发现在语言演变的历史长河它们是那么的独一无二，具有难以言表在时间的美，仿佛洞开了一个新世界。lisp相对现在很古老，在这期间有很多篇文章试图向别人灌输它的思想，这篇文章也是其中一篇，但lisp系语言太庞大，就elisp本身的官方手册都有一千多页，要在一篇文章里面完全展现它是不现实的，只能按笔者自己的有限理解来书写，可能夹杂着不恰当的比喻，希望观者有所悟。

<!--more-->

## 七言七日七符
> 中国古代只用“七言”寥寥数语就表达出了无尽的情感和哲思，简直是“少即是多”的典范。而在西方传闻上帝仅仅花了七日便创造了世界。so what？说明“七”是吉祥的数字......传说lisp语言在创造之初也仅仅存在七个操作符。分别是 quote,atom,eq,car,cdr,cons以及 cond，如果从这七个角度出发就能化繁为简，理解lisp语言之源。之前的文章有提到三点**操作符前置**、**数据结构**以及**返回机制**，其实都是针对lisp语言的最大特征--**括号**而言的，在括号里面操作符会前置，遇到括号就执行并返回的机制，在括号外部引入引号让括号返回本身。实际上括号的只体现了lisp语言构成二分一的元素--语法。而七个操作符体现了语言的另一半组成--语义。


### atom
atom翻译过来是原子，不可分割的意思。可以联想到古代由于科学不发达，许多人都想找到世界里最基本东西，用来理解世界的本质或者掌握其中的运行规律。一个经典的模型就是“五行”--金木水火土，世间万物的都是由这五种基本元素组成的。基本元素的概念其实就对应着atom，语言中一些基础类型是atom的一种。比如最常见的字符串、数字还有布尔值都是。之前有讲过elisp里面用后缀为`p`的函数来判断一个object的类型，比如`stringp`、`numberp`等等，可是没有`atomp`，因为atom只是一个笼统的概念，没有实际对应的类型，就干脆把atom当做判断符，用来判断一个object是否是atom。

``` lisp
(atom "hello") ;;==>t
(atom 233)  ;;==>t
(atom nil)  ;;==>t
(atom :keyword) ;;==>t
```

实际上elisp用`type-of`返回一个object的具体类型。

``` lisp
(type-of "hello") ;;==>string
(type-of 233) ;;===>integer
(type-of nil) ;;===>symbol
```

### cons、car、cdr
不得不吐槽一下知道atom这个概念并没什么卵用，其实是为了理解cons这个概念做铺垫。cons可以翻译成序对或者点对，个人更倾向于后者。回到“五行”说，当我们手握五大元素的时候会做什么能更有趣？让它们发生关系！这就是所谓相克相生的运行体系，譬如钻木取火、水来土掩、水火不容等等...`cons`就是把两个基本元素(atom)结合的一种数据结构，让单独的atom彼此建立联系，`cons`这个字符本身就是建立这种结构的函数。如下图所示：

可以看出`cons`返回的形式是`(1 . 2)`，所以又可以叫做点对结构。

``` lisp
;;consp是用来判断是否是一个点对结构的函数。
(consp (cons "fire" "water"))  ;;毋庸置疑返回t
(consp '("fire"."water"))  ;;居然也返回t，说明也可以用'(xx.oo)的形式构造一个点对。
```

而`car`和`cdr`是对cons最基本的操作，比如`(car (cons "阴" "阳"))`会返回**阴**，cons的头部；`(cdr (cons "阴" "阳"))`则返回**阳**，cons的尾部；相当于一个负责“采阴”，另一个负责“取阳”。
`cons`犹如雌雄、阴阳、黑白、天地等相互对立又相互依存的统一矛盾体，不禁让人联想到太极，也如同太极能够演化出两仪四象八卦万物一样，`cons`理论上也能构造出任何的数据结构，简单而强大。不信请看：
之前常常提到的list（列表）也是一种特殊的cons，上面第三个例子就是list的结构图，不仅要有“你和我”而且还要大家围在一起手牵手。`(cons 1 (cons 2 (cons 3 (cons 4 nil))))`这样生成list方式太麻烦所以创造了一个`list`的函数。

``` lisp
;;下面两个表达式是完全相等的
(cons 1 (cons 2 (cons 3 (cons 4 nil))))  ;;==>(1 2 3 4)
(list 1 2 3 4)  ;;==>(1 2 3 4)
;;将233加入到一个list的头部，并返回加入之后的新list
(cons 233 (list 1 2 3 4))  ;;==>(233 1 2 3 4)
;;用consp来判断
(consp (list 1 2 3 4))  ;;==> t
```
由于cons存在无限的可能，`car`和`cdr`这种一次只取一步的操作显然就不合时宜。于是乎创造出各种延伸变异版本的car和cdr，简直丧心病狂。

``` lisp
;;X代表某种cons复合结构。
(caar X)  ;;Return the car of the car of X.
(cddr X)  ;;Return the cdr of the cdr of X.
.
.
.
(caaaar X) ;;Return the `car' of the `car' of the `car' of the `car' of X.
(cddddr X) ;;Return the `cdr' of the `cdr' of the `cdr' of the `cdr' of X.
```

实际上，`cons`更常用于表示概念名称和具体实物之间的关联，像`(cons key value)`或者`'(key . value)`便可以表达一个物体。更实际的是，虽然cons结构能千变万化，但是对内存的效率却不高，所以lisp方言们都吸取了其他语言的数据结构，比如数组(array)、哈希表(hash table)等等来提高操作数据的效率，诚然，这样会让lisp方言们摇身一变成了“通用性的语言”，更易于其他程序猿的接受和自身的推广，但点对结构不该被遗忘或抛弃，它是lisp纯粹本质的体现之一。
### quote
`quote`在之前已经提到过了，其实等同于`'`，引号是quote的简写，`(quote a)`完全等同于`'a`。之前还提到过因为lisp语法只要遇到括号就会根据前置的“操作符”求值返回，`quote`的存在就是对抗这种固化的机制而存在的。可以说没有quote的lisp里面都是“一个求值过程”，quote不求值只返回，数据就很容易构造出来，说到底quote 的存在是由于lisp代码和数据都是都用统一的S表达式带来了困扰。

``` lisp
(quote (1 2 3 4))
;;等同于
'(1 2 3 4)
;;等同于
(list 1 2 3 4)
```

这种不求值只返回的机制不仅仅用于构建数据，自然还可以让代码不求值返回，让代码之上又有的一层机制，为神奇的宏的出现埋好了伏笔。

### eq
判断相等的，我至今不明白为什么eq会位列七个操作符之内，它看起来并不是很重要。它的定义是：(eq x y)，如果x和y是同一个原子(atom)或者空表(cons)，则返回t，否则返回nil。eq是最表面的判断，equal才是更深入的判断，之后会罗列elisp里面所以和判断相等有关的函数。

### cond
cond是控制结构，类似其他语言里面的select-case控制功能，为什么七个操作符里面没有if，大概cond可以判断一种或者多种条件进行跳转，if只能一个或者两个条件，多种条件要不停的嵌套。
形如：

``` lisp
(cond 
    ((条件1) 表达式1)
    ((条件2) 表达式2)
    ((条件3) 表达式3)
    ((条件4) 表达式4)
)
```

从上到下依次执行，直到遇到条件表达式返回值为t，则将该条件后面表达式的值当做整个cond表达式的返回值。

## elisp的基本数据类型
> 回到elisp，基本数据类型有些和其他语言差不多，限于篇幅，挑重点讲。

### integer
整型，#bNNN表示二进制数，同理有#o(八进制数)、#x(十六进制数)。
### float
INF表示无穷大。
### Char
Char其实就是integer，字符型用整数型对应的编码表示了而已。可以用`?字母`来打印字面量。

``` lisp
(prin1 ?a) ;;==>97
(prin1 ?A) ;;==>65
```
也和其他语言的一样，`\n`来表示空一行，`\t`表示TAB键等等，elisp里面多了几个：

``` lisp
?\a ;=> 7                 ; control-g, `C-g'
?\b ;=> 8                 ; backspace, <BS>, `C-h'
?\t ;=> 9                 ; tab, <TAB>, `C-i'
?\n ;=> 10                ; newline, `C-j'
?\v ;=> 11                ; vertical tab, `C-k'
?\f ;=> 12                ; formfeed character, `C-l'
?\r ;=> 13                ; carriage return, <RET>, `C-m'
?\e ;=> 27                ; escape character, <ESC>, `C-['
?\s ;=> 32                ; space character, <SPC>
?\\ ;=> 92                ; backslash character, `\'
?\d ;=> 127               ; delete character, <DEL>
```
## elisp的容器类型
> 也是基础的数据结构，因为具有容纳的作用，而且彼此之间是有联系的，所以独起一章。这么多的容器式数据类型，区别它们的无非两点：存储的数据类型和底层的存储形式。关系如下图：

Sequence（序列）是hash-table外所有容器类型的总称了。用`(length aSequence)`获取Sequence的长度，`(elt sequence N)`来获取第N个元素，对任何一种序列都有用。

### List
list是特殊的cons点对结构，是emacs里面最常见的数据类型。

1. 创建list:使用list函数`(list 1 2 3 4)`或者`'(1 2 3 4)`
2. 获取list里面的元素

| 函数           | 功能                        |
|----------------|-----------------------------|
| (car X)        | first element               |
| (nth n X)      | nth element (start from 0)  |
| (car (last X)) | last element                |
| (cdr X)        | 2nd to last elements        |
| (nthcdr n X)   | nth to last elements        |
| (butlast X n)  | without the last n elements |

3. 在list头部添加元素:(cons element mylist)
4. 合并两个list:(append list1 list2)
5. 判断list是否为空:**null**
6. 判断某个元素是否存在：`(member element X)`
7. 返回元素所在的位置:`(position element X)`

#### alist(association list)
alist是elisp里面非常重要的概念，理解了点对结构之后，alist其实是把一组cons组织成一个列表，每个元素就是一个cons。形如：
``` lisp
(list
    '(key1 . value1)
    '(key2 . value2)
    '(key3 . value3)
)
```
emacs里面组织存放一组变量的最常见的形式。值得一提的是alist里面的元素是顺序的，且允许重复键值。
一个非常有用的函数**assoc**，用key来寻找value，key的匹配使用的是equal。这么用：

``` lisp
;;构建一个alist，并把它赋给dict
(setq dict
'((pine . cones)
 (oak . acorns)
 (maple . seeds)))
;;使用assoc，找到对应的cons并返回
(assoc 'oak dict) ;;==>(oak . acorns)
;;结合car、cdr可以把cons里面key和value分别取出来
(car (assoc 'oak dict))  ;;==>oak
(cdr (assoc 'oak dict))  ;;==>acorns
```
#### plist(Property list)
顾名思义，属性列表，其作用和alist相似，不过语法更加清晰些。形如：

``` lisp
'(:key1 value1 :key2 value2 :key3 value3)
```
注意`:`是不能省略的，而且key的值必须是唯一的。用plist-get取出值，plist-put来加入新的值。

``` lisp
;;构建一个plist，并把它赋给plst
(setq plst (list :buffer (current-buffer) :line 10 :pos 2000))
;;plist-get取出值
(plist-get plst :pos) ;;==>2000
```
### Array 
创建Array需要指定一个固定长度，除了char-table，这点限制了它的使用。
除了从sequence继承过来的**length**和**elt**函数，array本身也有几个函数，对下属的类型都有效。
设置元素位置和值：`(aset array index value)`

#### Vector
1. 创建一个Vector：两种形式`[v1 v2 v3 ...]`和`(vector v1 v2 v3 ...)`，显然引入了`[]`，在一堆括号里面也另类。
2. 将多个sequence合并成为一个vector：`(voncat seq1 seq2)`
3. vector和list的转换：`(append vector nil)`

#### bool-vector
1. bool-vector是vector的子集，只能用来存放nil或t。
2. 创建 bool-vector：`(make-bool-vector LENGTH INIT)`。

#### char-table

#### string
1. string是不可变的，这点和大部分的语言一致
2. string可以包含文本属性properties，形如`#("str" property-data)`，property-data是以plist形式包含了string的一些属性。
3. string用`string=`和`string<`进行对比。

### Hash Table(哈希表)
> 在其他语言比较常见的类型，数据容量到达一定级别速度依旧非常快，没有特殊的顺序要求，但不能重复。如果是小型数据装载还是建议用list，大集合用哈希表。

1. 创建的哈希表时候可以根据一些关键字`:key`来指定初始属性。
   * :test 因为哈希表里面的key值要求不能重复，所以key值需要比较，这个关键字指定key值比较的函数，可以是`eq`、`eql`、`equal`，默认是`eql`。
   * :size 指定初始容量大小

2. 访问hash table相关的函数：
   * (gethash KEY TABLE)
   * (puthash KEY VALUE TABLE)
   * (remhash KEY TABLE):移除某个元素
   * (clrhash TABLE):清理hash table
   * (maphash FUNCTION TABLE):对每个元素都应用FUNCTION

3. 拷贝：`(copy-hash-table table)` 
4. 个数：`(hash-table-count table)`

## emacs基本数据类型小结
### 数据类型间的转换
1. number-to-string/string-to-number
2. concat可以将序列转换成字符串：

    ``` lisp
    (concat '(?a ?b ?c ?d ?e)) ; => "abcde"
    (concat [?a ?b ?c ?d ?e]) ; => "abcde"
    ```
3. vconcat 把字符串转换成向量

    ``` lisp
    (vconcat "abcde") ;;==>[97 98 99 100 101]
    ```
4. append原来是用来拼接两个list或者向list添加元素的，这里也可以把字符串转换成一个列表。
 
    ``` lisp
    (append "abcdef" nil) ; => (97 98 99 100 101 102)
    ```
5. 字母大小写：downcase、upcase、capitalize
6. 转换数字为float类型：`(float number)`
7. float转integer：ceiling、round

### 对比相等
1. `(eq OBJ1 OBJ2)`：只是简单的对比两个object是否相等。
2. `(eql OBJ1 OBJ2)`：比eq高级一个点，遇到数字的时候还会判断数字的类型和大小。

    ``` lisp
    ;;除了数值为1，一个为整数一个为浮点数，所以为nil
   (eql 1 1.0) ;;==>nil
   ;;类型和数值都一样才返回t
   (eql 1.0 1.0) ;;==>t
    ```
3. `(equal O1 O2)`：因为比上面两个都长，所以更加深入，比较类型、数值以及结构全部相等情况，最常用的比较。

    ``` lisp
    ;;下面左右两个都是cons结构的生成方式，所以返回t
    (equal '(1 . 2) (cons 1 2)) ;;==>t
    ```
4. `=`：数字比较，主要针对值，`(= 2 2.0)`返回t。 
5. `string=`和`string-equal`：两者是一样的，前者是后者的别名，比较字符串是否相等。

### 正则表达式
之前已经讲到正则常用的函数`string-match`和`look-at`，前者必须指定正则表达式和需要匹配的字符串，而look-at则是从此buffer的光标开始出进行匹配。
elisp的正则语法和其他语言基本一致，但有一点特别丑陋，像`(,) {,} \`等都需要转义才能使用，因为"\"也需要转义所以需要以"\\"形式转义其他字符，比如匹配四位数字的正则"[0-9]\\{4\\}"。

``` lisp
;;匹配2016这个数字
(string-match "[0-9]\\{4\\}" "欢迎2016的到来")
;;中间参数0代表第0分组，也就是所有分组。
(match-string 0 "欢迎2016的到来") ;;==>"2016"
```

## Symbol(符号)
### 概述
理解symbol是理解elisp的关键。当初在elisp成形的时候，面向对象的思想还没这么盛行，这种思想用继承和实例化的方式结构化地梳理代码间的关系，尤为重要的是对象状态的组织和存储以一种非常合理的形式呈现。elisp没有包含面向对象的思想，所以不现代很落后。就像用对象来指代一切很多现代语言一样，symbol是elip 里面的通用货币。symbol可以**同时**拥有下面四种值:
1. "name"，symbol-name符号名字
2. "value"，存储的变量值
3. "function"，存储的函数或者宏
4. "property list"，存储的属性列表

### symbol定义
symbol具有唯一的名字，它拥有四个Cell，cell应该翻译成“槽”或者“域”，作用就是存储指向以上四个值的“指针”。symbol 名字定义的规则很简单，就是`(quote name)`或者`'name`，单引号+普通字符，遇到特殊的字符该转义的还是得转义，比如:

``` lisp
(symbolp '+1)  ;;==>nil
(symbolp '\+1)  ;;==>t
;;使用symbol-name获得名字
(symbol-name '\+1) ;;==>"+1"
```
### 求值规则和symbol
现在完全总结一下elisp里面的求值规则，一共三种：
1. 第一种就是自求值，数字、字符串、cons以及nil、t等会直接返回它所拥有的值。
2. 第二种是本节讨论的符号，符号虽然是使用了单引号这种形式，但它会根据不同环境或者“操作符”从cells里面返回不同值。
3. 第三种是列表求值，之前说的“遇到括号就返回”，如果在括号外围有quote或者单引号只会返回列表本身。
也可以用`(eval 表达式)`方式来主动求值（evaluation），相当于抵消quote的作用。
``` lisp
;;下面两个表达式是完全相等的
(+ 2 3 4 5) ;;==>14
(eval '(+ 2 3 4 5)) ;;==>14
```
### symbol组成
用`(set (quote sym) val)`可将值赋给一个符号，看起来太麻烦了，用`(setq sym val)`是同样的效果，其中val不能是自求值类型，占有一个符号中的cell只能是一个不能自求值类型。如果是自求值类型的话symbol不再是symbol而只是一个**变量**。
1. value cell的存储和取出
``` lisp
;;如果是基本类型直接赋值的话，根本不会当符号处理，因为它们返回自求值
(setq num 123)
(symbol-value num) ;;==>*** Eval error ***  Wrong type argument: symbolp,123
(symbolp num) ;;==>nil
(type-of num) ;;==>integer
;;但是将num这个变量使用quote的话，就变成一个符号
(setq sym—num 'num)
(symbolp sym-num) ;;==>t
;;symbol-value 的作用是从cell里面取出符号并进行eval
(symbol-value sym-num) ;;==>123
;;也可以先判断一个symbol是否已经绑定一个变量值

```
2. function cell 的存储和取出
``` lisp
;;使用symbol-function取出操作
(symbol-function 'car) ;;==> #<subr car>
;;使用fset将上节sym-num这个符号的function cell存储一个函数
(fset sym-num 'car) ;;==>car
;;fboundp来判断是否有绑定函数
(fboundp sym-num) ;;==>t
;;用funcall来调用
(funcall sym-num '(a . b)) ;;==>a
```
3. property list属性列表，属性列表为了变量和函数存在的，用来存储附加属性或者状态。形如(pro1 value1 pro2 value2...)。
``` lisp
;;使用put 和 get 分配或者取出操作，貌似一次只能分配一次
(put sym-num :key "value1")
(put sym-num :color "blue")
;;":key"是一种特殊的符号，一般用在property list 和hash table 里面
(get sym-num :color) ;;==>"blue"
;;symbol-plist获取整个属性列表
(symbol-plist sym-num) ;;==>(group-documentation "The X Window system." :key "value1" :color "blue")
;;把symbol-plist单独提取出来自然可以用plist-get和plist-put操作
(setq my-plist (symbol-plist sym-num))
(plist-get my-plist :color) ;;==>"blue"
```
### string 和symbol转换
值得一提的是，字符串和符号之间可以直接转换。
``` lisp
;;将symbol 转换成string
(symbol-name 'sym-num)
;;将string 转换成symbol
(intern "minecraft")
```
未完待续......
