## Scenario Design



#### MySQL如何解决重复支付的问题，同理如何解决超售问题？

**重复支付问题**的解决思路很多：

1. 纯前端解决：用户在点击付款之后，前端将支付按钮置灰。然后不断轮询支付结果，如果失败则重新将支付按钮可用
2. 纯后端解决方法：在支付api没返回支付结果之前支付验证不通过，两个标记位（结果是否返回、支付是否成功），给订单表中该条记录上行锁select for update



**超售**建议通过Redis来解决，decre命令，单线程的本身就线程安全

还有就是超售这种情况一般在下单过程中发生，当超售的用户走到付款那一步时发现因为商品库存不够而没法付款也是可以接受的，因为数量不会很多





#### Kafka的消费者数量比分区多如何处理？

有消费者处于空闲状态，同组内多个消费者同时消费一个分区会导致消息消费顺序性无法保证

可以考虑设置为两个消费者组，看实际的需求





权限系统模型设计对比 

spu, sku 如何在数据库表中设计，方便扩展 

MQ的对比和场景选型

设计抖音评论系统（完整的设计流程，从场景、数据库、api实现、应用的技术栈、高并发读写的解决方案等） 

1000个人同时打车，如何合理设计派单

在Spring中如何使用线程池

项目中如何使用ThreadLocal



#### Spring项目中如何配置多个MySQL数据源

有三种方式来解决

1. **通过Mapper分包**

   定义两个Mapper包，在properties中定义两个数据库连接，确定好主次

   在Config中@PackageScan匹配到特定的数据源

   在配置类方法的实现上对主数据源采用@Primary进行标记

   还有就是Bean的名字不能一样

2. **通过AOP切片**

   AOP动态配置数据源方式缺点在于无法实现全局分布式事务，所以如果只是对接第三方数据源，不涉及到需要保证分布式事务的话，是可以作为一种选择

3. **使用数据工具MyCat，自动匹配上分库逻辑**

   这种逻辑比较好，省去了应用层的开销



#### JWT的原理，JWT的弊端，如何使用JWT？





#### 使用Java处理音视频

使用FFMPEG这个第三方工具类来操作，操作的范围也就是做一些基本的剪切、音视频分离、水印等

常使用的就是它的几个类：Grabber、Recoder

总体原理是把原音视频逐帧进行解码，然后根据我们的需要，一帧一帧地copy到新的recoder里面去

```java
package com.xl.ffmpeg;

import org.bytedeco.ffmpeg.global.avcodec;
import org.bytedeco.ffmpeg.global.avutil;
import org.bytedeco.javacv.FFmpegFrameGrabber;
import org.bytedeco.javacv.FFmpegFrameRecorder;
import org.bytedeco.javacv.FFmpegLogCallback;
import org.bytedeco.javacv.Frame;


import static org.bytedeco.ffmpeg.global.avutil.*;

public class Test4 {

    public static void main(String[] args) throws Exception {
        //开启日志
        FFmpegLogCallback.set();

        String inputUrl="2.mp4";

        //抽取视频和音频存储路径
        String outputVideoUrl="d:\\1.mp4";
        String outputaduioUrl="d:\\2.aac";
        //1.拉取视频
        FFmpegFrameGrabber grabber = new FFmpegFrameGrabber(inputUrl);
        grabber.start();

        //2.录制视频
        FFmpegFrameRecorder videoRecorder=new FFmpegFrameRecorder(outputVideoUrl,grabber.getImageWidth(),grabber.getImageHeight());
        videoRecorder.setFrameRate(25);// 设置帧率
        videoRecorder.setGopSize(25);// 设置gop,与帧率相同，相当于间隔1秒chan's一个关键帧
        videoRecorder.setVideoCodec(avcodec.AV_CODEC_ID_H264);  //压缩方式264
        videoRecorder.setPixelFormat(avutil.AV_PIX_FMT_YUV420P); //视频源数据yuv
        videoRecorder.setVideoOption("threads", "8"); //解码线程数
        videoRecorder.start();

        //3.录制音频
        FFmpegFrameRecorder audioRecorder=new FFmpegFrameRecorder(outputaduioUrl,grabber.getAudioChannels());
        audioRecorder.setAudioCodec(avcodec.AV_CODEC_ID_AAC); //设置音频压缩方式
        audioRecorder.start();

        //读取每一种数据，一帧数据可能是音频或视频
        Frame avFrame=null;
        while((avFrame = grabber.grab())!=null){

            if(avFrame.streamIndex==AVMEDIA_TYPE_VIDEO){ //视频帧
                videoRecorder.record(avFrame); //录制视频
            }else if(avFrame.streamIndex==AVMEDIA_TYPE_AUDIO){  //音频帧
                audioRecorder.record(avFrame); //录制音频
            }

        }

        //关闭相关对象
        grabber.close();
        videoRecorder.close();
        audioRecorder.close();
    }
}
```



## Speical Themes

#### Amazon Prime Video 

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230319104908758.png" alt="image-20230319104908758" style="zoom: 33%;" />





## System Optimization



#### 接口优化

优化不要想着只从sql上去入手，接口的biz逻辑当中很多时候也存在着可以通过调换代码顺序而加速响应时间的地方，多留心这些

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230403120753329.png" alt="image-20230403120753329" style="zoom:33%;" />



















## 谷粒商城

#### Redis分布式锁保证下单安全

最原始的方式通过**setIfAbsent**和**RedisScripts**进行上锁开锁

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230311120311436.png" alt="image-20230311120311436" style="zoom:33%;" />







阻塞式加锁，自动续期，出现异常自动释放

也可以指定锁占用时间`lock (int time, TimeUnit unit)`



<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230311130127710.png" alt="image-20230311130127710" style="zoom:33%;" />



<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230311131457183.png" alt="image-20230311131457183" style="zoom: 67%;" />



##### Redisson读写锁

读写情况分别对待 



**RedissonSemaphore**

也和Java中的Semaphore使用一致



**Redisson的CountDownLatch**

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230311132549340.png" alt="image-20230311132549340" style="zoom: 33%;" />



#### 缓存一致性问题

当前这个问题的解决方向是**缓存失效**或者**双写**

**缓存失效**指的是给每一个key都设置一个过期时间，然后在修改了数据库之后同步去将该缓存失效，然后由用户请求来讲新数据载入缓存。这种方法更为推荐，因为考虑到开销还是让数据真正有需求了再重新入缓存

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230311154926034.png" alt="image-20230311154926034" style="zoom: 25%;" />



**双写**则是写数据库和修改缓存一前一后同步进行

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230311155126256.png" alt="image-20230311155126256" style="zoom: 25%;" />



<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230311155023682.png" alt="image-20230311155023682" style="zoom:33%;" />



#### ES商品搜索业务



#### 订单支付业务

价格数据 Java中**BigDecimal** MySQL中**Decimal**

验价这个操作应该new - old <= 0.01 而不能直接==



锁库存的操作在库存表中额外开一个字段来表示多少库存被锁了，而不能直接去减库存数量



库存解锁，一个是同步解锁，就是支付成功之后；另一个是异步解锁，也就是一段时间后订单被自动取消后



这种业务一定要主要在如下三个方面进行控制：

- 订单库存问题，包括秒杀、及时库存同步等问题

  订单详情 -----> 确认（验价） -----> 跳转到支付页面

- 支付结果同步回调

- 全局事务的控制



#### 订单接口幂等性设计

后端为每个订单生成接口调用产生一个token

用户提交订单的时候带上这个token，我们在Redis里面做一个CAS操作，看看是否存在

如果token存在说明订单提交过了，那么本次提交就拒掉，否则就通过

原子性操作Redis使用lua脚步，然后调用redisTemplate的execute( )方法来执行



