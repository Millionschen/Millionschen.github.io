---
layout: post
title: 从uri获取stream
category: 安卓
tags: Essay
---

android中ContentProvider经常可以提供一些资源的uri,那么如何从uri获取相应的stream并且读取内容呢?
涉及的类有AssetFileDescriptor,Uri,FileDescriptor.
过程如下:

    ```
    AssetFileDescriptor afd = null;

    afd = getActivity().getContentResolver().
        openAssetFileDescriptor(xxxUri, "r");

    FileDescriptor fileDescriptor = afd.getFileDescriptor();
    ```

其中FileDescriptor是对unix文件描述符的包装,由这个文件描述符就可以去构造输入输出的stream了.
