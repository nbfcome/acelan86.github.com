---
layout: post
title: "XAuth: 跨域本地存储"
description: ""
category: ""
tags: []
---
{% include JB/setup %}

I typically don’t get too excited when new open source JavaScript utilities are released. It may be the cynic in me, but generally I feel like there’s very little new under the sun that’s actually useful. Most of these utilities are knockoffs of other ones or are too large to be practically useful. When I first came across XAuth, though, a little tingly feeling of excitement swept over me. And the first coherent thought I had while looking at the source: this is absolutely brilliant.

当一个新的开源Javascript工具发布的时候，我通常不会太激动。这可能是因为我比较愤世嫉俗，但总得来说我。这些工具中有很大一部分。当我偶然看到XAuth，一丝小激动划过我的心里。当细读源码:这是绝对出色的。

## 什么是XAuth？

我不想花大量时间去解释什么是XAuth，而应该是你自己读它的文档去找到具体细节。简而言之，XAuth是一种在浏览器之间分享第三方认证信息的方式。代替每个应用需要通过一个服务处理认证信息，XAuth用于在你的浏览器中存储这些信息并且使它们能被web开发者访问。这意味着一个能够对你提供相关体验服务的站点当你登录Yahoo！之后不需要做任何额外的请求去判断你是否已经登录。你可以从Meebo的博客中了解到更多关于XAuth的信息。

## 很酷的部分

这篇文章写了很少关于XAuth的使用而更多的是关于它的实现。Meebo哪些聪明的人所做的本质上是在浏览器中创建了一个数据服务。他们通过结合corss-document messaging和localStorage的功能来做到这些。由于localStorage是同源的，因此你不能直接对存储在不同域下的数据进行访问。这使得仅靠使用localStorage的这些API不可能进行跨域间的数据分享（注意cookies不一样：你能够指定子域可以被访问但完全不同的域不行）。

localStorage的主要限制是同源策略，绕过安全问题是为了数据自由的方法。cross-document messaging功能就是为了在保证安全的情况下允许不同域间不同文档共享数据而设计的。这两部分技术被难以置信的简单的应用于XAuth，包括以下部分：


* **Server Page** – 有一个在主机http://xauth.org/server.html下的页面，它扮演“server”的角色。它唯一的任务是管理localStorage的请求。这个页面的javascript被压缩成最小，但你可以在GitHub中看到全部的源代码。
* **JavaScript Library** – 一个小的脚本文件包含暴露的Javascript API。这个API需要被包含到你的页面中，当你通过第一次通过API创建一个请求时，它会通过cross-document messaging创建一个iframe并且把它指向server page。一旦iframe被加载，对数据的请求使用cross-domain messaging，通过iframe被传递到server page。完整的源代码也放在GitHub中。

XAuth的目的是提供服务认证，因此相同的技术也可以被延伸到任何数据中。


## 主要技术

假设你的页面在www.example.com上，并且你想要获取存储在foo.example.com的localStorage上的某些信息。第一步是创建一个iframen指向foo.example.com上的一个页面，它扮演data server的角色。这个页面的任务是管理对数据的请求并且传回数据。以下是一个简单的例子:

    <!doctype html>
    <!-- Copyright 2010 Nicholas C. Zakas. All rights reserved. BSD Licensed. -->
    <html>
    <body>
    <script type="text/javascript">
    (function(){

        //allowed domains
        var whitelist = ["foo.example.com", "www.example.com"];

        function verifyOrigin(origin){
            var domain = origin.replace(/^https?:\/\/|:\d{1,4}$/g, "").toLowerCase(),
                i = 0,
                len = whitelist.length;

            while(i < len){
                if (whitelist[i] == domain){
                    return true;
                }
                i++;
            }

            return false;
        }

        function handleRequest(event){
            if (verifyOrigin(event.origin)){
                var data = JSON.parse(event.data),
                    value = localStorage.getItem(data.key);
                event.source.postMessage(JSON.stringify({id: data.id, key:data.key, value: value}), event.origin);
            }
        }

        if(window.addEventListener){
            window.addEventListener("message", handleRequest, false);
        } else if (window.attachEvent){
            window.attachEvent("onmessage", handleRequest);
        }
    })();
    </script>
    </body>
    </html>


这是我建议的最小实现。关键函数是`handleRequest()`, 它在window的message事件触发的时候被调用。因为我没有使用任何Javascript库，我需要用手工检查方式来使用合适的方法挂接事件句柄。

在`handleRequest()`内部，第一步是验证请求来源。这是至关重要的一步以确保只有某些人能够创建iframe，并指向这些文件，来获取你的localStorage的信息。事件对象包含一个属性称为origin，指定了协议，域和（可选的）端口号，它是请求的来源（比如，“http://www.example.com”）；这个属性不包含任何path和请求参数信息。`verifyOraigin()`函数简单的检查域白名单确保origin属性在白名单中。它通过使用正则表达式去除协议和端口然后转换成小写字符来和白名单数组中的domain进行匹配。

如果origin是可靠的，那么event.data属性被作为JSON对象传递并且key属性被作为key从localStorage中读取数据。然后一个信息被作为包含唯一ID的JSON对象传回

If the origin is verified then the event.data property is parsed as a JSON object and the key property is used as the key to read from localStorage. A message is then sent back as a JSON object that contains the unique ID that was passed initially, the key name, and the value; this is done using postMessage() on event.source, which is a proxy for the window object that sent the request. The first argument is the JSON-serialized message containing the value from localStorage and the second is the origin to which the message should be delivered. Even though the second argument is optional, it’s good practice to include the destination origin as an extra measure of defense against cross-site scripting (XSS) attacks. In this case, the original origin is passed.

For the page that wants to read data from the iframe, you need to create the iframe server and handle message passing. The following constructor creates an object to manage this process:
