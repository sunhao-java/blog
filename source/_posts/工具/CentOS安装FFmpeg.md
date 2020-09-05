title: CentOS安装FFmpeg
tags: [Docker,FFmpeg]
date: 2020-09-05
categories: 工具
---

## 设置环境变量

```shell
export FFMPEG_PACKAGE=/home/tools/packages
export FFMPEG_INSTALL=/home/tools/ffmpeg_install

export PATH=$PATH:$HOME/bin:$FFMPEG_INSTALL/bin
```

<!-- more -->

## 安装依赖

```shell
yum install -y autoconf automake bzip2 bzip2-devel cmake freetype-devel gcc gcc-c++ git libtool make mercurial pkgconfig zlib-devel
```


## libx264

```shell
cd $FFMPEG_PACKAGE
git clone https://code.videolan.org/videolan/x264.git
cd x264
PKG_CONFIG_PATH="$FFMPEG_INSTALL/ffmpeg_build/lib/pkgconfig" ./configure --prefix="$FFMPEG_INSTALL/ffmpeg_build" --bindir="$FFMPEG_INSTALL/bin" --enable-static --disable-asm
make
make install
```


## FFmpeg

```shell
cd $FFMPEG_PACKAGE
curl -O -L http://ffmpeg.org/releases/ffmpeg-4.3.1.tar.gz
tar -zxvf ffmpeg-4.3.1.tar.gz
cd ffmpeg-4.3.1
PATH="$HOME/bin:$PATH" PKG_CONFIG_PATH="$FFMPEG_INSTALL/ffmpeg_build/lib/pkgconfig" ./configure \
  --prefix="$FFMPEG_INSTALL/ffmpeg_build" \
  --pkg-config-flags="--static" \
  --extra-cflags="-I$FFMPEG_INSTALL/ffmpeg_build/include" \
  --extra-ldflags="-L$FFMPEG_INSTALL/ffmpeg_build/lib" \
  --extra-libs=-lpthread \
  --extra-libs=-lm \
  --bindir="$FFMPEG_INSTALL/bin" \
  --enable-gpl \
  --enable-libfreetype \
  --enable-libx264 \
  --enable-nonfree \
  --disable-x86asm
  
make
make install
hash -d ffmpeg  
```


## 准备字体文件

```shell
# 微软雅黑
cp msyh.ttc /home/tools/
```


## 测试是否安装成功

```shell
# 测试：
#   1. 获取视频文件信息 
#   2. 添加文字、图片水印 
#   3. 压缩前面加了水印的视频文件
# 测试文件放在/homt/tools下，命名为1.mp4
```

1. 获取视频文件信息

   ```shell
   [root@acr-2 tools]# ffprobe /home/tools/1.mp4
   ffprobe version 4.3.1 Copyright (c) 2007-2020 the FFmpeg developers
     built with gcc 4.8.5 (GCC) 20150623 (Red Hat 4.8.5-39)
     configuration: --prefix=/home/tools/ffmpeg_install/ffmpeg_build --pkg-config-flags=--static --extra-cflags=-I/home/tools/ffmpeg_install/ffmpeg_build/include --extra-ldflags=-L/home/tools/ffmpeg_install/ffmpeg_build/lib --extra-libs=-lpthread --extra-libs=-lm --bindir=/home/tools/ffmpeg_install/bin --enable-gpl --enable-libfreetype --enable-libx264 --enable-nonfree --disable-x86asm
     libavutil      56. 51.100 / 56. 51.100
     libavcodec     58. 91.100 / 58. 91.100
     libavformat    58. 45.100 / 58. 45.100
     libavdevice    58. 10.100 / 58. 10.100
     libavfilter     7. 85.100 /  7. 85.100
     libswscale      5.  7.100 /  5.  7.100
     libswresample   3.  7.100 /  3.  7.100
     libpostproc    55.  7.100 / 55.  7.100
   Input #0, mov,mp4,m4a,3gp,3g2,mj2, from '/home/tools/1.mp4':
     Metadata:
       major_brand     : mp42
       minor_version   : 1
       compatible_brands: mp41mp42isom
       creation_time   : 2017-09-28T02:49:03.000000Z
     Duration: 00:00:10.07, start: 0.000000, bitrate: 1185 kb/s
       Stream #0:0(und): Video: h264 (High) (avc1 / 0x31637661), yuv420p, 540x960, 1124 kb/s, 29.80 fps, 30 tbr, 15360 tbn, 60 tbc (default)
       Metadata:
         creation_time   : 2017-09-28T02:49:03.000000Z
         handler_name    : Core Media Video
       Stream #0:1(und): Audio: aac (LC) (mp4a / 0x6134706D), 44100 Hz, stereo, fltp, 48 kb/s (default)
       Metadata:
         creation_time   : 2017-09-28T02:49:03.000000Z
         handler_name    : Core Media Audio
   ```

2. 添加文字、图片水印

   ```shell
   [root@acr-2 tools]# ffmpeg -i /home/tools/1.mp4 -vcodec h264 -vf "[in]drawtext=x=(w-text_w)/2:y=(h-line_h)/2+14:text='ID\: 1234567890':fontfile=/home/tools/msyh.ttc:fontsize=15:fontcolor=white[text];movie=/home/tools/watermark.png[wm];[text][wm]overlay=(W-w)/2:(H-h)/2[out]" /home/tools/2.mp4
   # ...
   # ...
   # ...
   [libx264 @ 0x24e6940] consecutive B-frames: 17.5% 56.3%  8.9% 17.2%
   [libx264 @ 0x24e6940] mb I  I16..4: 13.8% 62.2% 24.0%
   [libx264 @ 0x24e6940] mb P  I16..4:  3.0%  7.4%  1.2%  P16..4: 55.5% 10.0%  4.0%  0.0%  0.0%    skip:18.9%
   [libx264 @ 0x24e6940] mb B  I16..4:  0.3%  0.5%  0.0%  B16..8: 34.7%  2.7%  0.5%  direct: 1.8%  skip:59.4%  L0:35.4% L1:55.3% BI: 9.3%
   [libx264 @ 0x24e6940] 8x8 transform intra:63.1% inter:55.8%
   [libx264 @ 0x24e6940] coded y,uvDC,uvAC intra: 45.5% 32.8% 3.8% inter: 15.5% 5.7% 0.2%
   [libx264 @ 0x24e6940] i16 v,h,dc,p: 19% 41% 15% 25%
   [libx264 @ 0x24e6940] i8 v,h,dc,ddl,ddr,vr,hd,vl,hu: 17% 24% 33%  3%  4%  4%  4%  4%  6%
   [libx264 @ 0x24e6940] i4 v,h,dc,ddl,ddr,vr,hd,vl,hu: 18% 26% 26%  5%  5%  5%  5%  5%  6%
   [libx264 @ 0x24e6940] i8c dc,h,v,p: 67% 18% 12%  2%
   [libx264 @ 0x24e6940] Weighted P-Frames: Y:10.1% UV:1.9%
   [libx264 @ 0x24e6940] ref P L0: 68.8% 19.3% 10.5%  1.3%  0.1%
   [libx264 @ 0x24e6940] ref B L0: 92.0%  7.7%  0.3%
   [libx264 @ 0x24e6940] ref B L1: 99.9%  0.1%
   [libx264 @ 0x24e6940] kb/s:1703.29
   [aac @ 0x24e5e00] Qavg: 1049.078
   # 出现以上信息并且在/home/tools目录下生成2.mp4文件就表示成功
   ```

3. 压缩前面加了水印的视频文件

   ```shell
   [root@acr-2 tools]# ffmpeg -i /home/tools/2.mp4 -vcodec h264 -vf "scale=iw*0.5:-1" -r 15  -ac 2 -ar 22050 -f mp4 -b:v 400k -y /home/tools/3.mp4
   # ...
   # ...
   # ...
   [libx264 @ 0x320a880] mb P  I16..4:  2.4%  7.0%  2.7%  P16..4: 32.1% 31.2% 19.1%  0.0%  0.0%    skip: 5.4%
   [libx264 @ 0x320a880] mb B  I16..4:  0.3%  0.4%  0.1%  B16..8: 43.4% 17.2%  4.8%  direct: 3.4%  skip:30.5%  L0:37.0% L1:46.9% BI:16.1%
   [libx264 @ 0x320a880] final ratefactor: 26.39
   [libx264 @ 0x320a880] 8x8 transform intra:53.9% inter:54.9%
   [libx264 @ 0x320a880] coded y,uvDC,uvAC intra: 58.2% 42.6% 8.4% inter: 32.0% 13.0% 0.8%
   [libx264 @ 0x320a880] i16 v,h,dc,p: 24% 33% 13% 29%
   [libx264 @ 0x320a880] i8 v,h,dc,ddl,ddr,vr,hd,vl,hu: 12% 17% 39%  4%  6%  5%  5%  5%  7%
   [libx264 @ 0x320a880] i4 v,h,dc,ddl,ddr,vr,hd,vl,hu: 14% 22% 25%  7%  6%  6%  6%  7%  7%
   [libx264 @ 0x320a880] i8c dc,h,v,p: 72% 15% 12%  2%
   [libx264 @ 0x320a880] Weighted P-Frames: Y:35.0% UV:8.8%
   [libx264 @ 0x320a880] ref P L0: 64.8% 26.5%  6.9%  1.5%  0.3%
   [libx264 @ 0x320a880] ref B L0: 96.8%  2.7%  0.5%
   [libx264 @ 0x320a880] ref B L1: 98.5%  1.5%
   [libx264 @ 0x320a880] kb/s:433.09
   [aac @ 0x31eb340] Qavg: 6954.015
   # 出现以上信息并且在/home/tools目录下生成3.mp4文件就表示成功
   ```


## 参考文献

- 官网：https://trac.ffmpeg.org/wiki/CompilationGuide/Centos