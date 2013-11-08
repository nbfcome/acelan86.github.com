---
layout: post
title: "Learning from XAuth: Cross domain localStorage"
description: ""
category: ""
tags: []
---
{% include JB/setup %}

I typically don’t get too excited when new open source JavaScript utilities are released. It may be the cynic in me, but generally I feel like there’s very little new under the sun that’s actually useful. Most of these utilities are knockoffs of other ones or are too large to be practically useful. When I first came across XAuth, though, a little tingly feeling of excitement swept over me. And the first coherent thought I had while looking at the source: this is absolutely brilliant.

## What is XAuth?

I don’t want to spend too much time explaining exactly what XAuth is, since you can read the documentation yourself to find the nitty gritty details. In short, XAuth is a way to share third-party authentication information in the browser. Instead of every application needing to go through the authorization process for a service, XAuth is used to store this information in your browser and make it available to web developers. That means a site that can serve you a more relevant experience when you’re signed into Yahoo! doesn’t need to make any extra requests to determine if you’re signed in. You can read more about XAuth over on the Meebo blog.

## The cool part

This post is really less about the usage of XAuth and more about the implementation. What the smart folks at Meebo did is essentially create a data server in the browser. The way that they did this is by combining the power of cross-document messaging and localStorage. Since localStorage is tied to a single origin, you can’t get direct access to data that was stored by a different domain. This makes the sharing of data across domains strictly impossible when using just this API (note the difference with cookies: you can specify which subdomains may access the data but not completely different domains).

Since the primary limitation is the same-origin policy of localStorage, circumventing that security issue is the way towards data freedom. The cross-document messaging functionality is designed to allow data sharing between documents from different domains while still being secure. The two-part technique used in XAuth is incredibly simple and consists of:

* `Server Page` – there’s a page that’s hosted at http://xauth.org/server.html that acts as the “server”. It’s only job is to handle requests for localStorage. The page is as small as possible with minified JavaScript, but you can see the full source at GitHub.
* `JavaScript Library` – a single small script file contains the JavaScript API that exposes the functionality. This API needs to be included in your page. When you make a request through the API for the first time, it creates an iframe and points it to the server page. Once loaded, requests for data are passed through the iframe to the server page via cross-document messaging. The full source is also available on GitHub.
Although the goal of XAuth is to provide authentication services, this same basic technique can be applied to any data.