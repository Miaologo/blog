---
title: iOS开发之VideoToolBox
date: 2019-08-15 16:49:12
tags:	
	- iOS
	- video
	- videotoolbox
---



---



**I帧**：关键帧 描述的是一张完整的图片，一组图片中一般选择第一张

**B帧**：双向参考帧，保存两边不一样的数据（可以丢弃）

**P帧**：向前参考帧，只会保存跟前一张不一样的数据

**I帧丢失就不能正确解码****如果解码时要等到后一帧传过来再解码，一定时间内没有收到的话，可以丢弃B帧**

**GOF(Group of Frame)一组帧**就是一个I帧到下一个I帧.这一组的数据.包括B帧/P帧.我们称为GOF（GOP）.

- 如果GOP分组中的P帧丢失就会造成解码端的图像发生错误.
- 为了避免花屏问题的发生,一般如果发现P帧或者I帧丢失.就不显示本GOP内的所有帧.只到下一个I帧来后重新刷新图像.
- 当这时因为没有刷新屏幕.丢包的这一组帧全部扔掉了.图像就会卡在哪里不动.这就是卡顿的原因. 所以总结起来,花屏是因为你丢了P帧或者I帧.导致解码错误. 而卡顿是因为为了怕花屏,将整组错误的GOP数据扔掉了.直达下一组正确的GOP再重新刷屏.而这中间的时间差,就是我们所感受的卡顿.

**SPS/PPS实际上就是存储GOP的参数.**

**SPS**: (Sequence Parameter Set,序列参数集)存放帧数,参考帧数目,解码图像尺寸,帧场编码模式选择标识等.

**PPS**:(Picture Parameter Set,图像参数集).存放熵编码模式选择标识,片组数目,初始量化参数和去方块滤波系数调整标识等.(与图像相关的信息) 大家只要记住,在一组帧之前我们首先收到的是SPS/PPS数据.如果没有这组参数的话,我们是无法解码

-----
https://juejin.im/post/5a30de56f265da431a432f19


# 接口概述

在iOS中，与视频相关的接口有5个，从顶层开始分别是 AVKit - AVFoundation - VideoToolbox - Core Media - Core Video

其中VideoToolbox可以将视频解压到CVPixelBuffer,也可以压缩到CMSampleBuffer。

如果需要使用硬编码的话，在5个接口中，就需要用到AVKit，AVFoundation和VideoToolbox。在这里我就只介绍VideoToolbox。

## VideoToolbox对象

- CVPixelBuffer - 未压缩光栅图像缓存区(Uncompressed Raster Image Buffer)

![1](/Users/miaoliu/blog/source/_posts/iOS开发之VideoToolBox/1.png)

- CVPixelBufferPool - 顾名思义，存放CVPixelBuffer



![2](/Users/miaoliu/blog/source/_posts/iOS开发之VideoToolBox/2.png)



- pixelBufferAttributes - CFDictionary对象，可能会包含视频的宽高，像素格式类型（32RGBA, YCbCr420），是否可以用于OpenGL ES等相关信息
- CMTime - 分子是64-bit的时间值，分母是32-bit的时标(time scale)
- CMVideoFormatDescription - 视频宽高，格式(kCMPixelFormat_32RGBA, kCMVideoCodecType_H264), 其他诸如颜色空间等信息的扩展
- CMBlockBuffer -





- CMSampleBuffer - 对于压缩的视频帧来说，包含了CMTime，CMVideoFormatDesc和CMBlockBuffer；对于未压缩的光栅图像的话，则包含了CMTime，CMVideoFormatDesc和CMPixelBuffer



![CMSampleBuffer](https://user-gold-cdn.xitu.io/2017/12/13/1604ee4c5d85d629?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



- CMClock - 封装了时间源，其中`CMClockGetHostTimeClock()`封装了`mach_absolute_time()`
- CMTimebase - CMClock上的控制视图。提供了时间的映射:`CMTimebaseSetTime(timebase, kCMTimeZero);`; 速率控制: `CMTimebaseSetRate(timebase, 1.0);`



![CMTimebase](https://user-gold-cdn.xitu.io/2017/12/13/1604ee4c5f280d7b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



# Case One - 播放视频流文件

使用VideoToolbox硬编码来播放网络上的流文件时，整个完整的流程是这样的：获取网络文件 -> 获取多个已压缩的H.264采样 -> 调用AVSampleBufferDisplayLayer -> 播放



![Overview](https://user-gold-cdn.xitu.io/2017/12/13/1604ee4c5fe28a6b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



更详细点看的话，在AVSamplerBufferDisplayLayer这一层中，我们还需要将视频解码到CVPixelBuffer中



![More Detail](https://user-gold-cdn.xitu.io/2017/12/13/1604ee4c82153ff6?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



## 处理过程

下面要介绍的就是流文件到CMSampleBuffers的H.264的处理过程：

在H.264的语法中，有一个最基础的层，叫做Network Abstraction Layer, 简称为NAL。H.264流数据正是由一系列的NAL单元(NAL Unit, 简称NALU)组成的。



![NALUs](https://user-gold-cdn.xitu.io/2017/12/13/1604ee4c83c52219?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



一个NALU可能包含有：

- 视频帧(或者是视频帧的片段) - P帧， I帧， B帧



![Video Frame](https://user-gold-cdn.xitu.io/2017/12/13/1604ee4c84fc544b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



- H.264属性集合：Sequence Parameter Set(SPS)和Picture Parameter Set（PPS）

流数据中，属性集合可能是这样的：



![Parameter Set in Stream](https://user-gold-cdn.xitu.io/2017/12/13/1604ee4c844b7ec8?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



经过处理之后，在Format Description中则是:



![Parameter Set in Format Description](https://user-gold-cdn.xitu.io/2017/12/13/1604ee4c8552c28b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



要从基础的流数据将SPS和PPS转化为Format Desc中的话，需要调用`CMVideoFormatDescriptionCreateFromH264ParameterSets()`方法

### NALU header

对于流数据来说，一个NALU的Header中，可能是0x00 00 01或者是0x00 00 00 01作为开头(两者都有可能，下面以0x00 00 01作为例子)。0x00 00 01因此被称为开始码(Start code).



![Start Code](https://user-gold-cdn.xitu.io/2017/12/13/1604ee4c925bdd9c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



一个MP4文件的话，则是以0x00 00 80 00作为开头。因此要将基本流数据转换成CMSampleBuffer的话，需CMBlockBuffer+CMVideoFormatDesc+CMTime(Optional)。我们可以调用`CMSampleBufferCreate()`来完成转换



![NALU Conversion](https://user-gold-cdn.xitu.io/2017/12/13/1604ee4cc90577fc?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



## 时间控制

如果需要控制每一帧图片的显示时间的话，可以通过CMTimebase进行时间的控制



![Time](https://user-gold-cdn.xitu.io/2017/12/13/1604ee4ca5ca8b4e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



```
sbDisplayLayer.controlTimebase = CMTimebaseCreateWithMasterClock(CMClockGetHostTimeClock());
CMTimebaseSetTime(sbDisplayLayer.controlTimebase, CMTimeMake(5, 1));CMTimebaseSetRate(sbDisplayLayer.controlTimebase, 1.0);
复制代码
```

## 总结

播放一个网络流文件的流程大概就是这样，总结起来就是：

1） 创建AVSampleBufferDisplayLayer

2）将H.264基础流转换为CMSampleBuffer

3）将CMSampleBuffers提供给AVSampleBufferDisplayLayer

4）可以使用自定义的CMTimebase

# Case Two - 从已压缩的流中获取CVPixelBuffers



![AVSampleBufferDisplayLayer](https://user-gold-cdn.xitu.io/2017/12/13/1604ee4cbf838556?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



## 获取解码器



![Get Access to the Decoder](https://user-gold-cdn.xitu.io/2017/12/13/1604ee4cada02752?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



这个步骤中，我们所需要的有：

- 源数据的描述 - CMVideoFormatDescription
- 输出缓存所需要的参数 - pixelBufferAttributes:

e.g :

```
NSDictionary *destinationImageBufferAttributes = [NSDictionary dictionaryWithObjectsAndKeys:

[NSNumber numberWithBool:YES],(id)kCVPixelBufferOpenGLESCompatibilityKey,nil];
复制代码
```

- 回调函数 - VTDecompressionOutputCallback。该回调函数接收一下参数： CVPixelBuffer输出，时间戳，编码的错误码，丢弃的帧

------

以上为Apple的Keynote中的介绍，下面通过代码来解释

在我自己的Project中，我利用FFMPEG和VideoToolbox来进行网络MP4文件的解析。关于FFMPEG的部分我就不解释了。只贴VideoToolbox硬解码部分。

另外，关于H.264开始码这部分相关的信息，也可以参考我[另一篇文章](https://link.juejin.im/?target=http%3A%2F%2Fsimking.github.io%2F2016%2F05%2F22%2FH264-Format-Intro-Translation%2F)

-----
https://juejin.im/post/5d4949bef265da03a0495ff2

# 编码

直接从采集的代理方法**captureOutput: didOutputSampleBuffer: fromConnection:**里开始对视频帧就行编码。大致的流程分为三步：

1. 准备编码器，即创建session：**VTCompressionSessionCreate**，并设置编码器属性；
2. 开始编码：**VTCompressionSessionEncodeFrame**
3. 编码完成的回调里处理数据：添加起始码**"\x00\x00\x00\x01"**，添加**sps pps**等。
4. 结束编码，清除数据，释放资源。

## 准备编码器

1. **创建session**： VTCompressionSessionCreate
2. **设置属性**：VTSessionSetProperty 是否实时编码输出、是否产生B帧、设置关键帧、设置期望帧率、设置码率、最大码率值等等
3. **准备开始编码**：VTCompressionSessionPrepareToEncodeFrames

```
-(void)initVideoToolBox
{
    // cEncodeQueue是一个串行队列
    dispatch_sync(cEncodeQueue, ^{

        frameID = 0;
        int width = 480,height = 640;
        
        //创建编码session
        OSStatus status = VTCompressionSessionCreate(NULL, width, height, kCMVideoCodecType_H264, NULL, NULL, NULL, didCompressH264, (__bridge void *)(self), &cEncodeingSession);
        NSLog(@"H264:VTCompressionSessionCreate:%d",(int)status);
        
        if (status != 0) {
            NSLog(@"H264:Unable to create a H264 session");
            return ;
        }
        
        //设置实时编码输出（避免延迟）
        VTSessionSetProperty(cEncodeingSession, kVTCompressionPropertyKey_RealTime, kCFBooleanTrue);
        VTSessionSetProperty(cEncodeingSession, kVTCompressionPropertyKey_ProfileLevel,kVTProfileLevel_H264_Baseline_AutoLevel);
        
        //是否产生B帧(因为B帧在解码时并不是必要的,是可以抛弃B帧的)
        VTSessionSetProperty(cEncodeingSession, kVTCompressionPropertyKey_AllowFrameReordering, kCFBooleanFalse);
        
        //设置关键帧（GOPsize）间隔，GOP太小的话图像会模糊
        int frameInterval = 10;
        CFNumberRef frameIntervalRaf = CFNumberCreate(kCFAllocatorDefault, kCFNumberIntType, &frameInterval);
        VTSessionSetProperty(cEncodeingSession, kVTCompressionPropertyKey_MaxKeyFrameInterval, frameIntervalRaf);
        
        //设置期望帧率，不是实际帧率
        int fps = 10;
        CFNumberRef fpsRef = CFNumberCreate(kCFAllocatorDefault, kCFNumberIntType, &fps);
        VTSessionSetProperty(cEncodeingSession, kVTCompressionPropertyKey_ExpectedFrameRate, fpsRef);
        
        //码率的理解：码率大了话就会非常清晰，但同时文件也会比较大。码率小的话，图像有时会模糊，但也勉强能看
        //码率计算公式，参考印象笔记
        //设置码率、上限、单位是bps
        int bitRate = width * height * 3 * 4 * 8;
        CFNumberRef bitRateRef = CFNumberCreate(kCFAllocatorDefault, kCFNumberSInt32Type, &bitRate);
        VTSessionSetProperty(cEncodeingSession, kVTCompressionPropertyKey_AverageBitRate, bitRateRef);
        
        //设置码率，均值，单位是byte
        int bigRateLimit = width * height * 3 * 4;
        CFNumberRef bitRateLimitRef = CFNumberCreate(kCFAllocatorDefault, kCFNumberSInt32Type, &bigRateLimit);
        VTSessionSetProperty(cEncodeingSession, kVTCompressionPropertyKey_DataRateLimits, bitRateLimitRef);
        
        //准备开始编码
        VTCompressionSessionPrepareToEncodeFrames(cEncodeingSession);

    });
    
}
复制代码
```

**VTCompressionSessionCreate**创建编码对象参数详解：

![image.png](https://user-gold-cdn.xitu.io/2019/8/6/16c66480af98c9e8?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



- **allocator**：NULL 分配器,设置NULL为默认分配
- **width**：width
- **height**：height
- **codecType**：编码类型,如kCMVideoCodecType_H264
- **encoderSpecification**：NULL encoderSpecification: 编码规范。设置NULL由videoToolbox自己选择
- **sourceImageBufferAttributes**：NULL sourceImageBufferAttributes: 源像素缓冲区属性.设置NULL不让videToolbox创建,而自己创建
- **compressedDataAllocator**：压缩数据分配器.设置NULL,默认的分配
- **outputCallback**：编码回调 ， 当VTCompressionSessionEncodeFrame被调用压缩一次后会被异步调用.这里设置的函数名是 didCompressH264
- **outputCallbackRefCon**：回调客户定义的参考值，此处把self传过去，因为我们需要在C函数中调用self的方法，而C函数无法直接调self
- **compressionSessionOut**： 编码会话变量

## 开始编码

1. **拿到未编码的视频帧**： CVImageBufferRef imageBuffer = (CVImageBufferRef)CMSampleBufferGetImageBuffer(sampleBuffer);
2. **设置帧时间**：CMTime presentationTimeStamp = CMTimeMake(frameID++, 1000);
3. **开始编码**：调用 VTCompressionSessionEncodeFrame进行编码

```
 - (void)captureOutput:(AVCaptureOutput *)captureOutput didOutputSampleBuffer:(CMSampleBufferRef)sampleBuffer fromConnection:(AVCaptureConnection *)connection
{
    //开始视频录制，获取到摄像头的视频帧，传入encode 方法中
    dispatch_sync(cEncodeQueue, ^{
        [self encode:sampleBuffer];
    });
}
复制代码
- (void) encode:(CMSampleBufferRef )sampleBuffer
{
  //拿到每一帧未编码数据
  CVImageBufferRef imageBuffer = (CVImageBufferRef)CMSampleBufferGetImageBuffer(sampleBuffer);

  //设置帧时间
  CMTime presentationTimeStamp = CMTimeMake(frameID++, 1000);

  //开始编码 
  OSStatus statusCode = VTCompressionSessionEncodeFrame(cEncodeingSession, imageBuffer, presentationTimeStamp, kCMTimeInvalid, NULL, NULL, &flags);

  if (statusCode != noErr) {
        //编码失败
        NSLog(@"H.264:VTCompressionSessionEncodeFrame faild with %d",(int)statusCode);
        
        //释放资源
        VTCompressionSessionInvalidate(cEncodeingSession);
        CFRelease(cEncodeingSession);
        cEncodeingSession = NULL;
        return;
    }
}

复制代码
```

**VTCompressionSessionEncodeFrame**编码函数参数详解：

![image.png](https://user-gold-cdn.xitu.io/2019/8/6/16c66480af52f4ef?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



- **session**：编码会话变量
- **imageBuffer**：未编码的数据
- **presentationTimeStamp**：获取到的这个sample buffer数据的展示时间戳。每一个传给这个session的时间戳都要大于前一个展示时间戳
- **duration**：对于获取到sample buffer数据,这个帧的展示时间.如果没有时间信息,可设置kCMTimeInvalid.
- **frameProperties**：包含这个帧的属性.帧的改变会影响后边的编码帧.
- **sourceFrameRefcon**：回调函数会引用你设置的这个帧的参考值.
- **infoFlagsOut**：指向一个VTEncodeInfoFlags来接受一个编码操作.如果使用异步运行,kVTEncodeInfo_Asynchronous被设置；同步运行,kVTEncodeInfo_FrameDropped被设置；设置NULL为不想接受这个信息.

## 编码完成后数据处理

1. **判断是否是关键帧**：是的话，**CMVideoFormatDescriptionGetH264ParameterSetAtIndex**获取sps和pps信息，并转换为二进制写入文件或者进行上传
2. **组装NALU数据**： 获取编码后的h264流数据：CMBlockBufferRef dataBuffer = CMSampleBufferGetDataBuffer(sampleBuffer)，通过 首地址 、单个长度、 总长度通过dataPointer指针偏移做遍历 OSStatus statusCodeRet = CMBlockBufferGetDataPointer(dataBuffer, 0, &length, &totalLength, &dataPointer); 读取数据时有个大小端模式：网络传输一般都是大端模式

```
/*
    1.H264硬编码完成后，回调VTCompressionOutputCallback
    2.将硬编码成功的CMSampleBuffer转换成H264码流，通过网络传播
    3.解析出参数集SPS & PPS，加上开始码组装成 NALU。提现出视频数据，将长度码转换为开始码，组成NALU，将NALU发送出去。
 */
void didCompressH264(void *outputCallbackRefCon, void *sourceFrameRefCon, OSStatus status, VTEncodeInfoFlags infoFlags, CMSampleBufferRef sampleBuffer)
{
    NSLog(@"didCompressH264 called with status %d infoFlags %d",(int)status,(int)infoFlags);
    //状态错误
    if (status != 0) {
        return;
    }
    
    //没准备好
    if (!CMSampleBufferDataIsReady(sampleBuffer)) {
        NSLog(@"didCompressH264 data is not ready");
        return;
    }
    
    ViewController *encoder = (__bridge ViewController *)outputCallbackRefCon;
    
    //判断当前帧是否为关键帧
    CFArrayRef array = CMSampleBufferGetSampleAttachmentsArray(sampleBuffer, true);
    CFDictionaryRef dic = CFArrayGetValueAtIndex(array, 0);
    bool keyFrame = !CFDictionaryContainsKey(dic, kCMSampleAttachmentKey_NotSync);
    
    //判断当前帧是否为关键帧
    //获取sps & pps 数据 只获取1次，保存在h264文件开头的第一帧中
    //sps(sample per second 采样次数/s),是衡量模数转换（ADC）时采样速率的单位
    //pps()
    if (keyFrame) {
        //图像存储方式，编码器等格式描述
        CMFormatDescriptionRef format = CMSampleBufferGetFormatDescription(sampleBuffer);
        
        //sps
        size_t sparameterSetSize,sparameterSetCount;
        const uint8_t *sparameterSet;
        OSStatus statusCode = CMVideoFormatDescriptionGetH264ParameterSetAtIndex(format, 0, &sparameterSet, &sparameterSetSize, &sparameterSetCount, 0);
        
        if (statusCode == noErr) {
            
            //获取pps
            size_t pparameterSetSize,pparameterSetCount;
            const uint8_t *pparameterSet;
            
            //从第一个关键帧获取sps & pps
            OSStatus statusCode = CMVideoFormatDescriptionGetH264ParameterSetAtIndex(format, 1, &pparameterSet, &pparameterSetSize, &pparameterSetCount, 0);
            
            //获取H264参数集合中的SPS和PPS
            if (statusCode == noErr)
            {
                NSData *sps = [NSData dataWithBytes:sparameterSet length:sparameterSetSize];
                NSData *pps = [NSData dataWithBytes:pparameterSet length:pparameterSetSize];
                
                if(encoder)
                {
                    [encoder gotSpsPps:sps pps:pps];
                }
            }
        }
    }
    
    CMBlockBufferRef dataBuffer = CMSampleBufferGetDataBuffer(sampleBuffer);
    size_t length,totalLength;
    char *dataPointer;
    OSStatus statusCodeRet = CMBlockBufferGetDataPointer(dataBuffer, 0, &length, &totalLength, &dataPointer);
    if (statusCodeRet == noErr) {
        size_t bufferOffset = 0;
        static const int AVCCHeaderLength = 4;//返回的nalu数据前4个字节不是001的startcode,而是大端模式的帧长度length
        
        //循环获取nalu数据
        while (bufferOffset < totalLength - AVCCHeaderLength) {
            
            uint32_t NALUnitLength = 0;
            
            //读取 一单元长度的 nalu
            memcpy(&NALUnitLength, dataPointer + bufferOffset, AVCCHeaderLength);
            
            //从大端模式转换为系统端模式
            NALUnitLength = CFSwapInt32BigToHost(NALUnitLength);
            
            //获取nalu数据
            NSData *data = [[NSData alloc]initWithBytes:(dataPointer + bufferOffset + AVCCHeaderLength) length:NALUnitLength];
            
            //将nalu数据写入到文件
            [encoder gotEncodedData:data isKeyFrame:keyFrame];
            
            //move to the next NAL unit in the block buffer
            //读取下一个nalu 一次回调可能包含多个nalu数据
            bufferOffset += AVCCHeaderLength + NALUnitLength;
        }
    }
}

//第一帧写入 sps & pps
- (void)gotSpsPps:(NSData*)sps pps:(NSData*)pps
{
    const char bytes[] = "\x00\x00\x00\x01";
    
    size_t length = (sizeof bytes) - 1;    // 最后一位是\0结束符
    
    NSData *ByteHeader = [NSData dataWithBytes:bytes length:length];
    
    [fileHandele writeData:ByteHeader];
    [fileHandele writeData:sps];
    [fileHandele writeData:ByteHeader];
    [fileHandele writeData:pps];
}

- (void)gotEncodedData:(NSData*)data isKeyFrame:(BOOL)isKeyFrame
{
    if (fileHandele != NULL) {
        //添加4个字节的H264 协议 start code 分割符
        //一般来说编码器编出的首帧数据为PPS & SPS
        //H264编码时，在每个NAL前添加起始码 0x00000001,解码器在码流中检测起始码，当前NAL结束。
        const char bytes[] ="\x00\x00\x00\x01";
        //长度
        size_t length = (sizeof bytes) - 1;
        
        //头字节
        NSData *ByteHeader = [NSData dataWithBytes:bytes length:length];
        //写入头字节
        [fileHandele writeData:ByteHeader];
        
        //写入H264数据
        [fileHandele writeData:data];
    }
}
复制代码
```

## 结束编码

```
-(void)endVideoToolBox
{
    VTCompressionSessionCompleteFrames(cEncodeingSession, kCMTimeInvalid);
    VTCompressionSessionInvalidate(cEncodeingSession);
    CFRelease(cEncodeingSession);
    cEncodeingSession = NULL;  
}
```

