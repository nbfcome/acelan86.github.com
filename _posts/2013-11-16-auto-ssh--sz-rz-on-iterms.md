---
layout: post
title: "在iterms中配置支持rz, sz的自动登录ssh脚本"
description: ""
category: ""
tags: []
---
{% include JB/setup %}

折腾了一天，终于搞定抛弃SecureCRT，在mac iterm(mac终端)实现自动登录ssh（可以保存密码）并支持sz，rz。

# MacPort
安装macport，它能让我们省去很多劳动，如果已经有了，可以忽略这一步

example:

    访问http://www.macports.org/install.php，下载对应的pkg安装即可

# zssh
普通的mac下ssh不支持sz，rz，因此需要用zssh这个工具:


example:

    sudo port install zssh

# 自动登陆ssh

正常ssh登陆需要使用下面的命令:

example:

    ssh username@ip
    password: 密码
    Enter Password: 动态密码


我们想要能够自动保存第一步的静态密码，只输入动态密码，这时候，需要expect登场

1.安装expect:

example:

    sudo port install expect


2. 将下面的内容保存成一个文件，例如~/bin/ssh.sh:

example:

    #!/usr/bin/expect
    spawn zssh username@ip
    expect -exact "password:"
    send -- "密码填这里\r"
    interact

简单解释下，expect是一个自动化工具，上面的脚本意思大概是:

* `#!/usr/bin/expect` -- 指定脚本类型expect
* `spawn zssh username@ip` -- 类似使用ssh命令，只是在外部用spawn加了个壳，用于接收和执行自动输入
* `expect -exact "password:"` -- 精确判断执行后输出内容是不是"password:"字符串
* `send` -- "密码填这里\r" -- 如果上面成立，则发送静态密码，相当于在终端输入静态密码，如果不成立，会等待一个时间，可以通过在spawn之前set TIMEOUT 秒数来指定时间，注意后面还有个\r, 就是输入回车
* `interact` -- 把控制权交给用户，用户此时可以进行键盘输入




3. 登陆的时候只要:

example:

    expect ～/bin/ssh.sh


# sz和rz

安装lrzsz:

example:

    sudo port install lrzsz


大家都知道，sz在ssh中用于从远端服务器下载文件:

example:

    sz -bey 远端文件

rz在ssh中用于將本地文件上传到远端:

example:

    rz
    弹出窗口选择文件
    确定后上传

但是。。。在mac终端中，他们没法正常实现，所以我们会换成上面的zssh


它的使用步骤稍微有点不同，sz:

example:

    sz 远端文件
    按下CTRL + 2， 进入本地目录状态，执行rz接收文件
    等待接收完毕，远端文件存放本地登录ssh的目录中


rz:

example:

    按下CTRL + 2, 进入本地目录状态
    执行sz 本地文件，回车
    等待上传本地文件结束




