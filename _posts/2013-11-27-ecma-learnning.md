---
layout: post
title: "ecmascript学习笔记"
description: ""
category: ""
tags: []
---
{% include JB/setup %}


# 10 可执行代码

## 10.1 执行代码类型

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


### 10.1.1 严格模式下的代码

*严格的全局代码*

`全局代码`以指令序言(`"use strict"` 或 `'use strict'`)开始

{% highlight javascript %}
    "use strict";
    var a = 1;
    alert(a++);
{% endhighlight %}

*严格的eval代码*

在严格模式下的代码，或者严格模式下直接调用eval函数的代码

{% highlight javascript %}
    "use strict";
    eval('var a = 1, b = 2;');
{% endhighlight %}

> 这里有个很关键的名词`直接调用eval`，什么叫做直接调用eval？

##### 直接调用eval

参考15.1.2.1.1，直接调用需要符合以下两个条件：

* 解释执行`CallExpression`中的`MemberExpression`的结果是个`引用` ，这个引用拥有一个`环境记录项`作为其基值，并且这个引用的名称是`"eval"`。
* 以这个`引用`作为参数调用`GetValue`抽象操作的结果是标准eval内置函数。

是不是很抽象？那么看下这些例子：

{% highlight javascript %}
    //example1:
    eval("var a = 1;");

    //example2:
    var otherEval = eval;
    otherEval("var a = 1;");

    //example3:
    (1, eval)("var a = 1;");
{% endhighlight %}

example1的`MemberExpression`执行后获取到的是eval这个引用，名称是“eval”，通过这个引用得到的值就是内置的eval函数，符合条件1，2。因此这是一个直接调用。


example2将eval内置方法赋值给了otherEval，当执行otherEval的时候，`MemberExpression`执行后获取到的是otherEval这个引用，这个引用的名字是“otherEval”。不符合1，因此不是直接调用.

example3中(1, eval)的结果不是引用，不符合1，因此也不是直接调用。


*严格的函数代码*

当一个`函数声明`、`函数表达式`或`函数赋值`访问器处在一段`严格模式下的代码`中，或其`函数代码`以指令序言开始，且该指令序言包含一个使用严格模式的指令序言时，该`函数代码`即为严格函数代码。
当调用内置的`Function`构造器时，如果最后一个参数所表达的字符串在作为`函数体`处理时以指令序言开始，且该指令序言包含一个使用严格模式的指令序言，则该`函数代码`即为严格函数代码。

不说话，直接上例子：

{% highlight javascript %}
    //example1
    "use strict";
    function a() {
        //严格模式
    }

    //example2
    function a() {
        "use strict";
        //严格模式
    }

    //example3
    new Function("'use strict'; //严格模式");
{% endhighlight %}



## 10.2 词法环境

一个词法环境由一个环境记录项和可能为空的外部词法环境引用构成， 用js来表达可能是这样
{% highlight javascript %}
    function LexicalEnvironment(envRecord, outerLexEnv) {
        this.environmentRecord = envRecord; //环境记录项
        this.outerLexicalEnvironment = outerLexEnv || null; //外部词法环境
    }
{% endhighlight %}

>`FunctionDeclaration`, `WithStatement` 或者 `TryStatement` 的 `Catch` 块这样的语法结构与词法环境相关联，类似代码每次执行都会有一个新的词法环境被创建出来。

### 10.2.1 环境记录项

环境记录项记录了在它的关联词法环境域内创建的标识符绑定情形， 用js表示为：
{% highlight javascript %}
    function EnvironmentRecord() {}
{% endhighlight %}


有两种类型的环境记录项：

*声明式函数记录项*

声明式环境记录项用于定义那些将`标识符`与语言值直接绑定的`ECMA脚本语法元素`，例如`函数定义` ， `变量定义` 以及 `Catch` 语句， 用js表示可能像下面这样：
{% highlight javascript %}
    function DeclarativeEnvironmentRecord() {} //认为他就是一个空对象
    extend()
{% endhighlight %}

例如，下面这段代码：
{% highlight javascript %}
    function foo() {
        var bar = 1;
        function inner() {

        }
    }
{% endhighlight %}

在进入foo函数体的时候，引擎执行了下面的步骤
{% highlight javascript %}
    var envRec = new DeclarativeEnvironmentRecord();
    //绑定函数声明
    envRec.inner = function () {};
    //绑定变量声明
    envRec.bar = 1;
    //生成词法环境
    var lexEnv = new LexicalEnvironment();
    lexEnv.environmentRecord = envRec;
    lexEnv.outerLexicalEnvironment = currentLexicalEnvironment;   
{% endhighlight %}


*对象式环境记录项*

对象式环境记录项用于定义那些将`标识符`与具体对象的属性绑定的`ECMA脚本元素`，例如`程序`以及`With`表达式，用js表达可能像下面这样：
{% highlight javascript %}
    function ObjectEnvironmentRecord(obj) {
        this.bindingObj = obj;
    }
{% endhighlight %}

例如，下面这段代码：
{% highlight javascript %}
    var obj = {
        a : 1,
        b : 2
    };
    with (obj) {
        console.debug(a++, b++);
    }
{% endhighlight %}

上面说到，在执行with语句块时，会产生一个词法环境，而这个词法环境包含一个环境记录项，这时候这个环境记录项是一个对象式环境记录项，这个对象是环境记录项绑定了obj这个对象，就像下面这样:
{% highlight javascript %}
    var envRec = new ObjectEnvironmentRecord();
    envRec.binddingObj = obj; //这里这么写来表明步骤
    var lexEnv = new LexicalEnvironment();
    lexEnv.environmentRecord = envRec;
    lexEnv.outerLexicalEnvironment = currentLexicalEnvironment;
{% endhighlight %}

生成这样一个环境记录项后，with语句块中的a，b两个标识符都会“映射”到词法环境的环境记录项（即当前绑定的obj对象）中查找。

> 注意，我们可以把浏览器宿主环境执行过程看成生成了含有对象式环境记录项的词法环境，它绑定了`window对象`, 即：

{% highlight javascript %}
    with (window) {
        alert('1'); //window.alert
    }
{% endhighlight %}


*环境记录项的抽象方法*
<table>
    <tr><th>方法</th><th>作用</th></tr>
    <tr>
        <td>HasBinding(N)</td>
        <td>判断环境记录项是否包含对某个标识符的绑定。如果包含该绑定则返回 true，反之返回 false。其中字符串 N 是标识符文本。</td>
    </tr>
    <tr>
        <td>CreateMutableBinding(N, D)</td>
        <td>在环境记录项中创建一个新的可变绑定。其中字符串 N 指定绑定名称。如果可选参数 D 的值为true，则该绑定在后续操作中可以被删除。</td>
    </tr>
    <tr>
        <td>SetMutableBinding(N,V, S)</td>
        <td>在环境记录项中设置一个已经存在的绑定的值。其中字符串 N 指定绑定名称。V 用于指定绑定的值，可以是任何 ECMA 脚本语言的类型。S 是一个布尔类型的标记，当 S 为 true 并且该绑定不允许赋值时，则抛出一个 TypeError 异常。S 用于指定是否为严格模式。</td>
    </tr>
    <tr>
        <td>GetBindingValue(N,S)</td>
        <td>返回环境记录项中一个已经存在的绑定的值。其中字符串 N 指定绑定的名称。S 用于指定是否为严格模式。如果 S 的值为 true 并且该绑定不存在或未初始化，则抛出一个 ReferenceError 异常。</td>
    </tr>
    <tr>
        <td>DeleteBinding(N)</td>
        <td>从环境记录项中删除一个绑定。其中字符串 N 指定绑定的名称。如果 N 指定的绑定存在，将其删除并返回 true。如果绑定存在但无法删除则返回false。如果绑定不存在则返回 true。</td>
    </tr>
    <tr>
        <td>ImplicitThisValue()</td>
        <td>当从该环境记录项的绑定中获取一个函数对象并且调用时，该方法返回该函数对象使用的 this 对象的值。</td>
    </tr>
</table>



