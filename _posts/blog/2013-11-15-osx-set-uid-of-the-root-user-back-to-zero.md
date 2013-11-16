---
layout: post
title: 重置root用户的UID为0
category: blog
description: 重置root用户的UID为0（Reset the UID of root back to zero）
tag: root uid 0
---
    
当你的OS X的root用户的UID不知道为什么变成了非0（至于为什么是0自行google），这时候你就陷入了一个死循环，要想修改root用户的UID，必须有root权限，但要有root权限root用户的UID一定是要0，如果你幸运，之前配置好了sudo那就好办了，可以:

	sudo -u '#0' dscl . change '/Users/root' 'UniqueID' '217' '0'
	
如果之前没有配置好sudo，那没办法只能重装了。