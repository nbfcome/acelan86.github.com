---
layout: post
title: "ecmascript学习笔记"
description: ""
category: ""
tags: []
---
{% include JB/setup %}


## 10 可执行代码

### 10.1 执行代码类型

*全局代码*

`全局代码`是指被作为`ECMA脚本程序`处理的源代码文本。一个特定`程序`的全局代码不包括作为`函数体`被解析的源代码文本。

{% highlight javascript %}
    var a = 1;          //全局代码部分

    function b() {      //属于全局代码
        return 1;       //不属于全局代码
    }                   //属于全局代码

    b();                //属于全局代码
{% endhighlight %}

> 其中，在函数b定义中的函数体不属于全局代码



*eval代码*

`Eval代码`是指提供给`eval`内置函数的源代码文本。更精确地说，如果传递给`eval`内置函数的参数为一个字符串，该字符串将被作为`ECMA脚本程序`进行处理。在特定的一次对`eval`的调用过程中，`eval代码`作为该`程序`的`#global-code`部分。

{% highlight javascript %}
    eval("var a = 1; //eval代码");
{% endhighlight %}



*函数代码*

`函数代码`是指作为`函数体`被解析的源代码文本。一个`函数体`的`函数代码`不包括作为其嵌套函数的`函数体`被解析的源代码文本。 
`函数代码`同时还特指`以构造器方式调用Function内置对象`时所提供的源代码文本。更精确地说，调用`Function构造器`时传递的最后一个参数将被转换为字符串并作为`函数体`使用。如果调用`Function构造器`时，传递了一个以上的参数，除最后一个参数以外的其他参数都将转换为字符串，并以逗号作为分隔符连接在一起成为一个字符串，该字符串被解析为`形参列表`供由最后一个参数定义的 函数体使用。
初始化`Function`对象时所提供的函数代码，并不包括作为其嵌套函数的`函数体`被解析的源代码文本。

{% highlight javascript %}
    function a() {
        return 1;       //函数代码
    }
    var b = new Function('return 1; //函数代码');
{% endhighlight %}

> 函数代码包括函数定义中的函数体，如函数a的函数体return 1， 也包括Function构造器生成的函数做为函数体的字符串部分，如函数b


#### 10.1.1 严格模式下的代码
