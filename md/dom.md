8.20学习打卡：

- React 入门与实战
- [x] DOM和虚拟DOM的概念
- [x] 虚拟DOM的本质和目的
- [x] 介绍Diff算法的概念
- HTML 入门与实战
- [x] Web基础
- [x] HTML入门
- JavaScript入门与实战
- [x] Javascript快速入门







DOM和虚拟DOM的概念：

dom：DOM就是文档对象模型

虚拟dom：（）





虚拟DOM的本质和目的以及与DOM的区别：

dom的本质是什么：浏览器中的概念，用js对象来表示页面上的元素，并提供了操作DOM对象的api

什么是React中的虚拟DOM：是框架中的概念，是程序员用js对象来模拟页面上的DOM和DOM嵌套

区别：

- DOM：浏览器中，提供的概念；用js对象，表示页面上的元素，并提供了操作元素的API；

- 虚拟DOM：是框架中的概念；是开发框架的程序员，手动用js对象来模拟DOM元素和嵌套关系
  - 本质：用js对象，来模拟DOM元素和嵌套关系；
  - 目的：就是为了实现页面元素的高效更新；



1.为了将性能做到最优，按需要渲染页面（更新时只更新渲染更新的数据所对应的页面元素）

2.如何实现页面的按需更新呢？

DOM树的概念：

一个网页呈现的过程：

1. 浏览器请求服务器获取页面的HTML代码
2. 浏览器要先在内存中，解析DOM结构，并在浏览器内存中，渲染出一棵DOM树；
3. 浏览器把DOM树，呈现在页面上；

3.如何获取到新旧DOM树，从而实现DOM树的比对呢？

​	分析：浏览器中，并没有直接提供获取DOM树的API：因此，我们无法拿到浏览器内存中的DOM树；

4.程序员可以手动模拟新旧两棵DOM树；

5.程序员手动模拟的这两棵新旧DOM树，就是React中虚拟DOM的概念；

总结：什么是虚拟DOM：用JS对象的形式，来模拟页面上DOM嵌套关系；（虚拟DOM是以JS对象的形式存在的）

这就是React中虚拟DOM的概念；

本质：用JS对象，来模拟DOM元素和嵌套关系；

目的：就是为了实现页面元素的高效更新；





Diff算法

- tree diff：新旧两棵DOM树，逐层对比的过程，就是tree Diff；当整棵DOM逐层对比完毕，则所有需要被按需更新的元素，必然能够找到；
- component diff：在进行Tree Diff的时候，每一层中，组件级别的对比，叫做Component Diff；
  - 如果对比前后，组件的类型相同，则暂时认为此组件不需要被更新；
  - 如果对比前后，组件类型不同，则需要移除旧组件，创建新组件，并追加到页面上；
- element diff：在进行组件对比的时候，如果两个组件类型相同，则需要进行元素级别的对比，这叫做Element Diff；



Web基础

**浏览器内核**

浏览器主要功能：渲染

1. Trident(IE内核)

   win10后转为Edge，内核为EdgeHTML

2. Gecko(firefox)

3. webkit(Safari)

4. Chromium/Bink(chrome)

5. Presto(Opera)

   

**Web标准**

Web标准构成

Web标准不是某一个标准，而是由W3C组织和其他标准化组织指定的一系列标准的集合。主要包括结构(Structure)、表现(Presentation)和行为(Behavior)三个方面。

**结构标准**：结构用于对网页元素进行整理和分类，主要包括XML和XHTML两个部分。

**样式标准**：表现用于设置网页元素的板式、颜色、大小等外观样式，主要指的是CSS。

**行为标准**：行为是指网页模型的定义及交互的编写，主要包括DOM和ECMAScirpt两个部分。

理想状态我们的源码：.html .css .js



**HTML初识**

HTML（Hyper Text Markup Language）中文译为”超文本标签语言“，主要是通过HTML标签对网页中的文本、图片、声音等内容进行描述。

html结构：

```
<html>
	<head>
		<title></title>
	</head>
	<body>
	</body>
</html>
```



**标签及标签关系**

网页组成元素：文字、图片、链接、视频、音频...

我们需要用代码来表达这些元素



代码标签：

双标签：

```html
<img> 图片标签
<p>   段白哦前
<br />换行标签
标签用于表达元素
```

单标签：

```html
<br />
```



- 嵌套关系

```html
<head>
    <title></title>
</head>
```

-   并列关系

```html
<head>
    <title></title>
</head>
<body>
</body>
```

一般框架：

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
</head>
<body>
再页面中输入一下2个单词
</body>
</html>
```





**JavaScript入门与实战**

javascript介绍

JavaScript是一种轻量级的脚本语言，也是一种嵌入式(embedded)语言，是一种**对象模型**语言，简称JS、

1. 基本的语法结构（比如操作符、控制结构、语句）
2. 标准库

想要实现其他复杂的操作和效果，都要依靠**宿主环境**提供API，目前，已经嵌入JavaScript的宿主环境有多种，最常见的环境就是浏览器，另外还有**服务器**环境（操作系统）



JavaScript现在的意义（应用场景）

JavaScript发展到现在几乎无所不能。

1. 网页特效
2. 服务器开发（Node.js）
3. 命令行工具（Node.js）
4. 桌面程序（Electron）
5. APP（cordova）
6. 控制硬件-物联网（Ruff）
7. 游戏开发（cocos2d-js）



声明变量并赋值

变量命名规则







8.28学习打卡：

- React 入门与实战

- [ ] Node和Chrome之间的关系

- [ ] webpack-dev-server的基本使用

- [ ] 配置html-webpack-plugin插件

- HTML 入门与实战

  HTML标签

- [x] 标签的语义化及标题标签

- [x] 段落标签和水平线标签

- [x] 课堂案例-新闻案例

- [x] 换行和div span标签

- [x] 文本格式化标签

- [x] 标签属性

- JavaScript入门与实战

- [x] 变量

- [x] JavaScript数据类型





## HTML 入门与实战

### 标签的语义化

#### 标签的语义化即：标签的含义

##### html标签

###### 排版标签

```
<h1>、<h2>到<h6>
```

###### 段落标签

```
<p> 文本内容 </p>
```

###### 水平线标签

```
<hr />是单标记
```

###### 换行标签

```
<br />
```

######  div span标签

div span是没有语义的，是我们网页布局的两个盒子

div为division的缩写

span为跨度，跨距，范围

语法格式：

```
<div>这是头部</div>	 <span>今日价格</span>
```

###### 文本格式化标签

```
<b></b><strong></strong>加粗
<i></i><em></em>斜体
<s></s>和<del></del>删除线
<u></u>和<ins></ins>下划线
```

###### 标签属性

属性就是特性

使用html制作网页时，如果想让Html标签提供更多的信息，可以使用HTML标签的属性加以设置。其基本语法如下：

```
<标签名 属性1=“属性值1” 属性2=“属性值2” ...>内容</标签名>，类似键值对
如长度500的水平线，颜色为红色
<hr width ="500" color="red" />
```



## JavaScript入门与实战

##### 变量声明

```
var a;
aa=18;
或者var aa=18;

var a=1,b=2,c=3;
```

##### 变量命名规则

1. 变量的名字必须是数字、字母、下划线_和$组成;
2. 变量的名字不能以数字开头;
3. 变量的名字不能是关键字;
4. 建议变量名有意义;
5. JS中变量区分大小写;
6. 命名尽可能用哪个驼峰法如userName;

##### javascript数据类型

- 数值Number

- 字符串String

  使用单引号或者双引号

  ```
  var str=' 我是\'王大\' '；
  alter(str);
  
  var s1='123';
  console.log(s1.length);
  
  //+既可以作为数学运算使用也可以作为字符串拼接使用
  //从前往后进行运算，如果两个变量都是数值型，那么+作为数学运算符
  //直到遇到一个字符串，此后所有+都是字符串拼接
  var s2='456';
  var s3=s1+s2;
  console.log(s3);
  
  var s4=1;
  console.log(s4+s3);
  
  ```

- 布尔Boolen

  Boolen字面量：true和false，区分大小写

  计算机内部存储：true为1，false为0

- undefined

  undefined表示一个声明了但没有赋值的变量，变量只声明的时候，默认为undefined

- null

  null表示一个空，变量的值如果想为null，必须手动设置

- 对象Object

  复杂数据类型

##### 代码注释

```
//单行
/*多行*/
```

##### 其他类型转换为字符换

```
var n=5;
n.tostring();
console.log(typeof s);

console.log(typeof String(n));

var s=''+n;
console.log(typeof s);

var n=true;
console.log(typeof n.tostring());
```

##### 数据类型转换

```
var a='1';
var b=Number(a);
console.log(typeof b);

var c=Number('c');//Nan，Nan不是个数值
var d=Number(null);//0
var e=Number(undefined);//nan

var a=parseInt('2');//2
var b=parseInt('k23');//Nan
var c=parseInt('null');//Nan
var d=parseInt('undefined');//Nan

var a=parseFloat('1.23df');//1.23
var b=parseFloat('1.3.4.5');//1.3
var c=parseFloat('h34');//Nan
var d=parseFloat('null');//Nan
var e=parseFloat('undefined');//Nan

```

##### 布尔类型转换

```
var a=Boolean('0');//true 字符串有内容都是true
var b=Boolean(0);//false
var c=Boolean('2');//true
var d=Boolean(null);//false
var e=Boolean(undefined);//false
var f=Boolean('');//false
var f=Boolean(' ');//true
```





8.29学习打卡：

- React 入门与实战

- [ ] Node和Chrome之间的关系

- [ ] webpack-dev-server的基本使用

- [ ] 配置html-webpack-plugin插件

- HTML 入门与实战

- [ ] 图像标签
- [ ] 链接标签
- [ ] 锚点定位
- [ ] base标签
- [ ] 特殊字符
- [ ] 注释标签

- JavaScript入门与实战

- [ ] JavaScript操作符与运算符
- [ ] JavaScript语句