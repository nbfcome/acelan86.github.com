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

#### 全局代码 

{% highlight javascript %}
    var a = 1;          //全局代码部分

    function b() {      //属于全局代码
        return 1;       //不属于全局代码
    }                   //属于全局代码

    b();                //属于全局代码
{% endhighlight %}

#### eval代码

{% highlight javascript %}
    eval("var a = 1; //eval代码");
{% endhighlight %}

#### 函数代码

{% highlight javascript %}
    function a() {
        return 1;       //函数代码
    }
    new Function('return 1; //函数代码');
{% endhighlight %}


