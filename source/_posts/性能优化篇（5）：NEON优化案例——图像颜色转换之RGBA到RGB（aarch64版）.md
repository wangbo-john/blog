# 性能优化篇（5）：NEON优化案例——图像颜色转换之RGBA到RGB（aarch64版）

Author:stormQ

Sunday, 8. December 2019 1 12:47AM

- 目录
  - [实现一个基本的图像转换函数](https://blog.csdn.net/wohenfanjian/article/details/103482021#jump实现一个基本的图像转换函数)
  - [利用 NEON 加速图像转换](https://blog.csdn.net/wohenfanjian/article/details/103482021#jump利用NEON加速图像转换)

------

### 实现一个基本的图像转换函数

如何将一个大小为`672x376`的`RGBA`图像转换成相同大小的`RGB`图像呢？

首先实现一个最简单的转换函数`simple_rgba2rgb()`，完整实现如下：

```
#define SRC_CHANNELS 4
#define DST_CHANNELS 3

void simple_rgba2rgb(const uint8_t *rgba_img, uint8_t *rgb_img, 
                      int height, int width)
{
    const int total_pixels = height * width;
    for (int i = 0; i < total_pixels; i++)
    {
        const int src_idx = i * SRC_CHANNELS;
        const int dst_idx = i * DST_CHANNELS;
        *(rgb_img + dst_idx) = *(rgba_img + src_idx);
        *(rgb_img + dst_idx + 1) = *(rgba_img + src_idx + 1);
        *(rgb_img + dst_idx + 2) = *(rgba_img + src_idx + 2);
    }
}
```

`simple_rgba2rgb()`函数将一个二维的图像看作是一维的，意味着只需要一个循环就可以遍历所有的像素。这种用法的好处在于，遵循空间局部性原理以高效利用`cache`从而改善程序性能，而不必关心二维图像在内存中的存储顺序——是行主序还是列主序。如果用两个循环的话会比较繁琐，移植性也不好。另外，由于源图像和目的图像的大小相同但所占的字节数不同，所以采用每次遍历一个像素的方式，这样便于处理。

------

### 利用 NEON 加速图像转换

首先，实现一个基本的利用`NEON`加速图像转换的函数`rgba2rgb_with_neon()`，完整实现如下：

```
void rgba2rgb_with_neon(const uint8_t *rgba_img, uint8_t *rgb_img, 
                        int height, int width)
{
    const int total_pixels = height * width;
    const int stride_pixels = 16;

    for (int i = 0; i < total_pixels; i += stride_pixels)
    {
        const uint8_t *src = rgba_img + i * SRC_CHANNELS;
        uint8_t *dst = rgb_img + i * DST_CHANNELS;
        
        uint8x16x4_t a = vld4q_u8(src);
        uint8x16x3_t b;
        b.val[0] = a.val[0];
        b.val[1] = a.val[1];
        b.val[2] = a.val[2];
        vst3q_u8(dst, b);
    }
}
```

上述函数没有考虑图像像素个数非整除时的情形，这个放到后面进行。该函数的实现用到了两个`NEON Intrinsics`，分别是`vld4q_u8`和`vst3q_u8`，两者都定义在`arm_neon.h`文件中。

`vld4q_u8`的函数原型为`uint8x16x4_t vld4q_u8 (const uint8_t * __a)`，作用为以步长为 4 交叉地加载数据到四个连续的 128-bit 的向量寄存器。具体地：将内存地址`__a`、`__a+4`、…、`__a+60`处的内容分别赋值给向量寄存器`Vn`的`lane[0]`、`lane[1]`、…、`lane[15]`，将内存地址`__a+1`、`__a+5`、…、`__a+61`处的内容分别赋值给向量寄存器`Vn+1`的`lane[0]`、`lane[1]`、…、`lane[15]`，将内存地址`__a+2`、`__a+6`、…、`__a+62`处的内容分别赋值给向量寄存器`Vn+2`的`lane[0]`、`lane[1]`、…、`lane[15]`，将内存地址`__a+3`、`__a+7`、…、`__a+63`处的内容分别赋值给向量寄存器`Vn+3`的`lane[0]`、`lane[1]`、…、`lane[15]`。也就是说，`vld4q_u8`在上述函数中的作用为：将连续 16 个像素的 R 通道： R0、R1、… 、R15 加载到向量寄存器`Vn`，将连续 16 个像素的 G 通道： G0、G1、… 、G15 加载到向量寄存器`Vn+1`，将连续 16 个像素的 B 通道： B0、B1、… 、B15 加载到向量寄存器`Vn+2`，将连续 16 个像素的 Alpha 通道： A0、A1、… 、A15 加载到向量寄存器`Vn+3`。

`vst3q_u8`的函数原型为`void vst3q_u8 (uint8_t * __a, uint8x16x3_t val)`，作用为以步长为 3 交叉地存储数据到内存中。

由于每一次迭代可以同时处理 16 个连续的像素。所以，变量`stride_pixels`的值为 16。

由于`aarch64`指令集中没有`ARM` 指令集中的`VSWP`指令。因此，这里引入一个类型为`uint8x16x3_t`的临时向量`b`用于存储`R`、`G`和`B`通道。 g++ 带优化选项`-Og`编译时，语句`b.val[0] = a.val[2]; b.val[1] = a.val[1]; b.val[2] = a.val[0];`对应三条`MOV`指令。

接着用语句`vst3q_u8(dst, b);`将转换后的结果写入内存。这样，一次图像转换操作就完成了。

现在考虑图像像素个数非整除时的情形，完整实现如下：

```
void rgba2rgb_with_neon(const uint8_t *rgba_img, uint8_t *rgb_img, 
                        int height, int width)
{
    const int total_pixels = height * width;
    const int stride_pixels = 16;
    const int left_pixels = total_pixels % stride_pixels;
    const int multiply_pixels = total_pixels - left_pixels;

    for (int i = 0; i < multiply_pixels; i += stride_pixels)
    {
        const uint8_t *src = rgba_img + i * SRC_CHANNELS;
        uint8_t *dst = rgb_img + i * DST_CHANNELS;
        
        uint8x16x4_t a = vld4q_u8(src);
        uint8x16x3_t b;
        b.val[0] = a.val[0];
        b.val[1] = a.val[1];
        b.val[2] = a.val[2];
        vst3q_u8(dst, b);
    }

    // handling non-multiply array lengths
    for (int i = multiply_pixels; i < total_pixels; i++)
    {
        const int src_idx = i * SRC_CHANNELS;
        const int dst_idx = i * DST_CHANNELS;
        *(rgb_img + dst_idx) = *(rgba_img + src_idx);
        *(rgb_img + dst_idx + 1) = *(rgba_img + src_idx + 1);
        *(rgb_img + dst_idx + 2) = *(rgba_img + src_idx + 2);
    }
}
```

为了验证`rgba2rgb_with_neon()`函数的正确性，这里引入两个函数：`init()`和`check()`。`init()`函数用于初始化图像，`check()`函数用于检查转换后的图像是否正确。完整实现如下：

```
#define RED 255
#define GREEN 125
#define BLUE 80
#define ALPHA 100

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
    const int total_bytes = height * width * SRC_CHANNELS;
    for (int i = 0; i < total_bytes; )
    {
        *(img + i++) = RED;
        *(img + i++) = GREEN;
        *(img + i++) = BLUE;
        *(img + i++) = ALPHA;
    }
}

bool check(const uint8_t *img, int height, int width)
{
    const int total_bytes = height * width * DST_CHANNELS;
    for (int i = 0; i < total_bytes; i += DST_CHANNELS)
    {
        if (*(img + i) != RED || *(img + i + 1) != GREEN || 
                *(img + i + 2) != BLUE)
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

#define HEIGHT 376
#define WIDTH 672
#define SRC_CHANNELS 4
#define DST_CHANNELS 3
#define RED 255
#define GREEN 125
#define BLUE 80
#define ALPHA 100

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
    const int total_bytes = height * width * SRC_CHANNELS;
    for (int i = 0; i < total_bytes; )
    {
        *(img + i++) = RED;
        *(img + i++) = GREEN;
        *(img + i++) = BLUE;
        *(img + i++) = ALPHA;
    }
}

bool check(const uint8_t *img, int height, int width)
{
    const int total_bytes = height * width * DST_CHANNELS;
    for (int i = 0; i < total_bytes; i += DST_CHANNELS)
    {
        if (*(img + i) != RED || *(img + i + 1) != GREEN || 
                *(img + i + 2) != BLUE)
        {
            return false;
        }
    }
    return true;
}

void simple_rgba2rgb(const uint8_t *rgba_img, uint8_t *rgb_img, 
                      int height, int width)
{
    const int total_pixels = height * width;
    for (int i = 0; i < total_pixels; i++)
    {
        const int src_idx = i * SRC_CHANNELS;
        const int dst_idx = i * DST_CHANNELS;
        *(rgb_img + dst_idx) = *(rgba_img + src_idx);
        *(rgb_img + dst_idx + 1) = *(rgba_img + src_idx + 1);
        *(rgb_img + dst_idx + 2) = *(rgba_img + src_idx + 2);
    }
}

void rgba2rgb_with_neon(const uint8_t *rgba_img, uint8_t *rgb_img, 
                        int height, int width)
{
    const int total_pixels = height * width;
    const int stride_pixels = 16;
    const int left_pixels = total_pixels % stride_pixels;
    const int multiply_pixels = total_pixels - left_pixels;

    for (int i = 0; i < multiply_pixels; i += stride_pixels)
    {
        const uint8_t *src = rgba_img + i * SRC_CHANNELS;
        uint8_t *dst = rgb_img + i * DST_CHANNELS;
        
        uint8x16x4_t a = vld4q_u8(src);
        uint8x16x3_t b;
        b.val[0] = a.val[0];
        b.val[1] = a.val[1];
        b.val[2] = a.val[2];
        vst3q_u8(dst, b);
    }

    // handling non-multiply array lengths
    for (int i = multiply_pixels; i < total_pixels; i++)
    {
        const int src_idx = i * SRC_CHANNELS;
        const int dst_idx = i * DST_CHANNELS;
        *(rgb_img + dst_idx) = *(rgba_img + src_idx);
        *(rgb_img + dst_idx + 1) = *(rgba_img + src_idx + 1);
        *(rgb_img + dst_idx + 2) = *(rgba_img + src_idx + 2);
    }
}

uint8_t *rgba_img = nullptr;
uint8_t *rgb_img1 = nullptr;
uint8_t *rgb_img2 = nullptr;

int main()
{
    rgba_img = new uint8_t[HEIGHT * WIDTH * SRC_CHANNELS];
    rgb_img1 = new uint8_t[HEIGHT * WIDTH * DST_CHANNELS];
    rgb_img2 = new uint8_t[HEIGHT * WIDTH * DST_CHANNELS];

    init(rgba_img, HEIGHT, WIDTH);

    simple_rgba2rgb(rgba_img, rgb_img1, HEIGHT, WIDTH);
    rgba2rgb_with_neon(rgba_img, rgb_img2, HEIGHT, WIDTH);

    CHECK(rgb_img1, HEIGHT, WIDTH);
    CHECK(rgb_img2, HEIGHT, WIDTH);
    
    delete [] rgb_img2;
    delete [] rgb_img1;
    delete [] rgba_img;
    return 0;
}
```

编译程序（On Jetson TX2）：

```
g++ -std=c++11 -g -Og -o main_Og main.cpp
```

| 启动程序方式 | 第一次执行耗时(us)                          | 第二次执行耗时(us)                          | 第三次执行耗时(us)                          | 第四次执行耗时(us)                          | 第五次执行耗时(us)                          |
| ------------ | ------------------------------------------- | ------------------------------------------- | ------------------------------------------- | ------------------------------------------- | ------------------------------------------- |
| ./main_Og    | simple_rgba2rgb:4446rgba2rgb_with_neon:2185 | simple_rgba2rgb:6027rgba2rgb_with_neon:2106 | simple_rgba2rgb:5748rgba2rgb_with_neon:2184 | simple_rgba2rgb:5372rgba2rgb_with_neon:2227 | simple_rgba2rgb:4446rgba2rgb_with_neon:2169 |

从统计结果中可以看出，`rgba2rgb_with_neon()`函数的执行速度比`simple_rgba2rgb()`函数快 2 到 3 倍左右。

**那么 NEON 版的实现为什么能达到这样一个加速呢？** 下面从 cache 的角度进行分析。

统计 cache 性能数据：

```
--------------------------------------------------------------------------------
I1 cache:         16384 B, 64 B, 4-way associative
D1 cache:         16384 B, 64 B, 4-way associative
LL cache:         262144 B, 64 B, 8-way associative
Command:          ./main_Og
Data file:        cachegrind.out.26634
Events recorded:  Ir I1mr ILmr Dr D1mr DLmr Dw D1mw DLmw
Events shown:     Ir I1mr ILmr Dr D1mr DLmr Dw D1mw DLmw
Event sort order: Ir I1mr ILmr Dr D1mr DLmr Dw D1mw DLmw
Thresholds:       0.1 100 100 100 100 100 100 100 100
Include dirs:     
User annotated:   main.cpp
Auto-annotation:  on

--------------------------------------------------------------------------------
        Ir  I1mr  ILmr        Dr   D1mr   DLmr        Dw   D1mw   DLmw 
--------------------------------------------------------------------------------
18,330,632 1,581 1,388 2,833,111 71,198 64,190 2,014,469 42,002 41,005  PROGRAM TOTALS

--------------------------------------------------------------------------------
       Ir I1mr ILmr        Dr   D1mr   DLmr        Dw   D1mw   DLmw  file:function
--------------------------------------------------------------------------------
8,085,518    0    0 1,516,032 23,690 23,690         0      0      0  /home/test2/main.cpp:check(unsigned char const*, int, int)
4,548,101    3    3   758,016 15,793 15,793   758,016 11,845 11,845  /home/test2/main.cpp:simple_rgba2rgb(unsigned char const*, unsigned char*, int, int)
3,790,086    2    2         0      0      0 1,010,688 15,792 15,792  /home/test2/main.cpp:init(unsigned char*, int, int)
  545,429   12   11   148,897  2,038  1,708    50,257     29     16  /build/glibc-BinVK7/glibc-2.23/elf/dl-lookup.c:_dl_lookup_symbol_x
  537,013   36   36   191,329  5,065  1,024    93,645     43     24  /build/glibc-BinVK7/glibc-2.23/elf/dl-lookup.c:do_lookup_x
  211,319   28   28    48,062  3,377  2,954    20,608  1,748    886  /build/glibc-BinVK7/glibc-2.23/elf/../sysdeps/aarch64/dl-machine.h:_dl_relocate_object
  126,348    0    0         0      0      0         0      0      0  /home/test2/main.cpp:rgba2rgb_with_neon(unsigned char const*, unsigned char*, int, int)
  108,209    8    6    37,471    330    238    14,588     25     11  /build/glibc-BinVK7/glibc-2.23/elf/dl-lookup.c:check_match
   97,629    5    5    12,840    221     96         0      0      0  /build/glibc-BinVK7/glibc-2.23/string/../sysdeps/aarch64/strcmp.S:strcmp
   78,960    1    1    63,168 15,793 15,793    47,376 11,845 11,845  /usr/lib/gcc/aarch64-linux-gnu/5/include/arm_neon.h:rgba2rgb_with_neon(unsigned char const*, unsigned char*, int, int)
   56,685    3    3    16,081  2,184  1,017     3,209      2      0  /build/glibc-BinVK7/glibc-2.23/elf/do-rel.h:_dl_relocate_object
   44,112    8    8    13,412  1,022    986        12      1      0  /build/glibc-BinVK7/glibc-2.23/elf/dl-addr.c:_dl_addr

--------------------------------------------------------------------------------
-- User-annotated source: main.cpp
--------------------------------------------------------------------------------
  No information has been collected for main.cpp

--------------------------------------------------------------------------------
-- Auto-annotated source: /usr/lib/gcc/aarch64-linux-gnu/5/include/arm_neon.h
--------------------------------------------------------------------------------
    Ir I1mr ILmr     Dr   D1mr   DLmr     Dw   D1mw   DLmw 

-- line 16574 ----------------------------------------
     .    .    .      .      .      .      .      .      .    return ret;
     .    .    .      .      .      .      .      .      .  }
     .    .    .      .      .      .      .      .      .  
     .    .    .      .      .      .      .      .      .  __extension__ static __inline uint8x16x4_t __attribute__ ((__always_inline__))
     .    .    .      .      .      .      .      .      .  vld4q_u8 (const uint8_t * __a)
     .    .    .      .      .      .      .      .      .  {
     .    .    .      .      .      .      .      .      .    uint8x16x4_t ret;
     .    .    .      .      .      .      .      .      .    __builtin_aarch64_simd_xi __o;
15,792    0    0 63,168 15,793 15,793      0      0      0    __o = __builtin_aarch64_ld4v16qi ((const __builtin_aarch64_simd_qi *) __a);
     .    .    .      .      .      .      .      .      .    ret.val[0] = (uint8x16_t) __builtin_aarch64_get_qregxiv16qi (__o, 0);
     .    .    .      .      .      .      .      .      .    ret.val[1] = (uint8x16_t) __builtin_aarch64_get_qregxiv16qi (__o, 1);
     .    .    .      .      .      .      .      .      .    ret.val[2] = (uint8x16_t) __builtin_aarch64_get_qregxiv16qi (__o, 2);
     .    .    .      .      .      .      .      .      .    ret.val[3] = (uint8x16_t) __builtin_aarch64_get_qregxiv16qi (__o, 3);
     .    .    .      .      .      .      .      .      .    return ret;
     .    .    .      .      .      .      .      .      .  }
     .    .    .      .      .      .      .      .      .  
     .    .    .      .      .      .      .      .      .  __extension__ static __inline uint16x8x4_t __attribute__ ((__always_inline__))
-- line 16590 ----------------------------------------
-- line 23620 ----------------------------------------
     .    .    .      .      .      .      .      .      .    __o = __builtin_aarch64_set_qregciv2di (__o, (int64x2_t) val.val[2], 2);
     .    .    .      .      .      .      .      .      .    __builtin_aarch64_st3v2di ((__builtin_aarch64_simd_di *) __a, __o);
     .    .    .      .      .      .      .      .      .  }
     .    .    .      .      .      .      .      .      .  
     .    .    .      .      .      .      .      .      .  __extension__ static __inline void __attribute__ ((__always_inline__))
     .    .    .      .      .      .      .      .      .  vst3q_u8 (uint8_t * __a, uint8x16x3_t val)
     .    .    .      .      .      .      .      .      .  {
     .    .    .      .      .      .      .      .      .    __builtin_aarch64_simd_ci __o;
15,792    0    0      0      0      0      0      0      0    __o = __builtin_aarch64_set_qregciv16qi (__o, (int8x16_t) val.val[0], 0);
15,792    1    1      0      0      0      0      0      0    __o = __builtin_aarch64_set_qregciv16qi (__o, (int8x16_t) val.val[1], 1);
15,792    0    0      0      0      0      0      0      0    __o = __builtin_aarch64_set_qregciv16qi (__o, (int8x16_t) val.val[2], 2);
15,792    0    0      0      0      0 47,376 11,845 11,845    __builtin_aarch64_st3v16qi ((__builtin_aarch64_simd_qi *) __a, __o);
     .    .    .      .      .      .      .      .      .  }
     .    .    .      .      .      .      .      .      .  
     .    .    .      .      .      .      .      .      .  __extension__ static __inline void __attribute__ ((__always_inline__))
     .    .    .      .      .      .      .      .      .  vst3q_u16 (uint16_t * __a, uint16x8x3_t val)
     .    .    .      .      .      .      .      .      .  {
     .    .    .      .      .      .      .      .      .    __builtin_aarch64_simd_ci __o;
     .    .    .      .      .      .      .      .      .    __o = __builtin_aarch64_set_qregciv8hi (__o, (int16x8_t) val.val[0], 0);
     .    .    .      .      .      .      .      .      .    __o = __builtin_aarch64_set_qregciv8hi (__o, (int16x8_t) val.val[1], 1);
-- line 23639 ----------------------------------------

--------------------------------------------------------------------------------
-- Auto-annotated source: /home/test2/main.cpp
--------------------------------------------------------------------------------
       Ir I1mr ILmr        Dr   D1mr   DLmr      Dw   D1mw   DLmw 

-- line 22 ----------------------------------------
        .    .    .         .      .      .       .      .      .  }                                               \
        .    .    .         .      .      .       .      .      .  else                                            \
        .    .    .         .      .      .       .      .      .  {                                               \
        .    .    .         .      .      .       .      .      .      std::cout << "Error:img.\n";                \
        .    .    .         .      .      .       .      .      .  }
        .    .    .         .      .      .       .      .      .  
        .    .    .         .      .      .       .      .      .  void init(uint8_t *img, int height, int width)
        .    .    .         .      .      .       .      .      .  {
        2    0    0         0      0      0       0      0      0      const int total_bytes = height * width * SRC_CHANNELS;
  758,020    2    2         0      0      0       0      0      0      for (int i = 0; i < total_bytes; )
        .    .    .         .      .      .       .      .      .      {
  758,016    0    0         0      0      0 252,672 15,792 15,792          *(img + i++) = RED;
  758,016    0    0         0      0      0 252,672      0      0          *(img + i++) = GREEN;
  758,016    0    0         0      0      0 252,672      0      0          *(img + i++) = BLUE;
  758,016    0    0         0      0      0 252,672      0      0          *(img + i++) = ALPHA;
        .    .    .         .      .      .       .      .      .      }
        .    .    .         .      .      .       .      .      .  }
        .    .    .         .      .      .       .      .      .  
        .    .    .         .      .      .       .      .      .  bool check(const uint8_t *img, int height, int width)
        .    .    .         .      .      .       .      .      .  {
        4    0    0         0      0      0       0      0      0      const int total_bytes = height * width * DST_CHANNELS;
2,021,382    0    0         0      0      0       0      0      0      for (int i = 0; i < total_bytes; i += DST_CHANNELS)
        .    .    .         .      .      .       .      .      .      {
5,053,440    0    0 1,010,688 15,794 15,794       0      0      0          if (*(img + i) != RED || *(img + i + 1) != GREEN || 
1,010,688    0    0   505,344  7,896  7,896       0      0      0                  *(img + i + 2) != BLUE)
        .    .    .         .      .      .       .      .      .          {
        .    .    .         .      .      .       .      .      .              return false;
        .    .    .         .      .      .       .      .      .          }
        .    .    .         .      .      .       .      .      .      }
        4    0    0         0      0      0       0      0      0      return true;
        .    .    .         .      .      .       .      .      .  }
        .    .    .         .      .      .       .      .      .  
        .    .    .         .      .      .       .      .      .  void simple_rgba2rgb(const uint8_t *rgba_img, uint8_t *rgb_img, 
        .    .    .         .      .      .       .      .      .                        int height, int width)
        .    .    .         .      .      .       .      .      .  {
        1    1    1         0      0      0       0      0      0      const int total_pixels = height * width;
1,010,692    1    1         0      0      0       0      0      0      for (int i = 0; i < total_pixels; i++)
        .    .    .         .      .      .       .      .      .      {
  252,672    1    1         0      0      0       0      0      0          const int src_idx = i * SRC_CHANNELS;
  252,672    0    0         0      0      0       0      0      0          const int dst_idx = i * DST_CHANNELS;
1,010,688    0    0   252,672 15,793 15,793 252,672  3,949  3,949          *(rgb_img + dst_idx) = *(rgba_img + src_idx);
1,010,688    0    0   252,672      0      0 252,672  3,948  3,948          *(rgb_img + dst_idx + 1) = *(rgba_img + src_idx + 1);
1,010,688    0    0   252,672      0      0 252,672  3,948  3,948          *(rgb_img + dst_idx + 2) = *(rgba_img + src_idx + 2);
        .    .    .         .      .      .       .      .      .      }
        .    .    .         .      .      .       .      .      .  }
        .    .    .         .      .      .       .      .      .  
        .    .    .         .      .      .       .      .      .  void rgba2rgb_with_neon(const uint8_t *rgba_img, uint8_t *rgb_img, 
        .    .    .         .      .      .       .      .      .                          int height, int width)
        .    .    .         .      .      .       .      .      .  {
        1    0    0         0      0      0       0      0      0      const int total_pixels = height * width;
        .    .    .         .      .      .       .      .      .      const int stride_pixels = 16;
        4    0    0         0      0      0       0      0      0      const int left_pixels = total_pixels % stride_pixels;
        1    0    0         0      0      0       0      0      0      const int multiply_pixels = total_pixels - left_pixels;
        .    .    .         .      .      .       .      .      .  
   63,171    0    0         0      0      0       0      0      0      for (int i = 0; i < multiply_pixels; i += stride_pixels)
        .    .    .         .      .      .       .      .      .      {
   31,584    0    0         0      0      0       0      0      0          const uint8_t *src = rgba_img + i * SRC_CHANNELS;
   31,584    0    0         0      0      0       0      0      0          uint8_t *dst = rgb_img + i * DST_CHANNELS;
        .    .    .         .      .      .       .      .      .          
        .    .    .         .      .      .       .      .      .          uint8x16x4_t a = vld4q_u8(src);
        .    .    .         .      .      .       .      .      .          uint8x16x3_t b;
        .    .    .         .      .      .       .      .      .          b.val[0] = a.val[0];
        .    .    .         .      .      .       .      .      .          b.val[1] = a.val[1];
        .    .    .         .      .      .       .      .      .          b.val[2] = a.val[2];
        .    .    .         .      .      .       .      .      .          vst3q_u8(dst, b);
        .    .    .         .      .      .       .      .      .      }
        .    .    .         .      .      .       .      .      .  
        .    .    .         .      .      .       .      .      .      // handling non-multiply array lengths
        3    0    0         0      0      0       0      0      0      for (int i = multiply_pixels; i < total_pixels; i++)
        .    .    .         .      .      .       .      .      .      {
        .    .    .         .      .      .       .      .      .          const int src_idx = i * SRC_CHANNELS;
        .    .    .         .      .      .       .      .      .          const int dst_idx = i * DST_CHANNELS;
        .    .    .         .      .      .       .      .      .          *(rgb_img + dst_idx) = *(rgba_img + src_idx);
        .    .    .         .      .      .       .      .      .          *(rgb_img + dst_idx + 1) = *(rgba_img + src_idx + 1);
        .    .    .         .      .      .       .      .      .          *(rgb_img + dst_idx + 2) = *(rgba_img + src_idx + 2);
        .    .    .         .      .      .       .      .      .      }
        .    .    .         .      .      .       .      .      .  }
        .    .    .         .      .      .       .      .      .  
        .    .    .         .      .      .       .      .      .  uint8_t *rgba_img = nullptr;
        .    .    .         .      .      .       .      .      .  uint8_t *rgb_img1 = nullptr;
        .    .    .         .      .      .       .      .      .  uint8_t *rgb_img2 = nullptr;
        .    .    .         .      .      .       .      .      .  
        .    .    .         .      .      .       .      .      .  int main()
        3    1    1         0      0      0       4      0      0  {
        6    1    1         0      0      0       1      1      0      rgba_img = new uint8_t[HEIGHT * WIDTH * SRC_CHANNELS];
        5    0    0         0      0      0       1      1      1      rgb_img1 = new uint8_t[HEIGHT * WIDTH * DST_CHANNELS];
        3    0    0         0      0      0       1      0      0      rgb_img2 = new uint8_t[HEIGHT * WIDTH * DST_CHANNELS];
        .    .    .         .      .      .       .      .      .  
        4    0    0         1      0      0       0      0      0      init(rgba_img, HEIGHT, WIDTH);
        .    .    .         .      .      .       .      .      .  
        5    1    1         2      2      2       0      0      0      simple_rgba2rgb(rgba_img, rgb_img1, HEIGHT, WIDTH);
        5    0    0         2      2      2       0      0      0      rgba2rgb_with_neon(rgba_img, rgb_img2, HEIGHT, WIDTH);
        .    .    .         .      .      .       .      .      .  
       12    1    1         1      1      1       0      0      0      CHECK(rgb_img1, HEIGHT, WIDTH);
       14    1    1         1      1      1       0      0      0      CHECK(rgb_img2, HEIGHT, WIDTH);
        .    .    .         .      .      .       .      .      .      
        5    0    0         1      1      1       0      0      0      delete [] rgb_img2;
        5    1    1         1      0      0       0      0      0      delete [] rgb_img1;
        5    0    0         1      0      0       0      0      0      delete [] rgba_img;
        .    .    .         .      .      .       .      .      .      return 0;
       22    3    2         9      0      0       5      0      0  }

-- line 123 ----------------------------------------

--------------------------------------------------------------------------------
The following files chosen for auto-annotation could not be found:
--------------------------------------------------------------------------------
  /build/glibc-BinVK7/glibc-2.23/elf/do-rel.h
  /build/glibc-BinVK7/glibc-2.23/elf/dl-lookup.c
  /build/glibc-BinVK7/glibc-2.23/string/../sysdeps/aarch64/strcmp.S
  /build/glibc-BinVK7/glibc-2.23/elf/dl-addr.c
  /build/glibc-BinVK7/glibc-2.23/elf/../sysdeps/aarch64/dl-machine.h

--------------------------------------------------------------------------------
Ir I1mr ILmr Dr D1mr DLmr Dw D1mw DLmw 
--------------------------------------------------------------------------------
91    1    1 82   78   86 90   94   96  percentage of events annotated
```

分析统计结果：

| 函数名称           | 内存读操作数量(Dr列) | 一级数据缓存读不命中次数(D1mr列) | 最后一级数据缓存读不命中次数(DLmr列) | 内存写操作数量(Dw列) | 一级数据缓存写不命中次数(D1mw列) | 最后一级数据缓存写不命中次数(DLmw列) |
| ------------------ | -------------------- | -------------------------------- | ------------------------------------ | -------------------- | -------------------------------- | ------------------------------------ |
| simple_rgba2rgb    | 758,016              | 15,793                           | 15,793                               | 758,016              | 11,845                           | 11,845                               |
| rgba2rgb_with_neon | 63,168               | 15,793                           | 15,793                               | 47,376               | 11,845                           | 11,845                               |

可以看出，`rgba2rgb_with_neon`函数的内存读操作数量和内存写操作数量分别为`simple_rgba2rgb`函数的 0.0833 倍、0.0625 倍，其他的都是相同的。因此，从 cache 的角度来看，NEON 大大减少了内存读写操作数量，从而改善程序性能。

**最后，能否降低 cache 不命中率从而进一步改善程序性能呢？…**