---
layout: post
title: iOS、OSX开发中的两个“坑”
category:   blog
description:   iOS、OSX开发虽然很舒服，但仍有不少坑要注意，否则一不留神就。。。 
---

iOS、OSX开发起从开发工具到API使用都是异常的舒服，是我最推崇的开发环境与模式，不过这中间还是有
不少坑需要注意，否则一不留神就掉进去了。说这些坑对于一些经验丰富的开发者来说可能并不算什么，
甚至连“坑”都不是，不过对一些初学者，或是刚刚进入iOS、OSX开发领域的开发者来说还是很重要的。

###NSURLConnection的回调
首先来说一下NSURLConnection，这个类应该是开发者用的比较多的一个类了，经常用来处理调用运程服务获取数据，一般的使用方法
是

    - (id)initWithRequest:(NSURLRequest *)request delegate:(id < NSURLConnectionDelegate >)delegate startImmediately:(BOOL)startImmediately

在创建NSURLConnection实例的时候传入一个delegate，当所有的数据返回后会调用delegate中的方法：

    - (void)connectionDidFinishLoading:(NSURLConnection *)connection

但是这个类很多时候可能在多线程环境中使用，这时候可能就会和你想像中的行为不太一样，当你设置好
了delegate写好了connectionDidFinishLoading方法后运行程序，理想的情况下是程序请求完远程服务拿
到数据后，调用你的connectionDidFinishLoading方法完成一些伟大的事情，但事实上你会发现你的connectionDidFinishLoading
总是不能被调到，于是你仔细重头检查了一遍代码， 发现并没有什么问题，挫败感油然而生。事实上这是
一个非常隐蔽的坑，事情的原因是这样的，当你创建了一个线程并执行后，你的selector请求远程服务获取
数据，拿到数据后回调方法，但事实上，在你的代码回调的时候线程已经退出了，所以无论如何你也调不到delegate中的方法。
但线程退出和我回调一个方法有什么关系呢？我猜测NSURLConnection在回调的时候应该是采用的线程通讯的方式，即：

    - (void)performSelector:(SEL)aSelector onThread:(NSThread *)thread ...

解决的办法有两个，第一个办法是初始化的时候并不立即发起请求，即startImmediately参数设为NO，并设置connection
的runloop mode:
    
    [connection scheduleInRunLoop: [NSRunloop currentRunLoop] forMode: NSRunLoopCommonModes]

另一种是使用Core Foundation的CFRunLoop:

    NSURLConnection *connection = [[NSURLConnection alloc] initWithRequest:request delegate:self startImmediately:YES];
    CFRunLoopRun()

使用后一种方法的时候另忘记在didFinisheLoading和didFailure的时候停掉RunLoop: 

    CFRunLoopStop(CFRunLoopGetCurrent())

更多RunLoop信息参见Threading Programming Guide的[RunLoop][Threading-Programming-Guide]章节

###你真的了解Block么？
对自己使用Block很有信心？来，先做个测试:
[Block使用测试][block-test]

Block做为iOS4引进的新功能被大量的使用在了UIKit、第三方框架的API中，但其实Block中的坑很多，
其中最麻烦的就数Block的内存管理了，当你使用类似下面代码的时候，就意味着你有内存泄漏的问题了：

    httpService = [[HttpService sharedInstance] sendRequest: complete: ^(Request *request) {
      [self stopAnimating];
    }, failure: ^(Request *request) {
      [self errorReport];
    }];
来自Apple的官方文档说明：

>* If you access an instance variable by reference, a strong reference is made to self;
 * If you access an instance variable by value, a strong reference is made to the variable.

上面的代码在complete块里使用的self，这时会对self进行retain（non-arc，arc中strong），而httpService持有
这个block，self又持有httpService，这就造成的循环引用，导致内存不能回收，最后内存泄漏。
要避免这个问题，就要使用弱引用：

    id typeof(self) weakSelf = self;
    
    httpService = [[HttpService sharedInstance] sendRequest: complete: ^(Request *request) {
      [weakSelf stopAnimation];
    }...

其实上面的代码还不够“强壮”，但足够用了。如果你使用ARC就不存在这些问题了，所以Apple一直是建议开发者
使用ARC。

暂时只想到了这两个，以后想起来在补充吧。

[block-test]: http://blog.parse.com/2013/02/05/objective-c-blocks-quiz/
[Threading-Programming-Guide]: https://developer.apple.com/library/ios/documentation/cocoa/Conceptual/Multithreading/RunLoopManagement/RunLoopManagement.html
