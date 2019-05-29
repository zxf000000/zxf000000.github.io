---
layout: post
title: cocoapods制作私有库bug记录
tags:
  - Cocoapods
---


# cocoapods制作私有库bug记录

## 未找到appIcon

私有库中使用**.xcassets**管理图片资源的时候，如果直接在**.podspec** 文件中配置xcassets的路径，如   

```
s.resource = 'YCLLiveModule/YCLLiveModule/**/Live.xcassets'
```

在lint 的时候，会提示找不到appIcon，进而导致失败   
在github 上面，有一系列的解决方案，我采用了其中一种，有效  

> 解决方案：   
> 创建bundle文件，将.xcassets 直接拖入，然后在 .podspec配置bundle的路径