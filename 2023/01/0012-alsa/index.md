# Linux音频体系结构——ALSA


## ALSA是什么

ALSA(Advanced Linux Sound Architecture)是linux上主流的音频结构，提供了对音频和MIDI的支持，在没有出现ALSA架构之前，一直使用OSS(Open Sound System)音频架构。

> Linux 2.6 内核之前使用 OSS 音频架构，之后使用 ALSA 音频架构

### ALSA 主要特点

- 支持多种声卡设备
- 模块化的内核驱动程序
- 支持SMP(对称多处理)和多线程
- 提供应用程序开发函数库
- 兼容 OSS 有用程序

### ALSA 与 OSS 对比

以下是关于OSS和ALSA的对比：

<div align=center><img src='/pic/linux/alsa-1.jpg'/></div>
<center>OSS和ALSA对比</center>

主要的区别就是在OSS架构下，App访问底层是直接通过Sound设备节点访问的。而在ALSA音频架构下，App是通过ALSA提供的alsa-lib库访问底层硬件的操作，不再访问Sound设备节点了。这样做的好处可以简化App实现的难度。 

### ALSA 兼容 OSS

ALSA为了兼容OSS，ALSA提供了内核模块来模拟OSS声音驱动，所以在OSS架构下编写的App无需修改就可以在ALSA下运行。另外libaoos库也可以模拟OSS，无需OSS相关的内核模块。 

<div align=center><img src='/pic/linux/alsa-2.jpg'/></div>
<center>ALSA兼容OSS</center>

### ALSA 系统架构

<div align=center><img src='/pic/linux/alsa-3.jpg'/></div>
<center>ALSA系统架构</center>

User空间：主要由Alsa Libray API对应用程序提供统一的API接口，各个APP应用程序只要调用 alsa-lib 提供的 API接口来实现放音、录音、控制。现在提供了两套基本的库，tinyalsa是一个简化的alsa-lib库，现在Android的系统中主要使用它。

ALSA CORE：alsa 核心层，向上提供逻辑设备（PCM/CTL/MIDI/TIMER/…）系统调用，向下驱动硬件设备（Machine/I2S/DMA/CODEC）

ASOC Core：是 ALSA 的标准框架，是 ALSA-driver 的核心部分，提供了各种音频设备驱动的通用方法和数据结构，为 Audio driver提供 ALSA Driver API

Hardware Driver：音频硬件设备驱动，由三大部分组成，分别是 Machine、Platform、Codec，提供的 ALSA Driver API 和相应音频设备的初始化及工作流程，实现具体的功能组件，这也是驱动开发人员需要具体实现的部分。

> [https://alsa.opensrc.org/](https://alsa.opensrc.org/)

## ALSA 相关工具

alsa utils 是一组小型且强大的有用程序，旨在允许用户控制 ALSA 系统的各个部分。

- alsactl 用来保存设备设置的方法
- amixer 允许调整设备的音量和声音控制的命令行工具
- alsamixer 是 amixer 的命令行版本
- acconnect 和 aseqview 应用程序用于进行 MIDI 连接和查看连接端口列表
- aply 和 arecord 用于命令行回放和记录许多文件类型，包括 raw、wave、aiff采样率、位深等

## 如何使用 ALSA Audio 接口

> 翻译自 [http://equalarea.com/paul/alsa-audio.html](http://equalarea.com/paul/alsa-audio.html)

本文档试图提供ALSA音频API的介绍。

它不是一个完整的API参考手册，它没有涵盖许多更复杂的软件需要解决的具体问题，它仅仅试图为一个相当熟练的程序员提供足够的背景和信息，

### 理解音频接口

让我们首先回顾一下音频接口的基本设计。作为应用程序开发人员，您不需要担心这种级别的操作——它全部由设备驱动程序处理(这是ALSA提供的组件之一)。但是如果您想要编写高效且灵活的软件，您确实需要了解概念层面上的情况。

音频接口是一种允许计算机从外部世界接收和发送音频数据的设备。在计算机内部，音频数据表示为比特流。然而，音频接口可以以`模拟信号`(时变电压)或`数字信号`(一些比特流)的形式发送和接收音频。在任何一种情况下，计算机用来表示特定声音的一组位在被传送到外部世界之前都需要转换，同样，接口接收到的外部信号在对计算机有用之前也需要转换。这两种转换是音频接口存在的原因。

在音频接口中有一个称为“硬件缓冲区”的区域。当外部世界的音频信号到达时，接口将其转换为计算机可用的比特流，并将其存储在用于向计算机发送数据的部件硬件缓冲区中。当它在硬件缓冲区中收集了足够的数据时，接口中断计算机，告诉它已经为它准备好了数据。当数据从计算机发送到外部世界时，同样的过程反过来也会发生。接口中断计算机，告诉它硬件缓冲区中有空间，计算机继续在那里存储数据。接口稍后将这些位转换成所需的任何形式，以便将其传递给外部世界，并将其传递。理解接口使用这个缓冲区作为“循环缓冲区”是非常重要的。当它到达缓冲区的末尾时，它继续绕到开头。

为了使这个过程正确工作，需要配置许多变量。它们包括:
- 在计算机使用的比特流和外界使用的信号之间进行转换时，接口应该使用什么格式?
- 音频数据在接口和计算机之间传输的速率是多少?
- 当计算机设备发生中断前，应该有多少数据(或存储空间)?
- 硬件缓冲区应该有多大?

前两个问题是控制音频数据质量的基础。后两个问题会影响音频信号的“延迟”。
- 来自外部世界的数据到达音频接口，并且它对计算机可用(“输入延迟”)
- 前两个问题是控制音频数据质量的基础据由计算机传送，并被传送到外部世界(“输出延迟”)

这两点对于许多类型的音频软件都是非常重要的，尽管有些程序不需要涉及这些问题。

### 典型的音频应用程序是做什么的

一个典型的音频应用程序有这样的粗略结构:

```c
open_the_device();

set_the_parameters_of_the_device();
while (!done) {
    /* one or both of these */
    receive_audio_data_from_the_device();
    deliver_audio_data_to_the_device();
}

close the device
```

### 一个最小的后台播放程序

该程序打开一个音频接口播放，配置为立体声，16位，44.1kHz，交错常规读/写访问。然后它向它发送一组随机数据，然后退出。它代表了ALSA Audio API最简单的可能用法，并不是一个真正的程序。

```c
#include <stdio.h>
#include <stdlib.h>
#include <alsa/asoundlib.h>

int main (int argc, char *argv[])
{
    int i;
    int err;
    short buf[128];
    snd_pcm_t *playback_handle;
    snd_pcm_hw_params_t *hw_params;

    //if ((err = snd_pcm_open (&playback_handle, argv[1], SND_PCM_STREAM_PLAYBACK, 0)) < 0) {
    if ((err = snd_pcm_open (&playback_handle, "default", SND_PCM_STREAM_PLAYBACK, 0)) < 0) {
        fprintf (stderr, "cannot open audio device 'default' (%s)\n", snd_strerror (err));
        exit (1);
    }

    if ((err = snd_pcm_hw_params_malloc (&hw_params)) < 0) {
        fprintf (stderr, "cannot allocate hardware parameter structure (%s)\n", snd_strerror (err));
        exit (1);
    }

    if ((err = snd_pcm_hw_params_any (playback_handle, hw_params)) < 0) {
        fprintf (stderr, "cannot initialize hardware parameter structure (%s)\n", snd_strerror (err));
        exit (1);
    }

    if ((err = snd_pcm_hw_params_set_access (playback_handle, hw_params, SND_PCM_ACCESS_RW_INTERLEAVED)) < 0) {
        fprintf (stderr, "cannot set access type (%s)\n", snd_strerror (err));
        exit (1);
    }

    if ((err = snd_pcm_hw_params_set_format (playback_handle, hw_params, SND_PCM_FORMAT_S16_LE)) < 0) {
        fprintf (stderr, "cannot set sample format (%s)\n", snd_strerror (err));
        exit (1);
    }

    unsigned int rate = 44100;
    if ((err = snd_pcm_hw_params_set_rate_near (playback_handle, hw_params, &rate, 0)) < 0) {
        fprintf (stderr, "cannot set sample rate (%s)\n", snd_strerror (err));
        exit (1);
    }

    if ((err = snd_pcm_hw_params_set_channels (playback_handle, hw_params, 2)) < 0) {
        fprintf (stderr, "cannot set channel count (%s)\n", snd_strerror (err));
        exit (1);
    }

    if ((err = snd_pcm_hw_params (playback_handle, hw_params)) < 0) {
        fprintf (stderr, "cannot set parameters (%s)\n", snd_strerror (err));
        exit (1);
    }

    snd_pcm_hw_params_free (hw_params);

    if ((err = snd_pcm_prepare (playback_handle)) < 0) {
        fprintf (stderr, "cannot prepare audio interface for use (%s)\n", snd_strerror (err));
        exit (1);
    }

    for (i = 0; i < 10; ++i) {
        if ((err = snd_pcm_writei (playback_handle, buf, 128)) != 128) {
            fprintf (stderr, "write to audio interface failed (%s)\n", snd_strerror (err));
            exit (1);
        }
    }

    snd_pcm_close (playback_handle);

    exit (0);
}
```

下面这个程序打开一个音频接口进行捕获，配置为立体声，16位，44.1kHz，交错常规读/写访问。然后它从中读取一组随机数据，然后退出。这并不是一个真正的程序。

```c
#include <stdio.h>
#include <stdlib.h>
#include <alsa/asoundlib.h>

int main (int argc, char *argv[])
{
    int i;
    int err;
    short buf[128];
    snd_pcm_t *capture_handle;
    snd_pcm_hw_params_t *hw_params;

    if ((err = snd_pcm_open (&capture_handle, "default", SND_PCM_STREAM_CAPTURE, 0)) < 0) {
        fprintf (stderr, "cannot open audio device 'default' (%s)\n", snd_strerror (err));
        exit (1);
    }

    if ((err = snd_pcm_hw_params_malloc (&hw_params)) < 0) {
        fprintf (stderr, "cannot allocate hardware parameter structure (%s)\n", snd_strerror (err));
        exit (1);
    }

    if ((err = snd_pcm_hw_params_any (capture_handle, hw_params)) < 0) {
        fprintf (stderr, "cannot initialize hardware parameter structure (%s)\n", snd_strerror (err));
        exit (1);
    }

    if ((err = snd_pcm_hw_params_set_access (capture_handle, hw_params, SND_PCM_ACCESS_RW_INTERLEAVED)) < 0) {
        fprintf (stderr, "cannot set access type (%s)\n", snd_strerror (err));
        exit (1);
    }

    if ((err = snd_pcm_hw_params_set_format (capture_handle, hw_params, SND_PCM_FORMAT_S16_LE)) < 0) {
        fprintf (stderr, "cannot set sample format (%s)\n", snd_strerror (err));
        exit (1);
    }

    unsigned int rate = 44100;
    if ((err = snd_pcm_hw_params_set_rate_near (capture_handle, hw_params, &rate, 0)) < 0) {
        fprintf (stderr, "cannot set sample rate (%s)\n", snd_strerror (err));
        exit (1);
    }

    if ((err = snd_pcm_hw_params_set_channels (capture_handle, hw_params, 2)) < 0) {
        fprintf (stderr, "cannot set channel count (%s)\n", snd_strerror (err));
        exit (1);
    }

    if ((err = snd_pcm_hw_params (capture_handle, hw_params)) < 0) {
        fprintf (stderr, "cannot set parameters (%s)\n", snd_strerror (err));
        exit (1);
    }

    snd_pcm_hw_params_free (hw_params);

    if ((err = snd_pcm_prepare (capture_handle)) < 0) {
        fprintf (stderr, "cannot prepare audio interface for use (%s)\n", snd_strerror (err));
        exit (1);
    }

    for (i = 0; i < 10; ++i) {
        if ((err = snd_pcm_readi (capture_handle, buf, 128)) != 128) {
            fprintf (stderr, "read from audio interface failed (%s)\n", snd_strerror (err));
            exit (1);
        }
    }

    snd_pcm_close (capture_handle);

    exit (0);
}
```

下面这个程序打开一个音频接口播放，配置为立体声，16位，44.1kHz，交错常规读/写访问。然后，它等待接口准备好播放数据，并在那个时候向它传递随机数据。这种设计允许您的程序轻松地移植到依赖回调驱动机制的系统，如JACK、LADSPA、CoreAudio、VST和许多其他系统。
```c
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
#include <poll.h>
#include <alsa/asoundlib.h>

snd_pcm_t *playback_handle;
short buf[4096];

int playback_callback (snd_pcm_sframes_t nframes)
{
    int err;

    printf ("playback callback called with %u frames\n", (unsigned int)nframes);

    /* ... fill buf with data ... */
    if ((err = snd_pcm_writei (playback_handle, buf, nframes)) < 0) {
        fprintf (stderr, "write failed (%s)\n", snd_strerror (err));
    }

    return err;
}

int main (int argc, char* argv[])
{
    (void) argc;
    (void) argv;
    snd_pcm_hw_params_t *hw_params;
    snd_pcm_sw_params_t *sw_params;
    snd_pcm_sframes_t frames_to_deliver;
    int err;

    if ((err = snd_pcm_open (&playback_handle, "default", SND_PCM_STREAM_PLAYBACK, 0)) < 0) {
        fprintf (stderr, "cannot open audio device 'default' (%s)\n", snd_strerror (err));
        exit (1);
    }

    if ((err = snd_pcm_hw_params_malloc (&hw_params)) < 0) {
        fprintf (stderr, "cannot allocate hardware parameter structure (%s)\n", snd_strerror (err));
        exit (1);
    }

    if ((err = snd_pcm_hw_params_any (playback_handle, hw_params)) < 0) {
        fprintf (stderr, "cannot initialize hardware parameter structure (%s)\n", snd_strerror (err));
        exit (1);
    }

    if ((err = snd_pcm_hw_params_set_access (playback_handle, hw_params, SND_PCM_ACCESS_RW_INTERLEAVED)) < 0) {
        fprintf (stderr, "cannot set access type (%s)\n", snd_strerror (err));
        exit (1);
    }

    if ((err = snd_pcm_hw_params_set_format (playback_handle, hw_params, SND_PCM_FORMAT_S16_LE)) < 0) {
        fprintf (stderr, "cannot set sample format (%s)\n", snd_strerror (err));
        exit (1);
    }

    unsigned int rate = 44100;
    if ((err = snd_pcm_hw_params_set_rate_near (playback_handle, hw_params, &rate, 0)) < 0) {
        fprintf (stderr, "cannot set sample rate (%s)\n", snd_strerror (err));
        exit (1);
    }

    if ((err = snd_pcm_hw_params_set_channels (playback_handle, hw_params, 2)) < 0) {
        fprintf (stderr, "cannot set channel count (%s)\n", snd_strerror (err));
        exit (1);
    }

    if ((err = snd_pcm_hw_params (playback_handle, hw_params)) < 0) {
        fprintf (stderr, "cannot set parameters (%s)\n", snd_strerror (err));
        exit (1);
    }

    snd_pcm_hw_params_free (hw_params);

    /* tell ALSA to wake us up whenever 4096 or more frames of playback data can be delivered. Also, tell ALSA that we'll start the device ourselves. */
    if ((err = snd_pcm_sw_params_malloc (&sw_params)) < 0) {
        fprintf (stderr, "cannot allocate software parameters structure (%s)\n", snd_strerror (err));
        exit (1);
    }
    if ((err = snd_pcm_sw_params_current (playback_handle, sw_params)) < 0) {
        fprintf (stderr, "cannot initialize software parameters structure (%s)\n", snd_strerror (err));
        exit (1);
    }
    if ((err = snd_pcm_sw_params_set_avail_min (playback_handle, sw_params, 4096)) < 0) {
        fprintf (stderr, "cannot set minimum available count (%s)\n", snd_strerror (err));
        exit (1);
    }
    if ((err = snd_pcm_sw_params_set_start_threshold (playback_handle, sw_params, 0U)) < 0) {
        fprintf (stderr, "cannot set start mode (%s)\n", snd_strerror (err));
        exit (1);
    }
    if ((err = snd_pcm_sw_params (playback_handle, sw_params)) < 0) {
        fprintf (stderr, "cannot set software parameters (%s)\n", snd_strerror (err));
        exit (1);
    }

    /* the interface will interrupt the kernel every 4096 frames, and ALSA will wake up this program very soon after that. */
    if ((err = snd_pcm_prepare (playback_handle)) < 0) {
        fprintf (stderr, "cannot prepare audio interface for use (%s)\n", snd_strerror (err));
        exit (1);
    }

    while (1) {
        /* wait till the interface is ready for data, or 1 second has elapsed. */
        if ((err = snd_pcm_wait (playback_handle, 1000)) < 0) {
            fprintf (stderr, "poll failed (%s)\n", strerror (errno));
            break;
        }	           

        /* find out how much space is available for playback data */
        if ((frames_to_deliver = snd_pcm_avail_update (playback_handle)) < 0) {
            if (frames_to_deliver == -EPIPE) {
                fprintf (stderr, "an xrun occured\n");
                break;
            }
            else {
                fprintf (stderr, "unknown ALSA avail update return value (%d)\n", (int)frames_to_deliver);
                break;
            }
        }

        frames_to_deliver = frames_to_deliver > 4096 ? 4096 : frames_to_deliver;

        /* deliver the data */
        if (playback_callback (frames_to_deliver) != frames_to_deliver) {
            fprintf (stderr, "playback callback failed\n");
            break;
        }
    }

    snd_pcm_close (playback_handle);

    exit (0);
}
```

### 一些术语

|术语|解释|
|----|----|
|capture|从外部世界接收数据(与“记录”不同，“记录”意味着将数据存储在某个地方，并且不是ALSA API的一部分)|
|playback|向外部世界传递数据(尽管不一定)，以便能够被听到。|
|duplex|捕获和回放同时发生在同一界面上的一种情况。|
|xrun|一旦音频接口开始运行，它就会继续运行，直到被告知停止。它将生成数据供计算机使用和/或将数据从计算机发送到外部世界。由于各种原因，您的程序可能跟不上它。对于回放，这可能导致接口需要来自计算机的新数据，但它不存在，迫使它使用遗留在硬件缓冲区中的旧数据。这就是所谓的“欠压”。为了捕获，接口可能有数据要传递给计算机，但没有地方存储这些数据，因此它必须覆盖硬件缓冲区中包含计算机尚未接收到的数据的部分。这就是所谓的“泛滥”。为简单起见，我们使用通用术语“xrun”来指代这两种情况|
|PCM|脉冲编码调制。这个短语(和首字母缩写词)描述了用数字形式表示模拟信号的一种方法。它是几乎被计算机音频接口使用的方法，在ALSA API中用作“音频”的缩写。|
|channel||
|frame|采样是描述音频信号在单个声道上的单个时间点的振幅的单个值。当我们谈论使用数字音频时，我们经常想要谈论在单个时间点上表示所有频道的数据。这是一个样本的集合，每个通道一个，通常称为“帧”。当我们以框架的形式讨论时间的流逝时，它大致相当于人们以样本的形式测量的时间，但更准确;更重要的是，当我们讨论在一个时间点上表示所有通道所需的数据量时，它是唯一有意义的单位。几乎每个ALSA Audio API函数都使用帧作为数据量的度量单位。|
|interleaved|一种数据布局安排，其中将在同一时间播放的每个通道的样本按顺序相互跟随。参见“non-interleaved”|
|non-interleaved|一种数据布局，其中单个通道的样本按顺序相互跟随;另一个通道的样本要么在另一个缓冲区中，要么在该缓冲区的另一部分中。与"interleaves"形成对比|
|sample clock|一种计时源，用于标记样品应该发送/接收到外部世界的时间。有些音频接口允许你使用外部采样时钟，要么是“文字时钟”信号(通常用于许多工作室)，要么是“自动同步”(使用传入数字数据中的时钟信号)。所有音频接口都至少有一个样本时钟源，该时钟源存在于接口本身，通常是一个小晶体时钟。有些接口不允许改变时钟的速率，有些接口的时钟实际上并不能精确地以您期望的速率运行(44.1kHz等)。没有两个样本时钟可以被期望以完全相同的速率运行——如果你需要两个样本流保持彼此同步，它们必须从同一个样本时钟运行。|

### API使用流程
1. 打开设备
2. 设置参数
3. 硬件参数
    - 采样率：如果接口有模拟I/O，则控制A/D/D/A转换的速率。对于全数字接口，它控制用于将数字音频数据移动到/从外部世界的时钟的速度。在某些接口上，其他特定于设备的配置可能意味着您的程序不能控制这个值(例如，当接口被告知使用外部字时钟源来确定采样率时)。
    - 格式：它控制用于向接口传输数据和从接口传输数据的格式。它可能与硬件直接支持的格式对应，也可能不对应。
    - 通道数：
    - 数据访问与布局：这控制程序从接口传递/接收数据的方式。有两个参数由4个可能的设置控制。其中一个参数是是否使用“读/写”模型，即使用显式函数调用来传输数据。这里的另一个选项是使用“mmap模式”，在这种模式下，数据是通过在内存区域之间复制来传输的，API调用只需要注意它开始和结束的时间。另一个参数是数据布局是交错的还是非交错的。
    - 中断时间间隔：这决定了接口每次对其硬件缓冲区的完整遍历将产生多少中断。它可以通过指定周期大小的一些周期来设置。因为这决定了在接口中断计算机之前必须积累的空间/数据帧数。它是控制延迟的核心。
    - 缓冲区大小：这决定了硬件缓冲区的大小。它可以以时间或帧为单位指定。
4. 软件参数(这些参数控制设备驱动程序的操作，而不是硬件本身。大多数使用ALSA音频API的程序不需要设置这些;少数将需要设置它们的子集。)
    - 何时启动设备：当您打开音频接口时，ALSA确保它不是活动的-没有数据被移动到或从它的外部连接器。可以推测，在某个时刻，您希望数据传输开始。有几种方法可以实现这一目标。这里的控制点是开始阈值，它定义了自动启动设备所需的空间/数据帧数。如果设置为播放的某个值而不是0，则有必要在设备启动之前预填充播放缓冲区。如果设置为0，第一次写入设备的数据(或第一次尝试从捕获流读取数据)将启动设备。您还可以显式地使用`snd_pcm_start`启动设备，但是在播放流的情况下，这需要缓冲区预填充。如果尝试不这样做就启动流，则会得到`-EPIPE`作为返回代码，表示没有数据等待传递到回放硬件缓冲区。
    - 如何处理 xruns：如果发生了xrun，设备驱动程序可能会根据请求采取某些步骤来处理它。选项包括停止设备，或静音用于回放的全部或部分硬件缓冲区。
        - 停止阈值：如果可用的数据/空间帧数达到或超过这个值，驱动程序将停止接口。
        - 沉默阈值：如果一个回放流的可用空间帧数达到或超过这个值，驱动程序将用静默填充部分回放硬件缓冲区。
        - 沉默值大小：当满足静默阈值水平时，这将决定有多少静默帧被写入回放硬件缓冲区
    - 可用于唤醒的最小空间/数据帧数：使用poll(2)或select(2)来确定音频数据何时可以传输到接口或从接口传输的程序可以将此设置为控制相对于硬件缓冲区的状态，它们希望在哪个点被唤醒。
    - 传输块大小：这决定了从设备硬件缓冲区传输数据时使用的帧数。

还有一些其他的软件参数，但在这里不需要考虑。

## ALSA API

> ALSA 提供了 Kernel API 和 库API，本文当描述了库 API 以及它如何与内核 API 交互。

应用程序程序员应该使用库API而不是内核API。该库提供了内核API的100%功能，但在可用性方面进行了重大改进，使应用程序代码更简单、更美观。此外，将来的修复程序或兼容性代码可能会放在库代码中，而不是放在内核驱动程序中。

### API 链接

- [页面控件接口解释了基本控件API](https://www.alsa-project.org/alsa-doc/alsa-lib/control.html)
- [基本控件插件解释了基本控件插件的设计](https://www.alsa-project.org/alsa-doc/alsa-lib/control_plugins.html)
- [高级控件接口解释了高级原语控件API](https://www.alsa-project.org/alsa-doc/alsa-lib/hcontrol.html)
- [页面混合器接口解释了混合器控件API](https://www.alsa-project.org/alsa-doc/alsa-lib/mixer.html)
- [PCM(数字音频)接口页面解释了PCM(数字音频)API的设计](https://www.alsa-project.org/alsa-doc/alsa-lib/pcm.html)
- [PCM(数字音频)插件页面解释了PCM(数字音频)插件的设计](https://www.alsa-project.org/alsa-doc/alsa-lib/pcm_plugins.html)
- [Page PCM External Plugin SDK介绍了外部PCM插件SDK](https://www.alsa-project.org/alsa-doc/alsa-lib/pcm_external_plugins.html)
- [Page ctl_external_plugins解释了外部控制插件SDK]()
- [页面RawMidi接口说明了RawMidi API的设计](https://www.alsa-project.org/alsa-doc/alsa-lib/rawmidi.html)
- [Page Timer接口解释了Timer API的设计](https://www.alsa-project.org/alsa-doc/alsa-lib/timer.html)
- [Page Sequencer界面解释了Sequencer API的设计](https://www.alsa-project.org/alsa-doc/alsa-lib/seq.html)
- [页面用例接口解释用例API](https://www.alsa-project.org/alsa-doc/alsa-lib/group__ucm.html)
- [Page ALSA Topology Interface解释了DSP拓扑API](https://www.alsa-project.org/alsa-doc/alsa-lib/group__topology.html)

### 配置

- [Page Configuration文件解释了库配置的语法](https://www.alsa-project.org/alsa-doc/alsa-lib/conf.html)
- [配置文件中的Page运行时参数解释运行时参数语法](https://www.alsa-project.org/alsa-doc/alsa-lib/confarg.html)
- [配置文件中的运行时函数页解释了运行时函数的定义及其用法](https://www.alsa-project.org/alsa-doc/alsa-lib/conffunc.html)
- [配置文件中的Page hook解释了运行时钩子的定义及其用法](https://www.alsa-project.org/alsa-doc/alsa-lib/confhooks.html)
- [页面用例配置解释了UCM配置和它们的用法](https://www.alsa-project.org/alsa-doc/alsa-lib/group__ucm__conf.html)



