```
Player API

FFmpeg Opengles Opensles
yuv420p + nv21 + nv12 + rgb
new FFPlayer();
fun onSource(url: String)
fun onSeekTo(percent: Int)
fun onPlay()
fun onPause() 
fun onStop() 
fun onRelease()
```

![Image text](https://github.com/ABCDQ123/ffmpegtest/blob/main/lib_ffmpeg/src/main/image/player.gif)

```
FFMPEG编译流程

Ⅰ提前编译动态库.so(把要用到的功能编译好，附带编译指定平台，方便后期使用时不用再编译并减少库的大小)

1.windows平台 + mingw(自带的msys)
  配置mingw + msys环境变量(mingw/bin + mingw/mingw32/bin +mingw/msys/bin)

2.修改FFmpeg的configure
  下载FFmpeg源代码之后，首先需要对源代码中的configure文件进行修改。
  由于编译出来的动态库文件名的版本号在.so之后（例如“libavcodec.so.5.100.1”），而android平台不能识别这样文件名，所以需要修改这种文件名。

  在configure文件中找到下面几行代码：
  SLIBNAME_WITH_MAJOR='$(SLIBNAME).$(LIBMAJOR)'
  LIB_INSTALL_EXTRA_CMD='$$(RANLIB)"$(LIBDIR)/$(LIBNAME)"'
  SLIB_INSTALL_NAME='$(SLIBNAME_WITH_VERSION)'
  SLIB_INSTALL_LINKS='$(SLIBNAME_WITH_MAJOR)$(SLIBNAME)'
  替换为下面内容就可以了：
  SLIBNAME_WITH_MAJOR='$(SLIBPREF)$(FULLNAME)-$(LIBMAJOR)$(SLIBSUF)'
  LIB_INSTALL_EXTRA_CMD='$$(RANLIB)"$(LIBDIR)/$(LIBNAME)"'
  SLIB_INSTALL_NAME='$(SLIBNAME_WITH_MAJOR)'
  SLIB_INSTALL_LINKS='$(SLIBNAME)'

3.生成类库 (编写.sh文件,通过mingw/msys执行编译)
NDK= ??????????
function build_android{
  ./configure \
  --enable-neon \
  --enable-hwaccels \
  --enable-gpl \
  --disable-postproc \
  --disable-debug \
  --enable-small \
  --enable-jni \
  --enable-mediacodec \
  --enable-decoder=h264_mediacodec \
  --disable-static \
  --enable-shared \
  --disable-doc \
  --enable-ffmpeg \
  --enable-openssl \
  --disable-ffplay \
  --disable-ffprobe \
  --disable-avdevice \
  --disable-symver \
  --enable-cross-compile \
  --target-os=android \
  --sysroot=$SYSROOT \
  --arch=$ARCH \
  --cpu=$CPU \
  --cc=$CC \
  --cxx=$CXX \
  --prefix=$PREFIX \
  --cross-prefix=$CROSS_PREFIX \
  --extra-cflags="-Os -fpic $OPTIMIZE_CFLAGS" \
  --extra-ldflags="$ADDI_LDFLAGS"
  make clean
  make
  make install
}

#arm64-v8a
PLATFORM=windows-x86_64
ARCH=arm64
CPU=armv8-a
CPP=aarch64-linux-android
API=21
# 工具链
TOOLCHAIN=$NDK/toolchains/llvm/prebuilt/$PLATFORM
CC=$TOOLCHAIN/bin/$CPP$API-clang
CXX=$TOOLCHAIN/bin/$CPP$API-clang++
# 编译针对的平台
SYSROOT=$TOOLCHAIN/sysroot
#交叉编译前缀
CROSS_PREFIX=$TOOLCHAIN/bin/$CPP-
#输出路径
PREFIX=$(pwd)/android/$CPU
build_android

#armv7-a
PLATFORM=windows-x86_64
ARCH=arm
CPU=armv7-a
CPP=armv7a-linux-androideabi
API=21
TOOLCHAIN=$NDK/toolchains/llvm/prebuilt/$PLATFORM
CC=$TOOLCHAIN/bin/$CPP$API-clang
CXX=$TOOLCHAIN/bin/$CPP$API-clang++
SYSROOT=$TOOLCHAIN/sysroot
CROSS_PREFIX=$TOOLCHAIN/bin/$CPP-
PREFIX=$(pwd)/android/$CPU
OPTIMIZE_CFLAGS="-mfloat-abi=softfp -mfpu=vfp -marm -march=$CPU "
build_android

6个常用功能模块
libavformat:多媒体文件或协议的封装和解封装库
libavcodec：音视频编解码库
libavfilter：音视频、字幕滤镜库
libswscale：图像格式转换库
libswresample：音频重采样库
libavutil：工具库
6常用结构体
AVFormatContext：解封装 上下文: 文件名、音视频流、时长、比特率等
AVCodecContext：编解码器 上下文: 编解码时用到的结构体、编解码器类型、视频宽高、音频通道数和采样率等
AVCodec：存储 编解码器 信息
AVStream：存储音视频 流 信息
AVPacket：存储音视频 编码 数据
AVFrame：存储音视频 解码 数据（原始数据）
```

```
图像基础

像素：
2x2分辨率 -> 2x2个像素
RGB RGB
RGB RGB

位深：
1bit    -> 2种颜色，黑白两色
3bit    -> 红1bit 蓝1bit 绿1bit
8bit    -> 红3bit 蓝3bit 绿3bit
16bit   -> 红5bit 蓝5bit 绿6bit
24bit   -> 红8bit 蓝8bit 绿8bit
颜色数量 -> 2∧位深 -> 2∧1/2∧3/2∧8/2∧16/2∧24
   
图片格式：
JPG(JPEG) -> 有损压缩 -> 24bit
PNG -> 无损压缩 -> 8/24/32bit
GIF -> 无损压缩 -> 8bit

RBG：
红 + 蓝 + 绿
R + B + G

YUV(YCbCr)：
亮度 + 蓝色色度 + 红色色度
420采样 -> Y + UV/4
422采样 -> Y + UV/2
444采样 -> Y + UV
yuv420sp888(420) -> yyyy + u + v
NV12/NV21(420) -> yyyy + vu / yyyy + uv

转换公式：
RGB的取值范围是[0, 1]
Y的取值范围是[0, 1]
UV的取值范围是[-0.5, 0.5]
Y = 0.299R + 0.587G + 0.114B
U = 0.564(B - Y) = -0.169R - 0.331G + 0.500B
V = 0.713(R - Y) = 0.500R - 0.419G - 0.081B
R = Y + 1.403V
G = Y - 0.344U - 0.714V
B = Y + 1.770U
RGB的取值范围是[0, 255]
YUV的取值范围是[0, 255]
Y = 0.299R + 0.587G + 0.114B
U = -0.169R - 0.331G + 0.500B + 128
V = 0.500R - 0.419G - 0.081B + 128
R = Y + 1.403(V - 128)
G = Y - 0.343(U - 128) - 0.714(V - 128)
B = Y + 1.770(U - 128)
RGB的取值范围是[0,255]
Y的取值范围是[16,235]
UV的取值范围是[16,239]
Y = 0.257R + 0.504G + 0.098B + 16
U = -0.148R - 0.291G + 0.439B + 128
V = 0.439R - 0.368G - 0.071B + 128
R = 1.164(Y - 16) + 2.018(U - 128)
G = 1.164(Y - 16) - 0.813(V - 128) - 0.391(U - 128)
B = 1.164(Y - 16) + 1.596(V - 128)
```
```
视频编码

常见编码
JPEG + MPEG + MPEG-4 + H.263 + H.264/AVC + H.265/HEVC

H264
别称 MPEG-4 Part 10 Advanced Video Coding
简称 MPEG-4 AVC
原始视频 10秒+30fps+1920x1080+YUV420P -> (10*30)*(1920*1080)*1.5 = 933120000byte ≈ 889.89MB
H264压缩编码 -> ≈ 889.89MB / 10 ≈ 88M

编码
x264是一款免费的高性能的H.264开源编码器 x264编码器在FFmpeg中的名称是libx264
AVCodec *encodec = avcodec_find_encoder_by_name("libx264");
解码
FFmpeg默认已经内置了一个H.264的解码器 名称是h264
AVCodec *decodec = avcodec_find_decoder_by_name("h264");

编码流程
划分帧类型 -> 帧内/帧间编码 - > 变换 + 量化 -> 滤波 -> 熵编码
在连续的几帧图像中 一般只有10%以内的像素有差别 亮度的差值变化不超过2% 而色度的差值变化只在1%以内

GOP(视频由N个GOP组成)
图像群组 -> I + B + B + B + P + B + P
I帧 -> 关键帧(帧内编码)
    是视频的第一帧 也是GOP的第一帧 一个GOP只有一个I帧
    对整帧图像数据进行编码 可以理解为一张静态图像
P帧 -> 预测编码(帧间编码)
    以前方I帧/P帧作为参考帧 只编码与参考帧的差异数据
B帧 -> 前后预测编码(帧间编码)
    同时以前后的I帧/P帧作为参考帧，只编码与前后参考帧的差异数据
帧占用         -> 编码后的数据大小：I帧 > P帧 > B帧
GOP长度        -> 越长有利于减小视频文件大小 太长会导致GOP后部帧的画面失真 影响视频质量
Open GOP      -> B帧可以参考下一个GOP的I帧
Close GOP     -> B帧不能参考下一个GOP的I帧 GOP不能以B帧结尾
特殊I帧(IDR帧) -> 会清空参考帧队列 不再参考该IDR之前的帧 使错误不继续往下传播

宏块
完整的帧拆分多个 -> 宏块(Macroblock) H.264中的宏块大小通常是16x16
进一步拆分为 -> 变换块(Transform blocks) + 预测块(Prediction blocks)
变换块尺寸 -> 16x16 8x8 4x4
预测块尺寸 -> 16×16 16×8 8×16 8×8 8×4 4×8 4×4

变换 + 量化
对残差值进行DCT变换(Discrete Cosine Transform 离散余弦变换)
```
```
音频基本属性

采样率(sample rate)
通常人耳能听到频率范围大约在20Hz～20kHz之间的声音 保证声音不失真采样频率应在40kHz以上 44100是最常见的采样率

声道数(channels)
声音录制时的音源数量 常见单声道 双声道

位深度(bit depth)
n个二进制位(bit)来存储每个幅度值，总共可以表示的数值数量为2的n次方-1
16位的PCM 采样值是介于-32768到+32767

比特率(bit rate)
数据流中每秒钟能通过的信息量
Bit bitRate = samples rate * channels * bit depth
Size in bitRate = samples rate * channels * bit depth * length of time

音频帧(frame)
音频数据是流式的 本身没有明确的一帧帧的概念 一般约定俗成取2.5ms~60ms为单位的数据量为一帧音频

Packet
Packet 是一个或者多个连续 frame 的集合
```