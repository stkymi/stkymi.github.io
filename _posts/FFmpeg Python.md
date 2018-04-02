---
layout: post
title: "FFmpeg Python" 
date:   2017-03-10 10:35:06
categories: 
---

<!-- more -->

### 第一篇: FFmpeg
剪辑
```
ffmpeg -ss 00:00:00 -t 00:00:30 -i i.mp4 -vcodec copy -acodec copy o1.mp4
ffmpeg -ss 00:00:30 -t 00:00:30 -i i.mp4 -vcodec copy -acodec copy o2.mp4
```
在list.txt文件中，对要合并的视频片段进行描述
```
file ./o1.mp4
file ./o2.mp4
```
合并
```
ffmpeg -f concat -i list.txt -c copy output.mp4
```
音量延迟
```
ffmpeg -i i.m4a  -filter_complex adelay="1000|1000"  o.m4a  #左右声道均延迟1000毫秒
```
