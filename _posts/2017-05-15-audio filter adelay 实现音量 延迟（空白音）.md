---
layout: post
title:  audio filter adelay 实现音量 延迟（空白音）

---
    目前遇到一个问题  需要视频中指定位置 混一小段音频 。

    1， 辛得大师兄指导  使用  ffmpeg -i voice1.m4a  -filter_complex adelay="1000|1000"  output.m4a    得以把 voice1.m4a  前面延迟（空白）1000毫秒,中间“|”表示 左右声道都延迟1000毫秒   后得到一个 output.m4a文件 ，它的时间长度 这个文件比voice1.m4a多出了前面空白的的的1000毫秒，

    2， 使用 -filter_complex amix混音 把上面得到的output.m4a 混合到 主视频中  从而实现指定位置混音， 命令如下 ：
ffmpeg -i cuc_ieschool.mp4 -i outputVol1.m4a -filter_complex amix=inputs=2:duration=first:dropout_transition=1000 outputAmix.mp4。
其中“cuc_ieschool.mp4 ”  ，“outputVol1.m4a” 为被混音的文件  ，“inputs=2” 表示混音文件为2个  ，“outputAmix.mp4”  是混音后的文件
