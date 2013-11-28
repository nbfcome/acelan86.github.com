---
layout: post
title: "让innerHTML的脚本也能正常执行"
description: ""
category: ""
tags: []
---
{% include JB/setup %}


# 问题

有时候，我们想要这么做

{% highlight javascript %}
    var content = [
        '<div>xxx</div>',
        '<script>alert("aaa");</script>'
    ].join('');

    dom.innerHTML = content;
{% endhighlight %}

但是，事与愿违，innerHTML里面的脚本文件通常是不会执行的


# 先抛出结论

1. ie6下，innerHTML中的具有defer属性的内联或者外部脚本都会执行
2. ie下，如果第一个节点为script节点，那么会被抛弃，不下载也不执行，就跟不存在一样
3. 其他的浏览器均无法执行innerHTML中的脚本
4. 所有浏览器中，虽然script不被执行，但除了2的情况外，所有script节点都存在


# 实现原理

因为结论4，我们可以先把content的内容innerHTML到dom中，这时候所有的script都存在，除了2的情况
为了保证2的情况也可以，我们可以这么做

{% highlight javascript %}
    function fill(dom, content) {
        dom.innerHTML = '<i>hack ie content firstchild is script node</i>' + content;
        ...
    }
{% endhighlight %}

然后获取所有的script节点

{% highlight javascript %}
    function fill(dom, content) {
        dom.innerHTML = '<i>hack ie content firstchild is script node</i>' + content;

        var scripts = document.getElementsByTagName('script');

        ...
    }
{% endhighlight %}

接下来，只要将所有的script按照类型进行解析

{% highlight javascript %}
    function fill(dom, content) {
        dom.innerHTML = '<i>hack ie content firstchild is script node</i>' + content;

        var scripts = document.getElementsByTagName('script'),
            script;

        for (var i = 0, len = scripts.length; i < len; i++) {
            script = scripts[0];
            if (script.src) {
                //外部脚本
                loadScript(script.src);
            } else {
                //内部脚本
                evalGlobalScript(script.text || script.textContent || script.innerHTML || "");
            }

            //去除执行过后的script
            if (script.parentNode ) {
                script.parentNode.removeChild(script);
            }
        }
    }
{% endhighlight %}

这里的evalGlobalScript方法如下，因为eval的代码在严格模式下的作用域跟非严格模式下有区别，因此需要用以下的方法来确保是在global作用域执行

{% highlight javascript %}
    function evalGlobalScript(data) {
        if (data && /\S/.test(data)) {
            (window.execScript || function (data) {
                window["eval"].call(window, data);
            })(data);
        }
    }
{% endhighlight %}

最后，请记住结论1， ie6下defer的问题，如果不处理，所有的defer代码会被执行2遍，因此，上面的代码变成

{% highlight javascript %}
    function fill(dom, content) {
        dom.innerHTML = '<i>hack ie content firstchild is script node</i>' + content;

        var scripts = document.getElementsByTagName('script'),
            script;

        for (var i = 0, len = scripts.length; i < len; i++) {
            script = scripts[0];
            //屏蔽结论1
            if (ie <= 6 && script.defer) {
                continue;
            }
            if (script.src) {
                //外部脚本
                loadScript(script.src);
            } else {
                //内部脚本
                evalGlobalScript(script.text || script.textContent || script.innerHTML || "");
            }

            //去除执行过后的script
            if (script.parentNode ) {
                script.parentNode.removeChild(script);
            }
        }
    }
{% endhighlight %}


# 需要注意的问题

1. 上面只是简单实现，执行顺序无法保障
2. 脚本中的`document.write`无法处理，除非解析脚本，或者重载全局`document.write`



sinaadToolkit.dom.fill