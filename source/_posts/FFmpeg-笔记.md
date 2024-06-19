title: FFmpeg 笔记
categories: 日志
tags: [ffmpeg]
date: 2022-04-16 13:02:00
---
## 基础概念

[https://www.ruanyifeng.com/blog/2020/01/ffmpeg.html][1]

### 码率

比特率，bps，每秒位数。

### muxer

封装，把视频流、音频流、字幕流等封装到一个容器格式中。

### demuxer

从容器格式中分离出流。

### 容器

视频文件本身其实是一个容器（container），里面包括了视频和音频，也可能有字幕等其他内容。

查看 ffmpeg 支持的容器：

``` bash
ffmpeg -formats
```

### 编码格式

视频和音频都需要经过编码，才能保存成文件。不同的编码格式（CODEC），有不同的压缩率，会导致文件大小和清晰度的差异。比如`H.264`就是一种视频编码格式。

查看 ffmpeg 支持的编码格式：

``` bash
ffmpeg -codecs
```

<!--more-->

### 编码器

编码器（encoders）是实现某种编码格式的库文件。只有安装了某种格式的编码器，才能实现该格式视频/音频的编码和解码。比如`libx264`是一种`H.264`编码器。

查看 ffmpeg 已安装的编码器：

``` bash
ffmpeg -encoders
```

### 查看文件的元信息

``` bash
ffmpeg -i input.mp4 -hide_banner
```

一个容器包含多个流（stream），视频流、音频流、字幕流等。

以下示例中包含一个视频流、一个音频流、一个字幕流和三个附加文件流（字体文件）。

```
Input #0, matroska,webm, from 'F:\Why.Poverty.1of8.Poor.Us.-.An.Animated.History.of.Poverty.1080p.WEB-DL.AVC.AAC-Conan06.mkv':
  Metadata:
    title           : Poor Us: An Animated History of Poverty
    encoder         : libebml v1.3.0 + libmatroska v1.4.0
    creation_time   : 2013-01-26T09:49:37.000000Z
  Duration: 00:58:05.04, start: 0.000000, bitrate: 2719 kb/s
  Stream #0:0(eng): Video: h264 (High), yuv420p(progressive), 1920x1080, SAR 1:1 DAR 16:9, 25 fps, 25 tbr, 1k tbn, 50 tbc (default)
    Metadata:
      title           : Episode 1 of 8
  Stream #0:1(eng): Audio: aac (LC), 44100 Hz, stereo, fltp (default)
    Metadata:
      title           : Conan06
  Stream #0:2(chi): Subtitle: ass (default)
    Metadata:
      title           : by Conan06
  Stream #0:3: Attachment: ttf
    Metadata:
      filename        : 方正准园.ttf
      mimetype        : application/x-truetype-font
  Stream #0:4: Attachment: ttf
    Metadata:
      filename        : 方正兰亭特黑长.TTF
      mimetype        : application/x-truetype-font
  Stream #0:5: Attachment: ttf
    Metadata:
      filename        : 张海山锐线体简.ttf
      mimetype        : application/x-truetype-font
```

下面这个示例包含一个视频流、两个音频流（即两个音轨）：

```
Input #0, matroska,webm, from 'D:\IronMan.mkv':
  Metadata:
    encoder         : libebml v1.3.0 + libmatroska v1.4.0
    creation_time   : 2014-08-26T16:56:08.000000Z
  Duration: 02:06:01.81, start: 0.000000, bitrate: 2048 kb/s
  Stream #0:0: Video: h264 (High), yuv420p(progressive), 1280x720 [SAR 1:1 DAR 16:9], 23.98 fps, 23.98 tbr, 1k tbn, 47.95 tbc (default)
  Stream #0:1(eng): Audio: aac (HE-AAC), 48000 Hz, 5.1, fltp (default)
    Metadata:
      title           : 英语
  Stream #0:2(chi): Audio: aac (HE-AAC), 48000 Hz, 5.1, fltp
    Metadata:
      title           : 国语
```

## 教程

[http://dranger.com/ffmpeg/ffmpeg.html][2]

教程中的某些函数在 FFmpeg 新版本中已被废弃，所以代码有一些改动。

源代码：[https://gitee.com/yjwx0017/ffmpeg_tutorial][3]

编译环境：vscode + cmake + vcpkg

### 截图

实现功能：加载视频文件，将开头几帧图像保存为.ppm文件（ppm文件可用photoshop查看）

``` c++
#include <fstream>
#include <iostream>
#include <sstream>

extern "C" {
#include <libavcodec/avcodec.h>
#include <libavformat/avformat.h>
#include <libavutil/imgutils.h>
#include <libswscale/swscale.h>
}

void saveFrame(AVFrame *frame, int width, int height, int frameIndex) {
  std::stringstream filename;
  filename << "frame" << frameIndex << ".ppm";
  std::ofstream file(filename.str(), std::ios_base::binary);

  // header
  file << "P6\n" << width << " " << height << "\n255\n";

  // pixel data
  for (int i = 0; i < height; ++i) {
    file.write((const char *)(frame->data[0] + i * frame->linesize[0]),
               width * 3);
  }
}

// Making Screencaps
int main(int argc, char *argv[]) {
  int ret(0);
  AVFormatContext *formatCtx(nullptr);
  AVCodecContext *codecCtx(nullptr);
  AVFrame *frame(nullptr);
  AVFrame *frameRGB(nullptr);
  uint8_t *buffer(nullptr);

  // 取命令行参数获取文件名
  std::string filename;
  if (argc > 1) {
    filename = argv[1];
  } else {
    filename = "C:/Users/zhou/Videos/bigbuckbunny.mp4";
  }

  // 打开文件获取上下文
  ret = avformat_open_input(&formatCtx, filename.c_str(), nullptr, nullptr);
  if (ret < 0) {
    std::cout << "error: avformat_open_input() failed" << std::endl;
    goto fail;
  }

  // 初始化流信息
  ret = avformat_find_stream_info(formatCtx, nullptr);
  if (ret < 0) {
    std::cout << "error: avformat_find_stream_info() failed" << std::endl;
    goto fail;
  }

  // 打印媒体文件信息
  av_dump_format(formatCtx, 0, filename.c_str(), 0);

  // 查找视频流
  int videoStream(-1);
  for (uint32_t i = 0; i < formatCtx->nb_streams; ++i) {
    if (formatCtx->streams[i]->codecpar->codec_type == AVMEDIA_TYPE_VIDEO) {
      videoStream = i;
      break;
    }
  }

  if (videoStream == -1) {
    std::cout << "error: Didn't find a video stream";
    goto fail;
  }

  // 获取解码器
  const AVCodec *codec =
      avcodec_find_decoder(formatCtx->streams[videoStream]->codecpar->codec_id);
  if (!codec) {
    std::cout << "error: Unsupported codec" << std::endl;
    goto fail;
  }

  // 创建解码器上下文
  codecCtx = avcodec_alloc_context3(codec);
  if (!codecCtx) {
    std::cout << "error: avcodec_alloc_context3() failed" << std::endl;
    goto fail;
  }

  // 使用视频流的编码参数设置解码器上下文
  ret = avcodec_parameters_to_context(
      codecCtx, formatCtx->streams[videoStream]->codecpar);
  if (ret < 0) {
    std::cout << "error: avcodec_parameters_to_context() failed" << std::endl;
    goto fail;
  }

  // 初始化解码器上下文
  ret = avcodec_open2(codecCtx, codec, nullptr);
  if (ret < 0) {
    std::cout << "error: avcodec_open2() failed" << std::endl;
    goto fail;
  }

  // 创建帧结构，RGB格式截图帧，用于保存视频帧转换后的结果
  frameRGB = av_frame_alloc();
  if (!frameRGB) {
    std::cout << "error: av_frame_alloc() failed" << std::endl;
    goto fail;
  }

  // 计算截图帧所需的内存大小
  int numBytes = av_image_get_buffer_size(AV_PIX_FMT_RGB24, codecCtx->width,
                                          codecCtx->height, 1);
  if (numBytes < 0) {
    std::cout << "error: av_image_get_buffer_size() failed" << std::endl;
    goto fail;
  }

  // 为截图帧分配内存
  buffer = (uint8_t *)av_malloc(numBytes);
  if (!buffer) {
    std::cout << "error: av_malloc() failed" << std::endl;
    goto fail;
  }

  // 将内存关联到截图帧frameRGB
  ret = av_image_fill_arrays(frameRGB->data, frameRGB->linesize, buffer,
                             AV_PIX_FMT_RGB24, codecCtx->width,
                             codecCtx->height, 1);
  if (ret < 0) {
    std::cout << "error: av_image_fill_arrays() failed" << std::endl;
    goto fail;
  }

  // 创建格式转换上下文
  // 用于执行视频帧到截图帧之间的像素格式转换，视频帧的像素格式一般为YUV
  SwsContext *swsCtx =
      sws_getContext(codecCtx->width, codecCtx->height, codecCtx->pix_fmt,
                     codecCtx->width, codecCtx->height, AV_PIX_FMT_RGB24,
                     SWS_BILINEAR, nullptr, nullptr, nullptr);

  // 创建帧结构，用于接收视频帧
  frame = av_frame_alloc();
  if (!frame) {
    std::cout << "error: av_frame_alloc() failed" << std::endl;
    goto fail;
  }

  // 循环读取 packet，媒体流被划分为多个 packet，将 packet 喂给解码器，
  // 然后从解码器获取解码后的帧，媒体流的一个 packet 包含一个或多个帧
  AVPacket packet;
  int frameFinished(0);
  int i(0);
  while (av_read_frame(formatCtx, &packet) >= 0) {
    // 判断是否是所需的媒体流
    if (packet.stream_index == videoStream) {
      // 将 packet 发送给解码器上下文
      ret = avcodec_send_packet(codecCtx, &packet);
      if (ret < 0) {
        std::cout << "error: avcodec_send_packet() failed" << std::endl;
        goto fail;
      }

      // 从解码器上下文中获取解码后的帧
      ret = avcodec_receive_frame(codecCtx, frame);
      if (ret == 0) {
        // 获取到一帧，执行像素格式转换
        sws_scale(swsCtx, frame->data, frame->linesize, 0, codecCtx->height,
                  frameRGB->data, frameRGB->linesize);

        // 将截图帧保存到文件，前5帧
        if (i < 5) {
          saveFrame(frameRGB, codecCtx->width, codecCtx->height, i);
          ++i;
        } else {
          break;
        }
      }
    }

    // 根据 av_read_frame() 函数的文档可知，函数成功时，
    // 需要调用 av_packet_unref() 释放内存
    av_packet_unref(&packet);
  }

fail:
  av_free(buffer);
  av_frame_free(&frameRGB);
  av_frame_free(&frame);
  avcodec_free_context(&codecCtx);
  avformat_close_input(&formatCtx);

  return 0;
}
```

### 输出到屏幕

实现功能：加载视频文件，通过 SDL 显示到窗口上。

``` c++
#include <SDL/SDL.h>
#include <SDL/SDL_thread.h>
#include <fstream>
#include <iostream>

extern "C" {
#include <libavcodec/avcodec.h>
#include <libavformat/avformat.h>
#include <libavutil/imgutils.h>
#include <libswscale/swscale.h>
}

// Outputting to the Screen
int main(int argc, char *argv[]) {
  // 初始化SDL
  if (SDL_Init(SDL_INIT_VIDEO | SDL_INIT_AUDIO | SDL_INIT_TIMER) < 0) {
    std::cout << "error: Could not initialize SDL - " << SDL_GetError()
              << std::endl;
    return 0;
  }

  int ret(0);
  AVFormatContext *formatCtx(nullptr);
  AVCodecContext *codecCtx(nullptr);
  AVFrame *frame(nullptr);
  SDL_Overlay *sdlOverlay(nullptr);

  // 取命令行参数获取文件名
  std::string filename;
  if (argc > 1) {
    filename = argv[1];
  } else {
    filename = "C:/Users/zhou/Videos/bigbuckbunny.mp4";
  }

  // 打开文件
  ret = avformat_open_input(&formatCtx, filename.c_str(), nullptr, nullptr);
  if (ret < 0) {
    std::cout << "error: avformat_open_input() failed" << std::endl;
    goto fail;
  }

  // 初始化流信息
  ret = avformat_find_stream_info(formatCtx, nullptr);
  if (ret < 0) {
    std::cout << "error: avformat_find_stream_info() failed" << std::endl;
    goto fail;
  }

  // 查找视频流
  int videoStream(-1);
  for (uint32_t i = 0; i < formatCtx->nb_streams; ++i) {
    if (formatCtx->streams[i]->codecpar->codec_type == AVMEDIA_TYPE_VIDEO) {
      videoStream = i;
      break;
    }
  }

  if (videoStream == -1) {
    std::cout << "error: Didn't find a video stream";
    goto fail;
  }

  // 获取解码器
  const AVCodec *codec =
      avcodec_find_decoder(formatCtx->streams[videoStream]->codecpar->codec_id);
  if (!codec) {
    std::cout << "error: Unsupported codec" << std::endl;
    goto fail;
  }

  // 创建解码器上下文
  codecCtx = avcodec_alloc_context3(codec);
  if (!codecCtx) {
    std::cout << "error: avcodec_alloc_context3() failed" << std::endl;
    goto fail;
  }

  // 使用视频流的编码参数设置解码器上下文
  ret = avcodec_parameters_to_context(
      codecCtx, formatCtx->streams[videoStream]->codecpar);
  if (ret < 0) {
    std::cout << "error: avcodec_parameters_to_context() failed" << std::endl;
    goto fail;
  }

  // 初始化解码器上下文
  ret = avcodec_open2(codecCtx, codec, nullptr);
  if (ret < 0) {
    std::cout << "error: avcodec_open2() failed" << std::endl;
    goto fail;
  }

  // 创建 SDL_Surface
  SDL_Surface *screen =
      SDL_SetVideoMode(codecCtx->width, codecCtx->height, 0, 0);
  if (!screen) {
    std::cout << "error: SDL: could not set video mode" << std::endl;
    goto fail;
  }

  // YV12 SDL_Overlay
  sdlOverlay = SDL_CreateYUVOverlay(codecCtx->width, codecCtx->height,
                                    SDL_YV12_OVERLAY, screen);

  // 创建格式转换上下文
  SwsContext *swsCtx =
      sws_getContext(codecCtx->width, codecCtx->height, codecCtx->pix_fmt,
                     codecCtx->width, codecCtx->height, AV_PIX_FMT_YUV420P,
                     SWS_BILINEAR, nullptr, nullptr, nullptr);

  // 创建帧结构，用于接收视频帧
  frame = av_frame_alloc();
  if (!frame) {
    std::cout << "error: av_frame_alloc() failed" << std::endl;
    goto fail;
  }

  // 循环读取 packet
  AVPacket packet;
  int frameFinished(0);
  int i(0);
  while (av_read_frame(formatCtx, &packet) >= 0) {
    if (packet.stream_index == videoStream) {
      // 将 packet 发送给解码器上下文
      ret = avcodec_send_packet(codecCtx, &packet);
      if (ret < 0) {
        std::cout << "error: avcodec_send_packet() failed" << std::endl;
        goto fail;
      }

      // 从解码器上下文中获取解码后的帧
      ret = avcodec_receive_frame(codecCtx, frame);
      if (ret == 0) {
        SDL_LockYUVOverlay(sdlOverlay);

        // 将 SDL_Overlay 的内存指定到 AVFrame 结构
        // YV12 和 YUV420P 格式区别，UV分量顺序不同
        // https://blog.csdn.net/dss875914213/article/details/120836765
        AVFrame frameDst;
        frameDst.data[0] = sdlOverlay->pixels[0];
        frameDst.data[1] = sdlOverlay->pixels[2];
        frameDst.data[2] = sdlOverlay->pixels[1];
        frameDst.linesize[0] = sdlOverlay->pitches[0];
        frameDst.linesize[1] = sdlOverlay->pitches[2];
        frameDst.linesize[2] = sdlOverlay->pitches[1];

        // 执行像素格式转换
        sws_scale(swsCtx, frame->data, frame->linesize, 0, codecCtx->height,
                  frameDst.data, frameDst.linesize);

        SDL_UnlockYUVOverlay(sdlOverlay);

        SDL_Rect rect = {0, 0, (Uint16)codecCtx->width,
                         (Uint16)codecCtx->height};
        SDL_DisplayYUVOverlay(sdlOverlay, &rect);
      }
    }

    av_packet_unref(&packet);

    bool quit(false);

    SDL_Event event;
    SDL_PollEvent(&event);

    switch (event.type) {
    case SDL_QUIT:
      SDL_Quit();
      quit = true;
      break;

    default:
      break;
    }

    if (quit) {
      break;
    }
  }

fail:
  SDL_FreeYUVOverlay(sdlOverlay);
  av_frame_free(&frame);
  sws_freeContext(swsCtx);
  avcodec_free_context(&codecCtx);
  avformat_close_input(&formatCtx);

  return 0;
}
```

### 播放音频

实现功能：在上一个程序的基础上播放音频。

创建音频重采样上下文，用于将音频数据转换为所需的格式：

``` c++
AVChannelLayout outChLayout;
AVChannelLayout inChLayout;
av_channel_layout_default(&outChLayout, 2);
av_channel_layout_default(&inChLayout, audioCodecCtx->ch_layout.nb_channels);

// 创建音频重采样上下文
ret = swr_alloc_set_opts2(&swrContext, &outChLayout, AV_SAMPLE_FMT_S16,
                          audioCodecCtx->sample_rate, &inChLayout,
                          audioCodecCtx->sample_fmt,
                          audioCodecCtx->sample_rate, 0, nullptr);
if (ret < 0) {
  return -1;
}

swr_init(swrContext);
```

初始化 SDL Audio：

``` c++
// 初始化 SDL Audio
SDL_AudioSpec audioSpec;
audioSpec.freq = audioCodecCtx->sample_rate;
audioSpec.format = AUDIO_S16SYS;
audioSpec.channels = 2;
audioSpec.silence = 0;
audioSpec.samples = 1024;
audioSpec.callback = audioCallback;
audioSpec.userdata = audioCodecCtx;
if (SDL_OpenAudio(&audioSpec, nullptr) != 0) {
  std::cout << "error: SDL_OpenAudio() failed, " << SDL_GetError()
            << std::endl;
  goto fail;
}

SDL_PauseAudio(0);
```

处理音频 packet，放到一个队列中：

``` c++
if (packet.stream_index == videoStream) {
  // ...
} else if (packet.stream_index == audioStream) {
  AVPacket *copyPacket = av_packet_clone(&packet);
  std::unique_lock<std::mutex> lock(mutex);
  packets.push(copyPacket);
  lock.unlock();
  waitCondition.notify_all();
}
```

SDL 音频回调函数：

``` c++
// 该回调函数负责往 stream 缓冲区存放 len 字节的数据
void audioCallback(void *userdata, Uint8 *stream, int len) {
  // 存放解码出音频数据
  static uint8_t audioBuf[MAX_AUDIO_FRAME_SIZE];
  static uint32_t audioBufSize = 0;
  static uint32_t audioBufIndex = 0;

  AVCodecContext *codecCtx = (AVCodecContext *)userdata;

  // 需要存放的字节数剩余
  int lenRemain = len;
  while (lenRemain > 0) {
    if (audioBufIndex >= audioBufSize) {
      // 解码出的音频数据为空（或已消耗完毕），则执行音频解码
      int audioSize = audioDecodeFrame(codecCtx, audioBuf, sizeof(audioBuf));
      if (audioSize < 0) {
        // 如果出错，填充数据 0，即无声音
        audioBufSize = 1024;
        std::memset(audioBuf, 0, audioBufSize);
      } else {
        audioBufSize = audioSize;
      }

      audioBufIndex = 0;
    }

    // 需要拷贝的音频数据大小
    int cpSize = audioBufSize - audioBufIndex;
    if (lenRemain < cpSize) {
      cpSize = lenRemain;
    }

    // 拷贝数据，并更新缓冲区相关信息
    std::memcpy(stream, (uint8_t *)audioBuf + audioBufIndex, cpSize);
    audioBufIndex += cpSize;
    stream += cpSize;
    lenRemain -= cpSize;
  }
}
```

音频解码：

``` c++
int audioDecodeFrame(AVCodecContext *codecCtx, uint8_t *audioBuf, int bufSize) {
  int dataSize(-1);

  std::unique_lock<std::mutex> lock(mutex);
  if (packets.empty()) {
    waitCondition.wait(lock);
  }

  // 队列中取 packet
  AVPacket *packet = packets.front();
  packets.pop();
  lock.unlock();

  // packet 发送到解码器
  avcodec_send_packet(codecCtx, packet);

  // 获取解码后的帧，一个音频帧包含多个音频采样点
  AVFrame *frame = av_frame_alloc();
  while (avcodec_receive_frame(codecCtx, frame) == 0) {
    // 音频重采样，将音频数据转换为所需的格式（采样频率、声道数、样本位数）
    int out_nb_samples =
        swr_convert(swrContext, &audioBuf, frame->nb_samples,
                    (const uint8_t **)frame->data, frame->nb_samples);

    dataSize = av_samples_get_buffer_size(nullptr, 2, out_nb_samples,
                                          AV_SAMPLE_FMT_S16, 1);
  }

  av_frame_free(&frame);
  av_packet_free(&packet);
  return dataSize;
}
```

### 重构代码，使用线程

[https://gitee.com/yjwx0017/ffmpeg_tutorial/blob/master/tutorial_4/main.cpp][4]

### 同步：视频同步到音频

[https://gitee.com/yjwx0017/ffmpeg_tutorial/blob/master/tutorial_5/main.cpp][5]

维护一个音频时钟，显示视频画面时计算下一帧距当前音频时钟的时间间隔以便决定 delay 的值。

``` c++
double delay(10.);
VideoPicture vp_next = is->picture_queue.peek();
if (vp_next.bmp) {
  double diff = vp_next.pts - get_audio_clock(is);
  if (diff > 0) {
    delay = diff * 1000.;
  } else {
    delay = 5;
  }
}

if (delay < 5) {
  delay = 5;
}

schedule_refresh(is, (int)delay);
```

  [1]: https://www.ruanyifeng.com/blog/2020/01/ffmpeg.html
  [2]: http://dranger.com/ffmpeg/ffmpeg.html
  [3]: https://gitee.com/yjwx0017/ffmpeg_tutorial
  [4]: https://gitee.com/yjwx0017/ffmpeg_tutorial/blob/master/tutorial_4/main.cpp
  [5]: https://gitee.com/yjwx0017/ffmpeg_tutorial/blob/master/tutorial_5/main.cpp