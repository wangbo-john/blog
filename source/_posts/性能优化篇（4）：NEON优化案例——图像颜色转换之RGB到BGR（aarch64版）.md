# 性能优化篇（4）：NEON优化案例——图像颜色转换之RGB到BGR（aarch64版）

Author:stormQ

Sunday, 1. December 2019 1 07:13AM

- 目录
  - [为什么要交换图像颜色通道](https://blog.csdn.net/wohenfanjian/article/details/103407259#jump为什么要交换图像颜色通道)
  - [实现一个基本的图像颜色通道交换函数](https://blog.csdn.net/wohenfanjian/article/details/103407259#jump实现一个基本的图像颜色通道交换函数)
  - [利用 NEON 加速图像颜色通道交换](https://blog.csdn.net/wohenfanjian/article/details/103407259#jump利用NEON加速图像颜色通道交换)

------

### 为什么要交换图像颜色通道

对于只有`R`、`G`、`B`三种通道的图像来说，OpenCV默认的通道排列方式为`BGR`而非常见的`RGB`。在一个应用内，如果同时存在“有些图像处理函数以`RGB`图像作为输入输出，而有些图像处理函数以`BGR`图像作为输入输出”，那么在调用这些函数前需要进行图像颜色通道转换，以满足接口的输入要求。由于图像颜色通道转换比较耗时，甚至可能成为性能瓶颈。因此，研究如何加速图像颜色通道交换就非常有意义了。

------

### 实现一个基本的图像颜色通道交换函数

如何将一个大小为`1920x1080`的`RGB`图像转换成相同大小的`BGR`图像呢？

首先实现一个最简单的转换函数`simple_rgb2bgr()`，完整实现如下：

```
#define CHANNELS 3

void simple_rgb2bgr(uint8_t *img, int height, int width)
{
    const int total_bytes = height * width * CHANNELS;
    for (int i = 0; i < total_bytes; i += CHANNELS)
    {
        // swap R and B channel
        const auto r_channel = *(img + i);
        *(img + i) = *(img + i + 2);
        *(img + i + 2) = r_channel;
    }
}
```

`simple_rgb2bgr()`函数将一个二维的图像看作是一维的，意味着只需要一个循环就可以遍历所有的像素。这种用法的好处在于，遵循空间局部性原理以高效利用`cache`从而改善程序性能，而不必关心二维图像在内存中的存储顺序——是行主序还是列主序。如果用两个循环的话会比较繁琐，移植性也不好。

上述函数在`Jetson TX2`上运行时会耗时 3~4ms 左右，在其他机器上运行时耗时会有所不同。如果该耗时不可接受，那么只能想办法对其进行加速。常见的加速思路有以下几种：1）任务级并行（利用多线程）；2）数据级并行（利用 GPU）；3）利用 SIMD。这里我们只研究第三种加速方式——利用 SIMD，它没有“任务级并行所需要的正确的调度协作和占用更多的 CPU 资源”，也没有“数据级并行所带来的 GPU 与 CPU 之间数据传输开销”。

------

### 利用 NEON 加速图像颜色通道交换

首先，实现一个基本的利用`NEON`加速图像颜色通道交换的函数`rgb2bgr_with_neon()`，完整实现如下：

```
#include "arm_neon.h"

void rgb2bgr_with_neon(uint8_t *img, int height, int width)
{
    const int total_bytes = height * width * CHANNELS;
    const int stride_bytes = 48;

    for (int i = 0; i < total_bytes; i += stride_bytes)
    {
        uint8_t *target = img + i;

        // swap R and B channel with NEON
        uint8x16x3_t a = vld3q_u8(target);
        uint8x16x3_t b;
        b.val[0] = a.val[2];
        b.val[1] = a.val[1];
        b.val[2] = a.val[0];
        vst3q_u8(target, b);
    }
}
```

上述函数没有考虑图像像素个数非整除时的情形，这个放到后面进行。该函数的实现用到了两个`NEON Intrinsics`，分别是`vld3q_u8`和`vst3q_u8`，两者都定义在`arm_neon.h`文件中。

`vld3q_u8`的函数原型为`uint8x16x3_t vld3q_u8 (const uint8_t * __a)`，作用为以步长为 3 交叉地加载数据到三个连续的 128-bit 的向量寄存器。具体地：将内存地址`__a`、`__a+3`、…、`__a+45`处的内容分别赋值给向量寄存器`Vn`的`lane[0]`、`lane[1]`、…、`lane[15]`，将内存地址`__a+1`、`__a+4`、…、`__a+46`处的内容分别赋值给向量寄存器`Vn+1`的`lane[0]`、`lane[1]`、…、`lane[15]`，将内存地址`__a+2`、`__a+5`、…、`__a+47`处的内容分别赋值给向量寄存器`Vn+2`的`lane[0]`、`lane[1]`、…、`lane[15]`。也就是说，`vld3q_u8`在上述函数中的作用为：将连续 16 个像素的 R 通道： R0、R1、… 、R15 加载到向量寄存器`Vn`，将连续 16 个像素的 G 通道： G0、G1、… 、G15 加载到向量寄存器`Vn+1`，将连续 16 个像素的 B 通道： B0、B1、… 、B15 加载到向量寄存器`Vn+2`。

`vst3q_u8`的函数原型为`void vst3q_u8 (uint8_t * __a, uint8x16x3_t val)`，作用为以步长为 3 交叉地存储数据到内存中。

由于每一次迭代可以同时处理 16 个连续的像素，这些像素占用 48 字节（48（bytes） = 16（像素个数） * 3（通道数））。所以，变量`stride_bytes`的值为 48。

由于`aarch64`指令集中没有`ARM` 指令集中的`VSWP`指令。因此，这里引入一个类型为`uint8x16x3_t`的临时向量`b`用于交换`R`和`B`通道。 g++ 带优化选项`-Og`编译时，语句`b.val[0] = a.val[2]; b.val[1] = a.val[1]; b.val[2] = a.val[0];`对应三条`MOV`指令。

在通道交换完成后，用语句`vst3q_u8(target, b);`将交换后的结果写入内存。这样，一次交换操作就完成了。

现在考虑图像像素个数非整除时的情形，完整实现如下：

```
void rgb2bgr_with_neon(uint8_t *img, int height, int width)
{
    const int total_bytes = height * width * CHANNELS;
    const int stride_bytes = 48;
    const int left_bytes = total_bytes % stride_bytes;
    const int multiply_bytes = total_bytes - left_bytes;

    for (int i = 0; i < multiply_bytes; i += stride_bytes)
    {
        uint8_t *target = img + i;

        // swap R and B channel with NEON
        uint8x16x3_t a = vld3q_u8(target);
        uint8x16x3_t b;
        b.val[0] = a.val[2];
        b.val[1] = a.val[1];
        b.val[2] = a.val[0];
        vst3q_u8(target, b);
    }

    // handling non-multiply array lengths
    for (int i = multiply_bytes; i < total_bytes; i += CHANNELS)
    {
        const auto r_channel = *(img + i);
        *(img + i) = *(img + i + 2);
        *(img + i + 2) = r_channel;
    }
}
```

为了验证`rgb2bgr_with_neon()`函数的正确性，这里引入两个函数：`init()`和`check()`。`init()`函数用于初始化图像，`check()`函数用于检查交换后的图像是否正确。完整实现如下：

```
#include <iostream>

#define RED 255
#define GREEN 125
#define BLUE 80

#define CHECK(img, height, width)               \
if (check(img, height, width))                  \
{                                               \
    std::cout << "Correct:img.\n";              \
}                                               \
else                                            \
{                                               \
    std::cout << "Error:img.\n";                \
}

void init(uint8_t *img, int height, int width)
{
    const int total_bytes = height * width * CHANNELS;
    for (int i = 0; i < total_bytes; )
    {
        *(img + i++) = RED;
        *(img + i++) = GREEN;
        *(img + i++) = BLUE;
    }
}

bool check(const uint8_t *img, int height, int width)
{
    const int total_bytes = height * width * CHANNELS;
    for (int i = 0; i < total_bytes; i += CHANNELS)
    {
        if (*(img + i) != BLUE || *(img + i + 1) != GREEN || 
                *(img + i + 2) != RED)
        {
            return false;
        }
    }
    return true;
}
```

完整的程序为`main.cpp`：

```
#include "arm_neon.h"
#include <iostream>

#define HEIGHT 1080
#define WIDTH 1920
#define CHANNELS 3
#define RED 255
#define GREEN 125
#define BLUE 80

#define CHECK(img, height, width)               \
if (check(img, height, width))                  \
{                                               \
    std::cout << "Correct:img.\n";              \
}                                               \
else                                            \
{                                               \
    std::cout << "Error:img.\n";                \
}

void init(uint8_t *img, int height, int width)
{
    const int total_bytes = height * width * CHANNELS;
    for (int i = 0; i < total_bytes; )
    {
        *(img + i++) = RED;
        *(img + i++) = GREEN;
        *(img + i++) = BLUE;
    }
}

bool check(const uint8_t *img, int height, int width)
{
    const int total_bytes = height * width * CHANNELS;
    for (int i = 0; i < total_bytes; i += CHANNELS)
    {
        if (*(img + i) != BLUE || *(img + i + 1) != GREEN || 
                *(img + i + 2) != RED)
        {
            return false;
        }
    }
    return true;
}

void simple_rgb2bgr(uint8_t *img, int height, int width)
{
    const int total_bytes = height * width * CHANNELS;
    for (int i = 0; i < total_bytes; i += CHANNELS)
    {
        // swap R and B channel
        const auto r_channel = *(img + i);
        *(img + i) = *(img + i + 2);
        *(img + i + 2) = r_channel;
    }
}

void rgb2bgr_with_neon(uint8_t *img, int height, int width)
{
    const int total_bytes = height * width * CHANNELS;
    const int stride_bytes = 48;
    const int left_bytes = total_bytes % stride_bytes;
    const int multiply_bytes = total_bytes - left_bytes;

    for (int i = 0; i < multiply_bytes; i += stride_bytes)
    {
        uint8_t *target = img + i;

        // swap R and B channel with NEON
        uint8x16x3_t a = vld3q_u8(target);
        uint8x16x3_t b;
        b.val[0] = a.val[2];
        b.val[1] = a.val[1];
        b.val[2] = a.val[0];
        vst3q_u8(target, b);
    }

    // handling non-multiply array lengths
    for (int i = multiply_bytes; i < total_bytes; i += CHANNELS)
    {
        const auto r_channel = *(img + i);
        *(img + i) = *(img + i + 2);
        *(img + i + 2) = r_channel;
    }
}

uint8_t *rgb_img1 = nullptr;
uint8_t *rgb_img2 = nullptr;

int main()
{
    rgb_img1 = new uint8_t[HEIGHT * WIDTH * CHANNELS];
    rgb_img2 = new uint8_t[HEIGHT * WIDTH * CHANNELS];

    init(rgb_img1, HEIGHT, WIDTH);
    init(rgb_img2, HEIGHT, WIDTH);

    simple_rgb2bgr(rgb_img1, HEIGHT, WIDTH);
    rgb2bgr_with_neon(rgb_img2, HEIGHT, WIDTH);

    CHECK(rgb_img1, HEIGHT, WIDTH);
    CHECK(rgb_img2, HEIGHT, WIDTH);
    
    delete [] rgb_img2;
    delete [] rgb_img1;
    return 0;
}
```

编译程序（On Jetson TX2）：

```
g++ -std=c++11 -g -Og -o main_Og main.cpp
```

统计函数耗时（On Jetson TX2）：

| 启动程序方式 | 第一次执行耗时(us)                        | 第二次执行耗时(us)                        | 第三次执行耗时(us)                        | 第四次执行耗时(us)                        | 第五次执行耗时(us)                        |
| ------------ | ----------------------------------------- | ----------------------------------------- | ----------------------------------------- | ----------------------------------------- | ----------------------------------------- |
| ./main_Og    | simple_rgb2bgr:4303rgb2bgr_with_neon:1021 | simple_rgb2bgr:4469rgb2bgr_with_neon:1162 | simple_rgb2bgr:3920rgb2bgr_with_neon:1017 | simple_rgb2bgr:4248rgb2bgr_with_neon:1019 | simple_rgb2bgr:3966rgb2bgr_with_neon:1018 |

从统计结果中可以看出，`rgb2bgr_with_neon()`函数的执行速度比`simple_rgb2bgr()`函数快 3 到 4 倍。

**那么 NEON 版的实现为什么能达到这样一个加速呢？** 下面从 cache 的角度进行分析。

统计 cache 性能数据：

```
--------------------------------------------------------------------------------
I1 cache:         16384 B, 64 B, 4-way associative
D1 cache:         16384 B, 64 B, 4-way associative
LL cache:         262144 B, 64 B, 8-way associative
Command:          ./main_Og
Data file:        cachegrind.out.22890
Events recorded:  Ir I1mr ILmr Dr D1mr DLmr Dw D1mw DLmw
Events shown:     Ir I1mr ILmr Dr D1mr DLmr Dw D1mw DLmw
Event sort order: Ir I1mr ILmr Dr D1mr DLmr Dw D1mw DLmw
Thresholds:       0.1 100 100 100 100 100 100 100 100
Include dirs:     
User annotated:   main.cpp
Auto-annotation:  on

--------------------------------------------------------------------------------
         Ir  I1mr  ILmr         Dr    D1mr    DLmr         Dw    D1mw    DLmw 
--------------------------------------------------------------------------------
139,881,731 1,705 1,509 17,482,366 405,359 398,173 17,179,116 196,982 195,984  PROGRAM TOTALS

--------------------------------------------------------------------------------
        Ir I1mr ILmr         Dr    D1mr    DLmr         Dw    D1mw    DLmw  file:function
--------------------------------------------------------------------------------
66,355,214    3    3 12,441,600 194,402 194,402          0       0       0  /home/test/main.cpp:check(unsigned char const*, int, int)
49,766,412    1    1          0       0       0 12,441,600 194,401 194,401  /home/test/main.cpp:init(unsigned char*, int, int)
20,736,006    2    2  4,147,200  97,201  97,201  4,147,200       0       0  /home/test/main.cpp:simple_rgb2bgr(unsigned char*, int, int)
   648,017    3    3          0       0       0          0       0       0  /home/test/main.cpp:rgb2bgr_with_neon(unsigned char*, int, int)
   648,000    0    0    388,800  97,201  97,201    388,800       0       0  /usr/lib/gcc/aarch64-linux-gnu/5/include/arm_neon.h:rgb2bgr_with_neon(unsigned char*, int, int)
   548,801   12   11    149,830   2,085   1,743     50,617      36      24  /build/glibc-BinVK7/glibc-2.23/elf/dl-lookup.c:_dl_lookup_symbol_x
   541,307   42   41    192,949   5,227   1,153     94,348      53      33  /build/glibc-BinVK7/glibc-2.23/elf/dl-lookup.c:do_lookup_x
   211,533   28   28     48,140   3,391   2,964     20,629   1,748     887  /build/glibc-BinVK7/glibc-2.23/elf/../sysdeps/aarch64/dl-machine.h:_dl_relocate_object

--------------------------------------------------------------------------------
-- User-annotated source: main.cpp
--------------------------------------------------------------------------------
  No information has been collected for main.cpp

--------------------------------------------------------------------------------
-- Auto-annotated source: /home/test/main.cpp
--------------------------------------------------------------------------------
        Ir I1mr ILmr        Dr    D1mr    DLmr        Dw   D1mw   DLmw 

-- line 20 ----------------------------------------
         .    .    .         .       .       .         .      .      .  }                                               \
         .    .    .         .       .       .         .      .      .  else                                            \
         .    .    .         .       .       .         .      .      .  {                                               \
         .    .    .         .       .       .         .      .      .      std::cout << "Error:img.\n";                \
         .    .    .         .       .       .         .      .      .  }
         .    .    .         .       .       .         .      .      .  
         .    .    .         .       .       .         .      .      .  void init(uint8_t *img, int height, int width)
         .    .    .         .       .       .         .      .      .  {
         4    0    0         0       0       0         0      0      0      const int total_bytes = height * width * CHANNELS;
12,441,608    1    1         0       0       0         0      0      0      for (int i = 0; i < total_bytes; )
         .    .    .         .       .       .         .      .      .      {
12,441,600    0    0         0       0       0 4,147,200 64,801 64,801          *(img + i++) = RED;
12,441,600    0    0         0       0       0 4,147,200 64,800 64,800          *(img + i++) = GREEN;
12,441,600    0    0         0       0       0 4,147,200 64,800 64,800          *(img + i++) = BLUE;
         .    .    .         .       .       .         .      .      .      }
         .    .    .         .       .       .         .      .      .  }
         .    .    .         .       .       .         .      .      .  
         .    .    .         .       .       .         .      .      .  bool check(const uint8_t *img, int height, int width)
         .    .    .         .       .       .         .      .      .  {
         4    0    0         0       0       0         0      0      0      const int total_bytes = height * width * CHANNELS;
16,588,806    3    3         0       0       0         0      0      0      for (int i = 0; i < total_bytes; i += CHANNELS)
         .    .    .         .       .       .         .      .      .      {
41,472,000    0    0 8,294,400 129,602 129,602         0      0      0          if (*(img + i) != BLUE || *(img + i + 1) != GREEN || 
 8,294,400    0    0 4,147,200  64,800  64,800         0      0      0                  *(img + i + 2) != RED)
         .    .    .         .       .       .         .      .      .          {
         .    .    .         .       .       .         .      .      .              return false;
         .    .    .         .       .       .         .      .      .          }
         .    .    .         .       .       .         .      .      .      }
         4    0    0         0       0       0         0      0      0      return true;
         .    .    .         .       .       .         .      .      .  }
         .    .    .         .       .       .         .      .      .  
         .    .    .         .       .       .         .      .      .  void simple_rgb2bgr(uint8_t *img, int height, int width)
         .    .    .         .       .       .         .      .      .  {
         2    1    1         0       0       0         0      0      0      const int total_bytes = height * width * CHANNELS;
 8,294,404    0    0         0       0       0         0      0      0      for (int i = 0; i < total_bytes; i += CHANNELS)
         .    .    .         .       .       .         .      .      .      {
         .    .    .         .       .       .         .      .      .          // swap R and B channel
 4,147,200    0    0 2,073,600  32,401  32,401         0      0      0          const auto r_channel = *(img + i);
 6,220,800    1    1 2,073,600  64,800  64,800 2,073,600      0      0          *(img + i) = *(img + i + 2);
 2,073,600    0    0         0       0       0 2,073,600      0      0          *(img + i + 2) = r_channel;
         .    .    .         .       .       .         .      .      .      }
         .    .    .         .       .       .         .      .      .  }
         .    .    .         .       .       .         .      .      .  
         .    .    .         .       .       .         .      .      .  void rgb2bgr_with_neon(uint8_t *img, int height, int width)
         .    .    .         .       .       .         .      .      .  {
         2    1    1         0       0       0         0      0      0      const int total_bytes = height * width * CHANNELS;
         .    .    .         .       .       .         .      .      .      const int stride_bytes = 48;
         7    0    0         0       0       0         0      0      0      const int left_bytes = total_bytes % stride_bytes;
         1    1    1         0       0       0         0      0      0      const int multiply_bytes = total_bytes - left_bytes;
         .    .    .         .       .       .         .      .      .  
   518,404    0    0         0       0       0         0      0      0      for (int i = 0; i < multiply_bytes; i += stride_bytes)
         .    .    .         .       .       .         .      .      .      {
   129,600    0    0         0       0       0         0      0      0          uint8_t *target = img + i;
         .    .    .         .       .       .         .      .      .  
         .    .    .         .       .       .         .      .      .          // swap R and B channel with NEON
         .    .    .         .       .       .         .      .      .          uint8x16x3_t a = vld3q_u8(target);
         .    .    .         .       .       .         .      .      .          uint8x16x3_t b;
         .    .    .         .       .       .         .      .      .          b.val[0] = a.val[2];
         .    .    .         .       .       .         .      .      .          b.val[1] = a.val[1];
         .    .    .         .       .       .         .      .      .          b.val[2] = a.val[0];
         .    .    .         .       .       .         .      .      .          vst3q_u8(target, b);
         .    .    .         .       .       .         .      .      .      }
         .    .    .         .       .       .         .      .      .  
         .    .    .         .       .       .         .      .      .      // handling non-multiply array lengths
         3    1    1         0       0       0         0      0      0      for (int i = multiply_bytes; i < total_bytes; i += CHANNELS)
         .    .    .         .       .       .         .      .      .      {
         .    .    .         .       .       .         .      .      .          const auto r_channel = *(img + i);
         .    .    .         .       .       .         .      .      .          *(img + i) = *(img + i + 2);
         .    .    .         .       .       .         .      .      .          *(img + i + 2) = r_channel;
         .    .    .         .       .       .         .      .      .      }
         .    .    .         .       .       .         .      .      .  }
         .    .    .         .       .       .         .      .      .  
         .    .    .         .       .       .         .      .      .  uint8_t *rgb_img1 = nullptr;
         .    .    .         .       .       .         .      .      .  uint8_t *rgb_img2 = nullptr;
         .    .    .         .       .       .         .      .      .  
         .    .    .         .       .       .         .      .      .  int main()
         7    1    1         1       0       0         5      0      0  {
         7    1    1         0       0       0         1      1      1      rgb_img1 = new uint8_t[HEIGHT * WIDTH * CHANNELS];
         3    0    0         0       0       0         1      0      0      rgb_img2 = new uint8_t[HEIGHT * WIDTH * CHANNELS];
         .    .    .         .       .       .         .      .      .  
         4    0    0         1       0       0         0      0      0      init(rgb_img1, HEIGHT, WIDTH);
         4    1    1         1       1       1         0      0      0      init(rgb_img2, HEIGHT, WIDTH);
         .    .    .         .       .       .         .      .      .  
         6    1    1         1       1       1         0      0      0      simple_rgb2bgr(rgb_img1, HEIGHT, WIDTH);
         6    0    0         1       1       1         0      0      0      rgb2bgr_with_neon(rgb_img2, HEIGHT, WIDTH);
         .    .    .         .       .       .         .      .      .  
        12    1    1         1       1       1         0      0      0      CHECK(rgb_img1, HEIGHT, WIDTH);
        14    1    1         1       1       1         0      0      0      CHECK(rgb_img2, HEIGHT, WIDTH);
         .    .    .         .       .       .         .      .      .      
         5    1    1         1       1       1         0      0      0      delete [] rgb_img2;
         5    0    0         1       0       0         0      0      0      delete [] rgb_img1;
         .    .    .         .       .       .         .      .      .      return 0;
        27    4    3        11       3       1         5      0      0  }

--------------------------------------------------------------------------------
-- Auto-annotated source: /usr/lib/gcc/aarch64-linux-gnu/5/include/arm_neon.h
--------------------------------------------------------------------------------
     Ir I1mr ILmr      Dr   D1mr   DLmr      Dw D1mw DLmw 

-- line 16243 ----------------------------------------
      .    .    .       .      .      .       .    .    .    return ret;
      .    .    .       .      .      .       .    .    .  }
      .    .    .       .      .      .       .    .    .  
      .    .    .       .      .      .       .    .    .  __extension__ static __inline uint8x16x3_t __attribute__ ((__always_inline__))
      .    .    .       .      .      .       .    .    .  vld3q_u8 (const uint8_t * __a)
      .    .    .       .      .      .       .    .    .  {
      .    .    .       .      .      .       .    .    .    uint8x16x3_t ret;
      .    .    .       .      .      .       .    .    .    __builtin_aarch64_simd_ci __o;
129,600    0    0 388,800 97,201 97,201       0    0    0    __o = __builtin_aarch64_ld3v16qi ((const __builtin_aarch64_simd_qi *) __a);
      .    .    .       .      .      .       .    .    .    ret.val[0] = (uint8x16_t) __builtin_aarch64_get_qregciv16qi (__o, 0);
      .    .    .       .      .      .       .    .    .    ret.val[1] = (uint8x16_t) __builtin_aarch64_get_qregciv16qi (__o, 1);
      .    .    .       .      .      .       .    .    .    ret.val[2] = (uint8x16_t) __builtin_aarch64_get_qregciv16qi (__o, 2);
      .    .    .       .      .      .       .    .    .    return ret;
      .    .    .       .      .      .       .    .    .  }
      .    .    .       .      .      .       .    .    .  
      .    .    .       .      .      .       .    .    .  __extension__ static __inline uint16x8x3_t __attribute__ ((__always_inline__))
      .    .    .       .      .      .       .    .    .  vld3q_u16 (const uint16_t * __a)
-- line 16259 ----------------------------------------
-- line 23620 ----------------------------------------
      .    .    .       .      .      .       .    .    .    __o = __builtin_aarch64_set_qregciv2di (__o, (int64x2_t) val.val[2], 2);
      .    .    .       .      .      .       .    .    .    __builtin_aarch64_st3v2di ((__builtin_aarch64_simd_di *) __a, __o);
      .    .    .       .      .      .       .    .    .  }
      .    .    .       .      .      .       .    .    .  
      .    .    .       .      .      .       .    .    .  __extension__ static __inline void __attribute__ ((__always_inline__))
      .    .    .       .      .      .       .    .    .  vst3q_u8 (uint8_t * __a, uint8x16x3_t val)
      .    .    .       .      .      .       .    .    .  {
      .    .    .       .      .      .       .    .    .    __builtin_aarch64_simd_ci __o;
129,600    0    0       0      0      0       0    0    0    __o = __builtin_aarch64_set_qregciv16qi (__o, (int8x16_t) val.val[0], 0);
129,600    0    0       0      0      0       0    0    0    __o = __builtin_aarch64_set_qregciv16qi (__o, (int8x16_t) val.val[1], 1);
129,600    0    0       0      0      0       0    0    0    __o = __builtin_aarch64_set_qregciv16qi (__o, (int8x16_t) val.val[2], 2);
129,600    0    0       0      0      0 388,800    0    0    __builtin_aarch64_st3v16qi ((__builtin_aarch64_simd_qi *) __a, __o);
      .    .    .       .      .      .       .    .    .  }
      .    .    .       .      .      .       .    .    .  
      .    .    .       .      .      .       .    .    .  __extension__ static __inline void __attribute__ ((__always_inline__))
      .    .    .       .      .      .       .    .    .  vst3q_u16 (uint16_t * __a, uint16x8x3_t val)
      .    .    .       .      .      .       .    .    .  {
      .    .    .       .      .      .       .    .    .    __builtin_aarch64_simd_ci __o;
      .    .    .       .      .      .       .    .    .    __o = __builtin_aarch64_set_qregciv8hi (__o, (int16x8_t) val.val[0], 0);
      .    .    .       .      .      .       .    .    .    __o = __builtin_aarch64_set_qregciv8hi (__o, (int16x8_t) val.val[1], 1);
-- line 23639 ----------------------------------------

--------------------------------------------------------------------------------
The following files chosen for auto-annotation could not be found:
--------------------------------------------------------------------------------
  /build/glibc-BinVK7/glibc-2.23/elf/dl-lookup.c
  /build/glibc-BinVK7/glibc-2.23/elf/../sysdeps/aarch64/dl-machine.h

--------------------------------------------------------------------------------
Ir I1mr ILmr Dr D1mr DLmr Dw D1mw DLmw 
--------------------------------------------------------------------------------
99    1    1 97   96   98 99   99   99  percentage of events annotated
```

分析统计结果：

| 函数名称          | 内存读操作数量(Dr列) | 一级数据缓存读不命中次数(D1mr列) | 最后一级数据缓存读不命中次数(DLmr列) | 内存写操作数量(Dw列) | 一级数据缓存写不命中次数(D1mw列) | 最后一级数据缓存写不命中次数(DLmw列) |
| ----------------- | -------------------- | -------------------------------- | ------------------------------------ | -------------------- | -------------------------------- | ------------------------------------ |
| simple_rgb2bgr    | 4,147,200            | 97,201                           | 97,201                               | 4,147,200            | 0                                | 0                                    |
| rgb2bgr_with_neon | 388,800              | 97,201                           | 97,201                               | 388,800              | 0                                | 0                                    |

可以看出，`rgb2bgr_with_neon`函数的内存读操作数量和内存写操作数量分别为`simple_rgb2bgr`函数的 0.09375 倍，其他的都是相同的。因此，从 cache 的角度来看，NEON 大大减少了内存读写操作数量，从而改善程序性能。