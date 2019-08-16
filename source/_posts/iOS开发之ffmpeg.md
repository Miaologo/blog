---
title: iOS开发之ffmpeg
date: 2019-08-14 21:03:45
tags:
---



### 在mac平台安装ffmpeg

安装分为源码安装和命令行安装，关于这部分的教程网上非常多，所以我只介绍一种简单的安装->命令行安装。

我们使用`brew`来安装ffmpeg和ffplay命令。

关于`brew`以及其用法可参考 [Homebrew的安装及使用](https://link.juejin.im/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2F4e80b42823d5)

空白安装：

如果你的电脑上以前从来没有安装过`ffmpeg`，那么你可以直接使用以下命令直接安装。

```
brew install ffmpeg --with-ffplay
复制代码
```

安装成功后可以使用`ffplay --help`来检测是否成功安装。

非空白安装：

如果你的电脑上以前安装了`ffmpeg`，那么你需要把以前安装的`ffmpeg`卸载干净然后再利用上面的命令安装。

卸载方法：

```
brew uninstall ffmpeg
复制代码
```

你也可以直接在ffmpeg官网下载ffmpeg,ffplay等命令行工具，直接拷贝到你的bin目录下，直接运行也可以。 此部分主要目的就是能在mac上利用利用ffmpeg命令行工具来解析音视频文件。

### 简单常用的ffmpeg命令

关于ffmpeg的命令，最好的途径是直接在其官网上查看。网上也有很多的示例，有些比较简单的命令用的多了自然就习惯性的记着了。以下我贴出的是在一本书上看的最常见的命令，具体如下：

#### ffprobe

ffprobe是用来查看媒体文件头信息的工具。

- 查看音频文件头信息

  ```
  ffprobe 黄昏里.mp3
  
  复制代码
  ```

显示结果：

```

```
Input #0, mp3, from '黄昏里.mp3':
  Metadata:
    title           : 黄昏里
    artist          : 邓丽君
    album           : 爱的箴言
    Tagging time    : 2012-08-08T02:48:38
    TYER            : 1998-01-01
  Duration: 00:02:45.75, start: 0.025056, bitrate: 131 kb/s
    Stream #0:0: Audio: mp3, 44100 Hz, stereo, s16p, 128 kb/s
    Metadata:
      encoder         : LAME3.97 
    Stream #0:1: Video: mjpeg, yuvj420p(pc, bt470bg/unknown/unknown), 240x240 [SAR 1:1 DAR 1:1], 90k tbr, 90k tbn, 90k tbc
    Metadata:
      title           : e
      comment         : Cover (front)
```
复制代码
```

- 查看视频文件头信息

  ```
  ffprobe test.mp4
  复制代码
  ```

以上就是查看音频文件和视频文件头信息的方式。下面介绍几个更高级的用法。

- 输出格式信息format_name、时间长度duration、文件大小size、比特率bit_rate、流的数目nb_streams等

  ```
  ffprobe -show_format test.mp4
  复制代码
  ```

- 以json格式输出每一个流的最详细信息

  ```
  ffprobe -print_format json -show_streams test.mp4
  复制代码
  ```

- 显示帧信息

  ```
  ffprobe -show_frames test.mp4
  复制代码
  ```

- 查看包信息

  ```
  ffprobe -show_packets test.mp4
  复制代码
  ```

#### ffplay

ffplay是以ffmpeg框架为基础，外加渲染音视频库libSDL来构建的媒体文件播放器。它所以来的libSDL是1.2版本的。

- 播放音视频文件

  ```
  ffplay test.mp4/黄昏里.mp3
  复制代码
  ```

- 播放一段视频，循环10次

  ```
  ffplay test.mp4 -loop 10
  复制代码
  ```

- ffplay可以指定使用哪一路音频流或视频流播放

  ```
  ffplay test.mkv -ast 1  // 表示播放视频中的第一路音频流，如果参数ast后面跟的是2，那么就播放第二路音频流，如果没有第二路音频流，就会静音。
  
  ffplay test.mkv -vst 1
  //表示播放第一路视频流，如果参数ast后面跟的是2，那么就播放第二路视频流，如果没有第二路视频流，就会是黑屏即什么都不显示。
  复制代码
  ```

- 播放yuv文件

  ```
  ffplay -f rawvideo -video_size width*height testVideo.yuv
  复制代码
  ```

- 播放pcm文件

  ```
  ffplay song.pcm -f s16le -channels 2 -ar 44100
  复制代码
  ```

  或者

  ```
  ffplay -f s16le -ar 44100 -ac 1 song.pcm
  复制代码
  ```

-f 表示音频的格式，你可以使用`ffmpeg -formats`命令查看支持的格式列表：

```

```
qis-Mac-mini:tvuDebug qi$ ffmpeg -formats | grep PCM
 DE alaw            PCM A-law
 DE f32be           PCM 32-bit floating-point big-endian
 DE f32le           PCM 32-bit floating-point little-endian
 DE f64be           PCM 64-bit floating-point big-endian
 DE f64le           PCM 64-bit floating-point little-endian
 DE mulaw           PCM mu-law
 DE s16be           PCM signed 16-bit big-endian
 DE s16le           PCM signed 16-bit little-endian
 DE s24be           PCM signed 24-bit big-endian
 DE s24le           PCM signed 24-bit little-endian
 DE s32be           PCM signed 32-bit big-endian
 DE s32le           PCM signed 32-bit little-endian
 DE s8              PCM signed 8-bit
 DE u16be           PCM unsigned 16-bit big-endian
 DE u16le           PCM unsigned 16-bit little-endian
 DE u24be           PCM unsigned 24-bit big-endian
 DE u24le           PCM unsigned 24-bit little-endian
 DE u32be           PCM unsigned 32-bit big-endian
 DE u32le           PCM unsigned 32-bit little-endian
 DE u8              PCM unsigned 8-bit

```
复制代码
```

- 播放YUV420P格式视频帧

  ```
  ffplay -f rawvideo -pixel_format yuv420p -s 480*480 texture.yuv
  复制代码
  ```

- 播放RGB图像

```
ffplay -f rawvideo -pixel_format rgb24 -s 480*480 texture.rgb
复制代码
```

对于视频播放器，不得不提一个问题就是音画同步，在ffplay中音画同步的实现方式有三种，分别是：以音频为主时间轴作为同步源；以视频为主时间轴作为同步源；以外部时钟为主时间轴作为同步源。

在ffplay中默认的对齐方式就是以音频为基准进行对齐的，那么以音频为对齐基准是如何实现的呢？

播放器收到的视频帧和音频帧都会有时间戳(PTS时钟)来标识它实际什么时刻进行展示。实际的对齐策略如下：比较视频当前的播放时间和音频当前的播放时间，如果视频播放的过快，则通过加大延迟或者重复播放来降低视频播放速度；如果视频播放慢了，则通过减少延迟或者丢帧来追赶音频播放的时间点。关键在于音视频时间的比较以及延迟的计算，当然在比较过程中会设置一个阈值(Threshold)，若超过预设的阈值就应该做调整(丢帧渲染或者重复渲染),这就是对齐策略。

对于ffplay可以明确指定是哪一种对齐方式：

- 以音频为基准进行音视频同步

  ```
  ffplay test.mp4 -sync audio
  复制代码
  ```

- 以视频为基准进行音视频同步

  ```
  ffplay test.mp4 -sync video
  复制代码
  ```

- 以外部时钟为基准进行音视频同步

  ```
  ffplay test.mp4 -sync ext
  复制代码
  ```

#### ffmpeg

ffmpeg是强大的媒体文件转换工具。它可以转换任何格式的媒体文件，并且还可以利用自己的AudioFilter以及VideoFilter进行处理和编辑，总之一句话，有了它，进行离线处理视频时可以做任何你想做的事情。

- 列出ffmpeg支持的所有格式

  ```
  ffmpeg -formats
  复制代码
  ```

- 剪切一段媒体文件，可以是音频或视频文件

  ```
  ffmpeg -i input.mp4 -ss 00:00:50.0 -codec copy -t 20 output.mp4 // 将文件input.mp4从第50s开始剪切20s的时间，输出到文件output.mp4中，其中-ss指定偏移时间，-t指定时长
  
  复制代码
  ```

- 如果在手机中录制了一个时间比较长的视频无法分享到微信中，那么可以使用ffmpeg将该文件分割为多个文件

  ```
  ffmpeg -i input.mp4 -t 00:00:50 -c copy small-1.mp4 -ss 00:00:50 -codec copy small-2.mp4
  复制代码
  ```

- 提取额一个视频文件中的音频文件

  ```
  ffmpeg -i input.mp4 -vn -acodec copy output.m4a
  复制代码
  ```

- 使一个视频中的音频静音，即只保留视频

  ```
  ffmpeg -i input.mp4 -an -vcodec copy output.mp4
  复制代码
  ```

- 从MP4文件中抽取视频流导出为裸 H264数据

  ```
  ffmpeg -i output.mp4 -an -vcodec copy -bsf:v h264_mp4toannexb output.h264
  复制代码
  ```

- 使用AAC音频的数据和H264的视频生成MP4文件

  ```
  ffmpeg -i test.aac -i test.h264 -acodec copy -bsf:a aac_adtstoasc -vcodec copy -f mp4 output.mp4
  复制代码
  ```

  上述代码中使用了一个名为aac_adtstoasc的bitstream filter, AAC格式也有两种封装格式。

- 对音频文件的编码格式做转换

  ```
  ffmpeg -i input.wav -acodec libfdk_aac output.aac
  复制代码
  ```

- 从WAV音频文件中到处PCM裸数据

  ```
  ffmpeg -i input.wav -acodec pcm_s16le -f s16le output.pcm
  复制代码
  ```

这样就可以导出用16个bit来表示一个sample的pcm数据了，并且每个sample的字节排列顺序都是小尾端表示的格式，声道数和采样率使用的都是WAV文件的声道数和采样率的PCM数据。

- 重新编码视频文件，复制音频流，同时封装到MP4格式的文件中

  ```
  ffmpeg -i input.flv -vcodec libx264 -acodec copy output.mp4
  复制代码
  ```

- 将一个MP4格式的视频转换成为git格式的动图

  ```
  ffmpeg -i input.mp4 -vf scale=100:-1 -t 5 -r 10 image.gif
  复制代码
  ```

上述代码按照分辨比例不动宽度改为100(使用VideoFilter的scaleFilter)，帧率改为10(-r)，只处理前5秒钟(-t)的视频，生成gif。

- 将一个视频的画面部分生成图片，比如要分析一个视频里面的每一帧都是什么内容的时候，可能就需要用到这个命令了

  ```
  ffmpeg -i output.mp4 -r 0.25 frames_%04d.png
  复制代码
  ```

上述这个命令每四秒钟截取一帧视频画面生成一张图片，生成的图片从frames_0001.png开始一直递增下去。

- 使用一组图片可以组成一个gif，如果你连拍了一组照片，就可以使用下面的命令生成一个gif

  ```
  ffmpeg -i frames_%04.png -r 5 output.gif
  复制代码
  ```

- 使用音量效果器，可以改变一个音频媒体文件中的音量

  ```
  ffmpeg -i input.wav -af 'volume=0.5' output.wav
  复制代码
  ```

上述命令是将input.wav中的音量减小一半，输出到output.wav文件中，可以直接播放来听，或者放到一些音频编辑软件中直接观看波形幅度的效果。

- 淡入效果器的使用

  ```
  ffmpeg -i input.wav -filter_complex afade=t=in:ss=0:d=5 output.wav
  复制代码
  ```

上述命令可以将input.wav文件中的前5s做一个淡入效果，输出到output.wav中，可以将处理之前和处理之后的文件拖到Audacity音频编辑软件中查看波形图。

- 淡出效果器的使用

  ```
  ffmpeg -i input.wav -filter_complex afade=t=out:st=200:d=5 output.wav
  复制代码
  ```

上述命令可以将input.wav文件从200s开始，做5s的淡出效果，并放到output.wav文件中

*将两路声音进行合并，比如要给一段声音加上背景音乐

```
	ffmpeg -i vocal.wav -i accompany.wav -filter_complex amix=inputs=2:duration=shortest output.wav
复制代码
```

上述命令是将vocal.wav和accompany.wav两个文件记性mix，按照时间长度较短的音频文件的时间长度作为最终输出的output.wav的时间长度

- 对声音进行变速但不变调效果器的使用

  ```
  ffmpeg -i vocal.wav -filter_complex atempo=0.5 output.wav
  复制代码
  ```

上述命令是将vocal.wav按照0.5倍的速度尽心刚处理生成output.wav，时间长度将会变为输入的2倍。但是音高是不变的，这就是大家常说的变速不变调。

- 为视频添加水印效果

```
ffmpeg -i input.mp4 -i changeba_icon.png -filter_complex '[0:v][1:v]overlay=main_w-overlay_w-10:10:1[out]' -map '[out]' output.mp4
复制代码
```

上述命令包含了几个内置参数，main_w代表主视频宽度，overlay_w代表水印宽度，main_h戴波啊主视频高度，overlay_h代表水印高度。

- 视频提高效果器的使用

```
ffmpeg -i input.fly -c:v libx264 -b:v 800k -c:a libfdk_aac -vf eq=brightness=0.25 -f mp4 output.mp4
复制代码
```

提亮参数是bitrate，取值范围是从-1.0到1.0,默认值是0

- 为视频增加对比度效果

```
ffmpeg -i input.flv -c:v libx264 -b:v 800k -c:a libfdk_aac -vf eq=contrast=1.5 -f mp4 output.mp4
复制代码
```

对比度参数是contrast,取值范围是从-2.0到2.0,默认值是1.0

- 视频旋转效果器的使用

```
ffmpeg -i input.mp4 -vf "transpose=1" -b:v 600k output.mp4
复制代码
```

- 视频裁剪效果器的使用

```
ffmpeg -i input.mp4 -an -vf "crop=240:480:120:0" -vcodec libx264 -b:v 600k output.mp4
复制代码
```

### 在mac平台编译出ios开发所用的ffmpeg库

#### 安装homebrew

`Homebrew`是一款自由及开放源代码的软件包管理系统，用以简化Mac OS X系统上的软件安装过程，最初由马克斯·霍威尔（Max Howell）写成。因其可扩展性得到了一致好评[1]，而在Ruby on Rails社区广为人知。

[how to install HomeBrew](https://link.juejin.im/?target=https%3A%2F%2Fbrew.sh%2F)

#### 下载编译脚本文件

编译`ffmpeg`脚本的文件我们用 gas-preprocessor.

下载后把`gas-preprocessor.pl`拷贝到 `/usr/local/bin/`目录下，然后为文件开启可执行权限：

```
chmod 777 /usr/local/bin/gas-preprocessor.pl
复制代码
```

#### 安装yasm

在计算机领域中，Yasm是英特尔x86架构下的一个汇编器和反汇编器。它可以用来编写16位、32位（IA-32）和64位（x86-64）的程序。Yasm是一个完全重写的Netwide汇编器（NASM）。Yasm通常可以与NASM互换使用，并支持x86和x86-64架构。

安装 `yasm

```
brew install yasm
复制代码
```

#### ffmpeg ios编译脚本

[FFmpeg iOS build script](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fmediaios%2FFFmpeg-iOS-build-script)

以上是ffmpeg 的编译脚本，能编译出ios平台的库。具体详细信息可查看wiki.

直接运行以下命令编译出我们需要的库(可选择编译那种cpu架构)：

```
 ./build-ffmpeg-iOS-framework.sh 
复制代码
```

编译成功后，目录结构如图所示：



![img](https://user-gold-cdn.xitu.io/2019/5/31/16b0caa15b1d0209?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



#### 在项目中引入ffmpeg

直接把上面我们编译成功的ffmpeg库`FFmpeg-iOS`整体拖入工程，然后再加入如下库：

- libiconv.tdb
- libbz2.tbd
- libz.tbd



![img](https://user-gold-cdn.xitu.io/2019/5/31/16b0cb0e00445c61?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



### 简单介绍ffmpeg

#### ffmpeg库简介

ffmpeg一共包含8个库：

- avcodec : 编解码（最重要的库）
- avformat : 封装格式处理
- avfilter : 滤镜特效处理
- avdevice : 各种设备的输入输出
- avutil : 工具库(大部分库都需要这个库的支持)
- postproc : 后加工
- swresample : 音频采样数据格式转换
- swscale : 视频像素数据格式转换

#### ffmpeg数据结构分析

- AVFormatContext: 封装格式上下文结构体，也是统领全局的结构体，保存了视频文件封装格式相关信息
  - iformat: 输入视频的 AVInputFormat
  - nb_streams : 输入视频的 AVStream个数
  - streams : 输入视频的AVStream[] 数组
  - duration : 输入视频的时长（以微妙为单位）
  - bit_rate : 输入视频的码率
- AVInputFormat:每种封装格式（例如 FLV,MKV,MP4，AVI）对应一个结构体
  - name : 封装格式名称
  - long_name : 封装格式的长名称
  - extensions : 封装格式的扩展名
  - id : 封装格式ID
- AVStream:视频文件中每个视频(音频)流对应一个该结构体
  - id : 序号
  - codec : 该流对应的AVCodecContext
  - time_base : 该流的时基
  - r_frame_rate : 该流的帧率
- AVCodecContext:编码器上下文结构体，保存了视频（音频）编解码器相关信息
  - codec ： 编解码器的AVCodec
  - width,height : 图像的宽高（只针对视频）
  - pix_fmt : 像素格式（只针对视频）
  - sample_rate : 采样率（只针对音频）
  - channels : 声道数（只针对音频）
  - sample_fmt : 采样格式（只针对音频）
- AVCodec:每种视频(音频)编解码器（例如 H264解码器）对应一个该结构体
  - name :编解码器名称
  - long_name : 编解码器长名称
  - type : 编解码器类型
  - id : 编解码器ID
- AVPacket :存储一帧压缩编码数据
  - pts : 显示时间戳
  - dts : 解码时间戳
  - data : 压缩编码数据
  - size : 压缩编码数据大小
  - stream_index : 所属的AVStream
- AVFrame :存储一帧解码后像素（采样）数据
  - data : 解码后的图像像素数据 （音频采样数据）
  - linesize : 对视频来说是图像中一行像素大小，对音频来说是整个音频帧的大小
  - width,height : 图像的宽高(只针对视频)
  - key_frame : 是否为关键帧 （只针对视频）
  - pict_type : 帧类型（只针对视频）。 例如 I,B,P

## 利用ffmpeg对实时视频做h264软编码

大致流程如下：



![img](https://user-gold-cdn.xitu.io/2019/5/31/16b0caa165222b9c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)