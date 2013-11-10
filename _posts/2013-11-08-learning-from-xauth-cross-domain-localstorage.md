---
layout: post
title: "Learning from XAuth: Cross domain localStorage"
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

这篇文章写了很少关于XAuth的使用而更多的是关于它的实现。Meebo的聪明的做法本质上是再浏览器中创建了一个数据服务。
This post is really less about the usage of XAuth and more about the implementation. What the smart folks at Meebo did is essentially create a data server in the browser. The way that they did this is by combining the power of cross-document messaging and localStorage. Since localStorage is tied to a single origin, you can’t get direct access to data that was stored by a different domain. This makes the sharing of data across domains strictly impossible when using just this API (note the difference with cookies: you can specify which subdomains may access the data but not completely different domains).

Since the primary limitation is the same-origin policy of localStorage, circumventing that security issue is the way towards data freedom. The cross-document messaging functionality is designed to allow data sharing between documents from different domains while still being secure. The two-part technique used in XAuth is incredibly simple and consists of:

* `Server Page` – there’s a page that’s hosted at http://xauth.org/server.html that acts as the “server”. It’s only job is to handle requests for localStorage. The page is as small as possible with minified JavaScript, but you can see the full source at GitHub.
* `JavaScript Library` – a single small script file contains the JavaScript API that exposes the functionality. This API needs to be included in your page. When you make a request through the API for the first time, it creates an iframe and points it to the server page. Once loaded, requests for data are passed through the iframe to the server page via cross-document messaging. The full source is also available on GitHub.
Although the goal of XAuth is to provide authentication services, this same basic technique can be applied to any data.

example:

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