---
layout: post
title: iOS实现图文混排的两个方法
category: blog
description: iOS中实现图文混排的两个方法，可以用来实现类似于杂志应用类的效果
tag: ios coretext1
---

如果你想自定义文本的布局，例如像QQ、微信这样的应用中使用表情，那你多半会用到CoreText，CoreText是iOS、OSX平台的文本处理低层的框架，
可以实现任意的文字编排，更多详细信息请戳[官方文档][CoreText-official-guide]，一般来说， 我们们用下面的代码来实现图文混排：

    text = [[NSMutableAttributedString alloc] initWithString:@""];
    NSAttributedString *txt1 = [[NSAttributedString alloc] initWithString:@"测试"];
    [text appendAttributedString:txt1];
    
    [txt1 release];
    
    CTRunDelegateCallbacks callback;
    callback.version = kCTRunDelegateVersion1; //必须指定，否则不会生效，没有回调产生。
    callback.dealloc = deallocCallback;
    callback.getAscent = getAscent;
    callback.getDescent = getDescent;
    callback.getWidth = getWidth;
    NSDictionary *imgAttr = [[NSDictionary dictionaryWithObjectsAndKeys:@100, @"width", nil] retain];
    
    CTRunDelegateRef delegate = CTRunDelegateCreate(&callback, imgAttr);
    NSDictionary *txtDelegate = [NSDictionary dictionaryWithObjectsAndKeys:(id)delegate, (NSString*)kCTRunDelegateAttributeName, @100, @"width", nil];
    NSAttributedString *imgField = [[[NSAttributedString alloc] initWithString:@" " attributes:txtDelegate] autorelease];
    [text appendAttributedString:imgField];
    
    [text appendAttributedString:[[[NSAttributedString alloc] initWithString: @"结束"] autorelease]];
    
    CGMutablePathRef pathRef = CGPathCreateMutable();
    CGPathAddRect(pathRef, NULL, CGRectMake(0, 0, self.frame.size.width, self.frame.size.height));
    
    framesetter = CTFramesetterCreateWithAttributedString((CFAttributedStringRef)text);
    
    ctFrame = CTFramesetterCreateFrame(framesetter, CFRangeMake(0, 0), pathRef, NULL);
    
    CFArrayRef lines = CTFrameGetLines(ctFrame);
    
    CGPoint origins[CFArrayGetCount(lines)];
    
    CTFrameGetLineOrigins(ctFrame, CFRangeMake(0, 0), origins);
    
    for (int i = 0; i < CFArrayGetCount(lines); i++) {
        
        CTLineRef line = CFArrayGetValueAtIndex(lines, i);
        
        CFArrayRef runs = CTLineGetGlyphRuns(line);
        
        for (int j = 0; j < CFArrayGetCount(runs); j++) {
            
            CTRunRef run = CFArrayGetValueAtIndex(runs, j);
            
            CGPoint lineOrigin = origins[i];
            
            NSDictionary *meta = (NSDictionary*)CTRunGetAttributes(run);
            
            if (meta && ([meta valueForKey:@"width"] != nil)) {
                imageLocation.y = lineOrigin.y;
                
                CGFloat offset = CTLineGetOffsetForStringIndex(line, CTRunGetStringRange(run).location, NULL);
                imageLocation.x = lineOrigin.x + offset + self.frame.origin.x;
            }
        }
        
    }
    
    CFRelease(pathRef);
    [self setNeedsDisplay];

一直以来，我认为只有这种方法实现。好吧，其实我没有想过有没有其它实现方法的问题。直到有一天看类似效果的代码时惊奇的发现：怎么
没有CTRunDelegate? 于是就仔细想了一下这个问题，创建CTFrame的时候会指定一个path，通常这个path我会使用一个CGRect完事，然后在
有图片的地方使用CTRunDelegate处理一下，但其实完全可以使用CGMutablePath来画出一块不规则的文本路径，比如：
    ![image](/images/post_img/core-text-example.png)
这样，就可以在预定的位置画图片了，而不用会用CTRunDelegate来特殊处理，这种方式比较适合图片位置固定的应用。

[CoreText-official-guide]: http://developer.apple.com/library/ios/documentation/StringsTextFonts/Conceptual/CoreText_Programming/
