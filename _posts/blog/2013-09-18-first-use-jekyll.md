---
layout: post
title:  初次使用github写博客
category:   blog
description:    第一次使用github+Jekyll写博客，解决一些遇到的问题
---

###关于写博客
我是一个没有长性的人，写博客这事儿也是今天心血来潮想起来连着写两天，明天忘记了就停一年,
所以， 其实一共也没写过几篇，更不用说有质量的了，这次也一样，没准哪天就又停了。

###关于用github写博客
之前一直不愿意持续写的原因（你看，多会找借口）其中一个就是以前维护的步骤太繁琐，
，每次要写个东西都要:
>登录->新建文章->使用难用的文本编辑器->预览->修改->预览...->保存

太麻烦，还有各种排版，富文本编辑器，想想就烦，使用github
 （不知道github是什么？如果你是程序员，建议改行，如果不是
可以猛戳[这里][here])就简单多了，只消准备好vim、git就好了(虽然也要先配置一下，但
  这个配置只是一次性）, Vim+markdown+git来实现编写+管理，整体页面风格简约明了，
  使用vim本地编写，然后通过git来维护，想到这，我又把过期的域名续费了。。。

###如何使用Github写博客
搭建配置的过程还是很简单的，具体可以参考[Beiyuu][beiyuu]的这篇:[使用Github Page
建立独立博客][setup]，里面介绍很详细，这里说一下我遇到的问题：全部配置好后
新建了一个2013-09-18-first-use-jekyll.md，启动jekyll服务，发现什么也没有，没任何
变化，在vim中发现原来fork的项目中所有md文件都是:

    md*

估计是权限的问题，执行:

    chmod 755 2013-09-18-first-use-jekyll.md
    
就正常了。

最后，如果你像我一样懒可以fork[这个项目][fork]

[here]: http://github.com
[beiyuu]: http://beiyuu.com
[setup]: http://beiyuu.com/github-pages
[fork]: https://github.com/beiyuu/beiyuu.github.com
