---
layout: post
title: LeanCloud云引擎命令行工具非本机调试
---
我们使用了LeanCloud的云引擎服务来给客户端提供服务.LeanCloud提供了一个不错的[命令行工具](https://leancloud.cn/docs/leanengine_cli.html)来部署和调试云引擎代码, 但是有个限制, 它的命令行工具仅仅支持中国站点, 而我们的service部署在LeanCloud的美国站点, 所以一直以来都没有使用这个工具.

这里就带来一个很麻烦的问题, 每次调试代码要先把代码部署到云引擎端才能调试,对于初期开发大量代码的话,非常没有效率.

研究了下LeanCloud文档, 觉得可以利用下LeanCloud的这个命令行工具, 可以用它来[调试云引代码](https://leancloud.cn/docs/leanengine_cli.html#本地运行).这样一来就少去了大量因调试而需要重新部署的时间.不过由于这个工具只支持中国站点, 所以需要在中国站点建立一份同样的项目来进行调试, 调试完毕后发布到美国站点.

进行了这个方案后,节约了不少调试代码的时间.但还有一个问题是命令行工具只能进行本机调试,而我的Linux开发机(树莓派)并不是本机.

    启动应用：
    $ lean up
    可能会提示输入应用的MasterKey，粘贴后窗口不会有任何显示，直接回车，即可在本机调试云引擎。
    通过浏览器打开 http://localhost:3000，进入 web 应用的首页。
    通过浏览器打开 http://localhost:3001，进入云引擎云函数和 Hook 函数调试界面。

看了下文档和命令help, 并没有发现可以指定监听的IP. netstat看了下监听端口, 发现server监听本机所有的IP,那不是没问题了? 在其他主机上打开http://IP:3001，进入云引擎云函数和 Hook 函数调试界面,果然可以打开.但是发现执行调用函数却没有任何反应.看了下他的源代码,发现前端将调用路径写死成localhost了,考虑将localhost改成开发机的IP可能就可以工作了.找到avoscloud-code的代码目录,以localhost为关键字搜索,找到一个文件hardcode localhost

    /usr/local/lib/node_modules/avoscloud-code
    ./public/index.html:124:            var url = "http://localhost:" + leanenginePort + apiEndpoint + $('#functions').val();
    ./public/index.html:161:            var url = "http://localhost:" + leanenginePort + "/1.1/functions" + $('#hooks').val();

随即,将localhost改成该机的IP,重新测试,果然就可以从其他机器上远程测试云引擎函数.
