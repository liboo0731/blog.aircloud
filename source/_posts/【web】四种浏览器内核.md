---
title: 【web】四种浏览器内核
date: 2018-08-26 21:49:23
tags:
- web
---

“浏览器内核”主要指渲染引擎(Rendering Engine)，负责解析网页语法(如HTML、JavaScript)并渲染、展示网页。

因此，所谓的浏览器内核通常也就是指浏览器所采用的渲染引擎，渲染引擎决定了浏览器如何显示网页的内容以及页面的格式信息。

不同的浏览器内核对网页编写语法的解析也有所不同，因此同一网页在不同的内核浏览器里的渲染、展示效果也可能不同。

浏览器内核种类繁多，商用的加上非商业的免费内核，大约会超过10款，我们今天重点看一下目前主流的四大浏览器内核Trident、Gecko、WebKit以及Presto。

了解网页浏览器四种主要内核：

#### 一、Trident内核（代表：Internet Explorer）

说起Trident，很多人都会感到陌生，但提起IE(Internet Explorer)则无人不知无人不晓，由于其被包含在全世界使用率最高的操作系统Windows中，得到了极高的市场占有率，所以我们又经常称其为IE内核。

Trident(又称为MSHTML)，是微软开发的一种排版引擎。它在1997年10月与IE4一起诞生，一直在被不断地更新和完善。

而且除IE外，许多产品都在使用Trident核心，比如Windows的Help程序、RealPlayer、Windows Media Player、Windows Live Messenger、Outlook Express等等都使用了Trident技术。

Trident实际上是一款开放的内核，Trident引擎被设计成一个软件模块，使得其他软件开发人员很容易将网页浏览功能加到他们自行开发的应用程序里，

其接口内核设计相当成熟，因此涌现出许多采用IE内核而非IE的浏览器，但是Trident只能用于Windows平台。

使用Trient渲染引擎的浏览器包括：IE、傲游、世界之窗浏览器、Avant、腾讯TT、Sleipnir、GOSURF、GreenBrowser和KKman等。

#### 二、Gecko内核（代表：Mozilla Firefox）

Gecko是开放源代码、以C++编写的网页排版引擎，目前被Mozilla家族网页浏览器以及Netscape 6以后版本浏览器所使用。

这款软件原本是由网景通讯公司开发的，现在则由Mozilla基金会维护。由于Gecko的特点是代码完全公开，因此，其可开发程度很高，全世界的程序员都可以为其编写代码，增加功能。

因为这是个开源内核，因此受到许多人的青睐，采用Gecko内核的浏览器也很多，这也是Gecko内核虽然年轻但市场占有率能够迅速提高的重要原因。

Gecko排版引擎提供了一个丰富的程序界面以供与互联网相关的应用程序使用，例如网页浏览器、HTML编辑器、客户端/服务器等。

虽然最初的主要对象是Mozilla的衍生产品，如Netscape和Mozilla Firefox，但是现在已有很多其他软件利用这个排版引擎。此外Gecko也是一个跨平台内核，可以在Windows、BSD、Linux和Mac OS X中使用。

正在和曾经使用Gecko引擎的浏览器有Firefox、网景6～9、SeaMonkey、Camino、Mozilla、Flock、Galeon、K-Meleon、Minimo、Sleipni、Songbird、XeroBank。Google Gadget引擎采用的就是Gecko浏览器引擎。

#### 三、WebKit内核（代表：Safari、Chrome）

WebKit 是一个开放源代码的浏览器引擎(Web Browser Engine)，WebKit最初的代码来自KDE的KHTML和KJS(它们均为开放源代码，都是自由软件，在GPL协议下授权)。

所以WebKit也是自由软件，同时开放源代码。它的特点在于源码结构清晰、渲染速度极快。主要代表产品有Safari和Google的浏览器Chrome。

WebKit内核在手机上的应用也十分广泛，例如Google的Android平台浏览器、Apple的iPhone浏览器、Nokia S60浏览器等所使用的浏览器内核引擎，都是基于WebKit引擎的。

WebKit内核也广泛应用于Widget引擎产品，包括中国移动的BAE、Apple的Dashboard以及Nokia WRT在内采用的均为WebKit引擎。

#### 四、Presto内核（代表：Opera）

Presto是由Opera Software开发的浏览器排版引擎，供Opera 7.0及以上使用。

它取代了旧版Opera 4至6版本使用的Elektra排版引擎，包括加入动态功能，例如网页或其部分可随着DOM及Script语法的事件而重新排版。

Presto的特点就是渲染速度的优化达到了极致，它是目前公认的网页浏览速度最快的浏览器内核，然而代价是牺牲了网页的兼容性。

Presto实际上是一个动态内核，与Trident、Gecko等内核的最大区别就在于脚本处理上，Presto有着天生的优势，页面的全部或者部分都能够在回应脚本事件时等情况下被重新解析。

此外该内核在执行JavaScript时有着最快的速度，根据同等条件下的测试，Presto内核执行同等JavaScript所需的时间仅有Trident和Gecko内核的约1/3。

Presto是商业引擎，了Opera以外较少浏览器使用Presto内核，这在一定程度上限制了Presto的发展。

