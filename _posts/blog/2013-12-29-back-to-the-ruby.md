---
layout: post
title: 重返Ruby&Rails
category: blog
description: 用tornado开发了一个简单的小应用，但是发现要自制的东西太多了，实在忍不住，用rails重新开发了，这下-爽了。。。
tag: Ruby Python
---

由于有个小应用打算用VPS，所以放弃了之前一直使用、研究Ruby&Rails，转而使用Python&Tornado，Tornado没的说， 一级棒，内存占用、并发能力都很好，对于使用VPS来说，确实是一个优势，不过，Tornado要自制的东西实在是太多，除了提供了基本的http server、web framework和一个torndb(严格来说已经从tornado分离出去了)外，其它多数都要自制，而我开发这个小东西（是小东西）要的就是简单、快速而不想大量的时间花在开发基础的东西上面， 而rails恰好就是为了解决这个问题而出现的。当然，其实最重要的原因，还是我更喜欢Ruby的语法，更准确的说是Ruby的文化。选一门语言，最终其实就是在选语言的文化，社区的文化，这点很重要，不要相信那些所谓的语言不重要之类的话，那些人多数都是没有开发过真正的有用的系统，如果一种语言的使用上不能让你觉得到爽（或者你总感觉不爽），怎么能好好的使用下去，当然，好与不好是见仁见智的问题，一切还是全看你自己！