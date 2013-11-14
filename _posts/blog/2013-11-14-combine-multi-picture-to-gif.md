---
layout: post
title: 图片合成GIF
category: blog
description: iOS、OSX中实现多张图片合成一张GIF的代码。
tag: ios osx gif
---

看到有人发gif图片，忽然间想怎么把几张图片合成为一张gif，想了想，写了段代码实现：

    
    NSImage *img1 = [NSImage imageNamed:@"Textmate"];
    NSImage *img2 = [NSImage imageNamed:@"potraint"];
    
    NSString *path = [[NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) lastObject] stringByAppendingPathComponent:@"animated.gif"];

    NSLog(@"%@", path);
    
    NSURL *destUrl = [NSURL fileURLWithPath:path];
    
    CGImageDestinationRef imageDest;
    NSString *uti = @"com.compuserve.gif";
    
    NSData *img1Data = [img1 TIFFRepresentation];
    NSData *img2Data = [img2 TIFFRepresentation];
    
    
    NSDictionary *frameProperties = [NSDictionary dictionaryWithObject:[NSDictionary dictionaryWithObject:[NSNumber numberWithInt:2] forKey:(NSString *)kCGImagePropertyGIFDelayTime]
                                                                forKey:(NSString *)kCGImagePropertyGIFDictionary];
    NSDictionary *gifProperties = [NSDictionary dictionaryWithObject:[NSDictionary dictionaryWithObject:[NSNumber numberWithInt:0] forKey:(NSString *)kCGImagePropertyGIFLoopCount]
                                                              forKey:(NSString *)kCGImagePropertyGIFDictionary];

    
    CGImageSourceRef imageSourceRef1 = CGImageSourceCreateWithData((CFDataRef)img1Data, (CFDictionaryRef)frameProperties);
    CGImageSourceRef imageSourceRef2 = CGImageSourceCreateWithData((CFDataRef)img2Data, (CFDictionaryRef)frameProperties);
    CGImageRef imgRef1 = CGImageSourceCreateImageAtIndex(imageSourceRef1, 0, NULL);
    CGImageRef imgRef2 = CGImageSourceCreateImageAtIndex(imageSourceRef2, 0, NULL);
    
    
    imageDest = CGImageDestinationCreateWithURL((CFURLRef)destUrl, (CFStringRef)uti, 2, NULL);
    CGImageDestinationAddImage(imageDest, imgRef1, (CFDictionaryRef)frameProperties);
    CGImageDestinationAddImage(imageDest, imgRef2, (CFDictionaryRef)frameProperties);
    CGImageDestinationFinalize(imageDest);

上面实现了控制播放的速度，使用CoreAnimation来添加水印等功能可以查看API，还是比较简单的。
    
