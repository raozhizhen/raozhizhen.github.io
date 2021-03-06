---
date: 2015-12-23 12:00
status: public
title: '百度云推送区分fir上的正式版和APPStore的上线版'
---

	虽然上一篇博客还没写完，但手痒想写这个，先讲一下场景，百度云推送有生产状态和开发状态，但fir和APPStore上的版本都是属于生产状态，进行版本迭代的时候，想测试正式版本的全局推送，则也会推送给APPStored的上线版，这肯定是不希望出现的。

###解决方案就是注册两个百度云推送应用了。

首先服务器是有两个，正式服务器对应正式百度云推送API Key，测试服务器对应测试百度云推送API Key，deploy_status都设置为生产状态

客户端的话,服务器地址都选择测试服务器地址，API Key都选择测试百度云推送API Key。因为APPStore版本和fir上的版本都属于Release版本，要区分它们只能手动改，#ifdef DEBUG #else 就没有用了。一定要记得，打上线APPStore的包时，要把服务器地址和推送API Key改回来。

因为全局推送是百度云对百度云推送应用里的用户一个一个推的，所以不会影响另一个百度云推送应用的用户，这样进行版本迭代的时候，fir上的正式版和APPStore的上线版就互不影响。如果要测试直接运行在手机上的测试版，让服务端将测试服务器的deploy_status设置为测试状态就可以啦。如果要测试正式服务器的推送，你就问测试人员：“你为什么要在正式服务器上做测试呢？”，当然，任性的测试人员是不会理你的，好吧，你可以让服务端改一下正式服务器的百度云推送API Key，客户端再将服务器地址改成正式服务器地址，这样就可以使用fir上的正式版测试正式服务器的推送。

以上，和小伙伴[徐亚非](http://www.xuyafei.cn)讨论了一下这个问题，算是纪录一下。