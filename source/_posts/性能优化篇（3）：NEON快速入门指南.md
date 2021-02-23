

# 性能优化篇（3）：NEON快速入门指南

Author:stormQ

Sunday, 24. November 2019 10:28PM

------

- 目录
  - [向量数据类型](https://blog.csdn.net/wohenfanjian/article/details/103283319#jump向量数据类型)
  - [向量赋值](https://blog.csdn.net/wohenfanjian/article/details/103283319#jump向量赋值)
  - [访问/存储向量的值](https://blog.csdn.net/wohenfanjian/article/details/103283319#jump访问/存储向量的值)
  - [向量算术逻辑运算](https://blog.csdn.net/wohenfanjian/article/details/103283319#jump向量算术逻辑运算)

------

### 向量数据类型

- 语法格式

向量类型：

```
<type><size><number_of_lanes>_t
```

向量数组：

```
<type><size><number_of_lanes><length_of_array>_t
```

注：`type`，表示数据类型，可选项：int（带符号整型）、uint（无符号整型）、float（浮点型）和 poly（多项式）。`size`，表示一个元素占用多少 bit。`number_of_lanes`，表示一个向量中包含的元素数量，第一个元素位于 lane[0] 的位置，第 n 个元素位于 lane[n-1] 的位置，lane[0] 对应向量寄存器最低位的`size`个 bit。`length_of_array`，向量数组的大小。

- 示例

| 向量类型    | 含义                                                         |
| ----------- | ------------------------------------------------------------ |
| uint8x8_t   | 一个包含八个元素，元素类型为 8-bit 无符号整型的向量          |
| uint8x8x2_t | 一个包含两个向量的向量数组，每个向量包含八个元素，元素类型为 8-bit 无符号整型 |

------

### 向量赋值

- [向量常数赋值](https://blog.csdn.net/wohenfanjian/article/details/103283319#jump向量常数赋值)
- [向量内存赋值](https://blog.csdn.net/wohenfanjian/article/details/103283319#jump向量内存赋值)
- [向量单个元素内存赋值](https://blog.csdn.net/wohenfanjian/article/details/103283319#jump向量单个元素内存赋值)

------

### 向量常数赋值

- [uint8x8_t 向量常数赋值（每个元素的值都相同）](https://blog.csdn.net/wohenfanjian/article/details/103283319#jumpuint8x8_t向量常数赋值（每个元素的值都相同）)
- [uint8x8_t 向量常数赋值（每个元素的值可以不同）](https://blog.csdn.net/wohenfanjian/article/details/103283319#jumpuint8x8_t向量常数赋值（每个元素的值可以不相同）)



------

#### uint8x8_t 向量常数赋值（每个元素的值都相同）

- 函数原型

```
uint8x8_t vdup_n_u8(uint32_t a);
```

- 作用
  - 将类型为`uint8x8_t`的向量中每个元素的值设置为参数`a`的值。
- 注意事项
  - 参数`a`的值大于 255 时会发生”整型截断“；每个元素的赋值互不影响。
- 代码示例

```
#include "arm_neon.h"

int main()
{
    // 声明一个包含八个元素的向量，每个元素的数据类型为 uint8_t
    uint8x8_t a_uint8x8;
    // 将向量的每个元素赋值为 255
    a_uint8x8 = vdup_n_u8(255);
    return 0;
}
```

注：使用`NEON Intrinsics`需要包含头文件`arm_neon.h`。

- 代码调试

```
; 语句 a_uint8x8 = vdup_n_u8(255); 对应的汇编指令：dup	v0.8b, w0
(gdb) disas
...
0x0000000000400a18 <+40>:	dup	v0.8b, w0
...
; 执行 dup	v0.8b, w0 前，打印 w0 寄存器的值
(gdb) p $w0
$2 = 255
...
; 执行 dup	v0.8b, w0 后，打印 v0 寄存器的值
(gdb) p $v0.b.u
$3 = {255, 255, 255, 255, 255, 255, 255, 255, 0, 0, 0, 0, 0, 0, 0, 0}
(gdb) p $v0.b.u[0]
$4 = 255
(gdb) p $v0.b.u[7]
$5 = 255
(gdb) p $v0.b.u[8]
$6 = 0
```

可以看出 v0 寄存器的低位 8 字节分别被赋值为 255。

[返回上一级](https://blog.csdn.net/wohenfanjian/article/details/103283319#jump向量赋值)

------

#### uint8x8_t 向量常数赋值（每个元素的值可以不同）

- 函数原型

```
uint8x8_t vcreate_u8 (uint64_t __a);
```

- 作用
  - 将参数`a`赋值给类型为`uint8x8_t`的向量。
- 注意事项
  - 赋值顺序：参数`__a`的值从右到左每字节的内容依次赋值给向量的第一个元素到第八个元素（即`lane[0]`到`lane[7]`）。
- 代码示例

```
void print_uint8x8_t(uint8x8_t val)
{
    std::printf("lane0=0x%x\n", vget_lane_u8(val, 0));
    std::printf("lane1=0x%x\n", vget_lane_u8(val, 1));
    std::printf("lane2=0x%x\n", vget_lane_u8(val, 2));
    std::printf("lane3=0x%x\n", vget_lane_u8(val, 3));
    std::printf("lane4=0x%x\n", vget_lane_u8(val, 4));
    std::printf("lane5=0x%x\n", vget_lane_u8(val, 5));
    std::printf("lane6=0x%x\n", vget_lane_u8(val, 6));
    std::printf("lane7=0x%x\n", vget_lane_u8(val, 7));
    std::printf("\n");
}

void assign_different_constant()
{
    std::printf("assign_different_constant function..............\n");

    uint8x8_t a_uint8x8 = vcreate_u8(0x12345678abcdef01);
    print_uint8x8_t(a_uint8x8);
}
```

输出结果为：

```
assign_different_constant function..............
lane0=0x1
lane1=0xef
lane2=0xcd
lane3=0xab
lane4=0x78
lane5=0x56
lane6=0x34
lane7=0x12
```

可以看出，参数`__a`的值从右到左每字节的内容依次赋值给向量的第一个元素到第八个元素。



------

### 向量内存赋值

- [uint8x8_t 向量内存赋值](https://blog.csdn.net/wohenfanjian/article/details/103283319#jumpuint8x8_t向量内存赋值)
- [uint8x8x2_t 向量内存赋值](https://blog.csdn.net/wohenfanjian/article/details/103283319#jumpuint8x8x2_t向量内存赋值)

------

#### uint8x8_t 向量内存赋值

- 函数原型

```
uint8x8_t vld1_u8 (const uint8_t *a);
```

- 作用
  - 将内存起始地址为`a`的后面八字节的内容赋值给类型为`uint8x8_t`的向量。
- 注意事项
  - 如果参数`a`的有效元素数量小于8，那么会发生非法读。
- 代码示例

```
{
    // 源数据的有效元素数量等于8，合法
    uint8_t f_arr_uint8[] = {0xa1, 0xa2, 0xa3, 0xa4, 0xa5, 0xa6, 0xa7, 0xa8};
    uint8x8_t f_uint8x8 = vld1_u8(f_arr_uint8);
    print_uint8x8_t(f_uint8x8);
}

{
    // 源数据的有效元素数量小于8，最后一个元素为非法读
    uint8_t a_arr_uint8[] = {0xa1, 0xa2, 0xa3, 0xa4, 0xa5, 0xa6, 0xa7, 0xa8};
    uint8x8_t a_uint8x8 = vld1_u8(&a_arr_uint8[1]);
    print_uint8x8_t(a_uint8x8);
}

{
    // 源数据的有效元素数量大于8，合法
    uint8_t b_arr_uint8[] = {0xb0, 0xb1, 0xb2, 0xb3, 0xb4, 0xb5, 0xb6, 0xb7, 
                                0xb8, 0xb9, 0xba, 0xbb, 0xbc, 0xbd, 0xbe, 0xbf};
    uint8x8_t b_uint8x8 = vld1_u8(b_arr_uint8);
    print_uint8x8_t(b_uint8x8);
}

{
    // 源数据的有效元素数量小于8，第五个元素到第八个元素为非法读
    uint8_t c_arr_uint8[] = {0xc1, 0xc2, 0xc3, 0xc4};
    uint8x8_t c_uint8x8 = vld1_u8(c_arr_uint8);
    print_uint8x8_t(c_uint8x8);
}

{
    // 源数据的有效元素数量小于8，除第一个元素外其他元素为非法读
    uint8_t d_arr_uint8 = 0xd1;
    uint8x8_t d_uint8x8 = vld1_u8(&d_arr_uint8);
    print_uint8x8_t(d_uint8x8);
}
```

注：关于`print_uint8x8_t`函数的实现可以在其他示例中找到，此处省略。

- 代码调试

```
(gdb) x/b &a_arr_uint8[0]+8
0x7ffffff320:	0xff
; 打印 uint8x8_t a_uint8x8 = vld1_u8(&a_arr_uint8[1]); 执行的结果
(gdb) p/x $v0.b.u
$4 = {0xa2, 0xa3, 0xa4, 0xa5, 0xa6, 0xa7, 0xa8, 0xff, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0}
; 可以看出v0.b.u[7]的值为0xff，即内存地址为&a_arr_uint8[0]+8的值。验证了最后一个元素为非法读。

(gdb) x/4b c_arr_uint8+4
0x7ffffff31c:	0xb4	0xb5	0xb6	0xb7
; 打印 uint8x8_t c_uint8x8 = vld1_u8(c_arr_uint8); 执行的结果
(gdb) display/x $v0.b.u
1: /x $v0.b.u = {0xc1, 0xc2, 0xc3, 0xc4, 0xb4, 0xb5, 0xb6, 0xb7, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0}
; 可以看出v0.b.u[4]、v0.b.u[5]、v0.b.u[6]、v0.b.u[7]的值分别为0xb4、 0xb5、 0xb6、 0xb7，
; 即内存起始地址为c_arr_uint8+4后面4字节的值。验证了第五个元素到第八个元素为非法读。

(gdb) x/7b &d_arr_uint8+1
0x7ffffff2c8:	0x19	0xf3	0xff	0xff	0x7f	0x00	0x00
; 打印 uint8x8_t d_uint8x8 = vld1_u8(&d_arr_uint8); 执行的结果
(gdb) display/x $v0.b.u
1: /x $v0.b.u = {0xd1, 0x19, 0xf3, 0xff, 0xff, 0x7f, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0}
2: /x $v0.b.u = {0xd1, 0x19, 0xf3, 0xff, 0xff, 0x7f, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0}
; 可以看出v0.b.u[1]、v0.b.u[2]、v0.b.u[3]、v0.b.u[4]、v0.b.u[5]、v0.b.u[6]、v0.b.u[7]的值
; 分别为0x19、 0xf3、 0xff、 0xff、 0x7f、 0x0、 0x0，即内存起始地址为&d_arr_uint8+1后面7字节的值。
; 验证了除第一个元素外其他元素为非法读。
```

上述示例验证了：`vld1_u8()函数作用是将参数的值作为内存起始地址，无条件地将其后面的八字节的值赋值给向量。但如果源数据的有效元素数量小于8，会发生非法读`。



------

#### uint8x8x2_t 向量内存赋值

- 函数原型

```
uint8x8x2_t vld2_u8 (const uint8_t * __a);
```

- 作用
  - 内存地址为`__a、__a+2、__a+4、... __a+14`的内容分别赋值给第一个向量的`lane[0]、lane[1]、...、lane[7]`，内存地址为`__a+1、__a+3、__a+5、... __a+15`的内容分别赋值给第二个向量的`lane[0]、lane[1]、...、lane[7]`。
- 代码示例

```
void print_uint8x8_t(uint8x8_t val)
{
    std::printf("lane0=0x%x\n", vget_lane_u8(val, 0));
    std::printf("lane1=0x%x\n", vget_lane_u8(val, 1));
    std::printf("lane2=0x%x\n", vget_lane_u8(val, 2));
    std::printf("lane3=0x%x\n", vget_lane_u8(val, 3));
    std::printf("lane4=0x%x\n", vget_lane_u8(val, 4));
    std::printf("lane5=0x%x\n", vget_lane_u8(val, 5));
    std::printf("lane6=0x%x\n", vget_lane_u8(val, 6));
    std::printf("lane7=0x%x\n", vget_lane_u8(val, 7));
    std::printf("\n");
}

void print_uint8x8x2_t(uint8x8x2_t data)
{
    std::printf("print val[0] ...\n");
    print_uint8x8_t(data.val[0]);
    std::printf("print val[1] ...\n");
    print_uint8x8_t(data.val[1]);
}

void assign_from_mem()
{
    std::printf("assign_from_mem function..............\n");

    uint8_t arr_uint8[] = {0xe1, 0xe2, 0xe3, 0xe4, 0xe5, 0xe6, 0xe7, 0xe8, 
                            0xf1, 0xf2, 0xf3, 0xf4, 0xf5, 0xf6, 0xf7, 0xf8};
    uint8x8x2_t a_uint8x8x2 = vld2_u8(arr_uint8);
    print_uint8x8x2_t(a_uint8x8x2);
}
```

输出结果为：

```
assign_from_mem function..............
print val[0] ...
lane0=0xe1
lane1=0xe3
lane2=0xe5
lane3=0xe7
lane4=0xf1
lane5=0xf3
lane6=0xf5
lane7=0xf7

print val[1] ...
lane0=0xe2
lane1=0xe4
lane2=0xe6
lane3=0xe8
lane4=0xf2
lane5=0xf4
lane6=0xf6
lane7=0xf8
```



------

### 向量单个元素内存赋值

- [uint8x8_t 向量单个元素内存赋值](https://blog.csdn.net/wohenfanjian/article/details/103283319#jumpuint8x8_t向量单个元素内存赋值)

------

### uint8x8_t 向量单个元素内存赋值

- 宏定义

```
#define vld1_lane_u8(a, b, c)
```

- 作用
  - 设置向量中指定元素的值，而不改变其他元素的值。参数`a`为指向数据类型为`uint8_t`的指针；参数`b`为指向数据类型为`uint8x8_t`的向量；参数`c`为向量中元素的索引，合法范围为`0 <= c <=7`。
- 代码示例

```
void print_uint8x8_t(uint8x8_t val)
{
    std::printf("lane0=0x%x\n", vget_lane_u8(val, 0));
    std::printf("lane1=0x%x\n", vget_lane_u8(val, 1));
    std::printf("lane2=0x%x\n", vget_lane_u8(val, 2));
    std::printf("lane3=0x%x\n", vget_lane_u8(val, 3));
    std::printf("lane4=0x%x\n", vget_lane_u8(val, 4));
    std::printf("lane5=0x%x\n", vget_lane_u8(val, 5));
    std::printf("lane6=0x%x\n", vget_lane_u8(val, 6));
    std::printf("lane7=0x%x\n", vget_lane_u8(val, 7));
    std::printf("\n");
}

void assign_lane()
{
    std::printf("assign_lane function..............\n");

    uint8_t a_arr_uint8[] = {0xa1, 0xa2, 0xa3, 0xa4, 0xa5, 0xa6, 0xa7, 0xa8};
    uint8x8_t a_uint8x8;
    
    std::printf("set lane[0] ...\n");
    a_uint8x8 = vld1_lane_u8(a_arr_uint8, a_uint8x8, 0);
    print_uint8x8_t(a_uint8x8);
    
    std::printf("set lane[1] ...\n");
    a_uint8x8 = vld1_lane_u8(a_arr_uint8 + 1, a_uint8x8, 1);
    print_uint8x8_t(a_uint8x8);
    
    std::printf("set lane[2] ...\n");
    a_uint8x8 = vld1_lane_u8(a_arr_uint8 + 2, a_uint8x8, 2);
    print_uint8x8_t(a_uint8x8);
    
    std::printf("set lane[3] ...\n");
    a_uint8x8 = vld1_lane_u8(a_arr_uint8 + 3, a_uint8x8, 3);
    print_uint8x8_t(a_uint8x8);
    
    std::printf("set lane[4] ...\n");
    a_uint8x8 = vld1_lane_u8(a_arr_uint8 + 4, a_uint8x8, 4);
    print_uint8x8_t(a_uint8x8);
    
    std::printf("set lane[5] ...\n");
    a_uint8x8 = vld1_lane_u8(a_arr_uint8 + 5, a_uint8x8, 5);
    print_uint8x8_t(a_uint8x8);
    
    std::printf("set lane[6] ...\n");
    a_uint8x8 = vld1_lane_u8(a_arr_uint8 + 6, a_uint8x8, 6);
    print_uint8x8_t(a_uint8x8);
    
    std::printf("set lane[7] ...\n");
    a_uint8x8 = vld1_lane_u8(a_arr_uint8 + 7, a_uint8x8, 7);
    print_uint8x8_t(a_uint8x8);
}
```

输出结果为：

```
assign_lane function..............
set lane[0] ...
lane0=0xa1
lane1=0xed
lane2=0x21
lane3=0xe3
lane4=0x7f
lane5=0x0
lane6=0x0
lane7=0x0

set lane[1] ...
lane0=0xa1
lane1=0xa2
lane2=0x21
lane3=0xe3
lane4=0x7f
lane5=0x0
lane6=0x0
lane7=0x0

set lane[2] ...
lane0=0xa1
lane1=0xa2
lane2=0xa3
lane3=0xe3
lane4=0x7f
lane5=0x0
lane6=0x0
lane7=0x0

set lane[3] ...
lane0=0xa1
lane1=0xa2
lane2=0xa3
lane3=0xa4
lane4=0x7f
lane5=0x0
lane6=0x0
lane7=0x0

set lane[4] ...
lane0=0xa1
lane1=0xa2
lane2=0xa3
lane3=0xa4
lane4=0xa5
lane5=0x0
lane6=0x0
lane7=0x0

set lane[5] ...
lane0=0xa1
lane1=0xa2
lane2=0xa3
lane3=0xa4
lane4=0xa5
lane5=0xa6
lane6=0x0
lane7=0x0

set lane[6] ...
lane0=0xa1
lane1=0xa2
lane2=0xa3
lane3=0xa4
lane4=0xa5
lane5=0xa6
lane6=0xa7
lane7=0x0

set lane[7] ...
lane0=0xa1
lane1=0xa2
lane2=0xa3
lane3=0xa4
lane4=0xa5
lane5=0xa6
lane6=0xa7
lane7=0xa8
```



------

### 访问/存储向量的值

- [访问 uint8x8_t 的值](https://blog.csdn.net/wohenfanjian/article/details/103283319#jump访问uint8x8_t的值)
- [存储 uint8x8_t 的值](https://blog.csdn.net/wohenfanjian/article/details/103283319#jump存储uint8x8_t的值)

------

### 访问 uint8x8_t 的值

- 函数原型

```
uint8_t vget_lane_u8(uint8x8_t __a, const int __b);
```

- 作用
  - 返回类型为`uint8x8_t`向量（由参数`__a`确定）的第`n`个元素（由参数`__b`确定）的值。
- 注意事项
  - 参数`__b`指明访问向量中第几个元素的值，有效范围为`0 <= b <= 7`。
  - 如果参数`__b`的值不在有效范围，在编译时会报如下错误——`error: lane 8 out of range 0 - 7`。
  - 参数`__b`的值必须在编译期确定。也就是说，参数`__b`只能是常数。如果不是，编译时会报如下错误——`error: lane index must be a constant immediate`。
- 代码示例

```
// 声明一个包含八个元素的向量，每个元素的数据类型为 uint8_t
uint8x8_t a_uint8x8;
// 将向量的每个元素赋值为 255
a_uint8x8 = vdup_n_u8(255);

// 分别打印变量 a_uint8x8 的八个元素的值（十六进制）
std::printf("a_uint8x8: lane0=0x%x\n", vget_lane_u8(a_uint8x8, 0));
std::printf("a_uint8x8: lane1=0x%x\n", vget_lane_u8(a_uint8x8, 1));
std::printf("a_uint8x8: lane2=0x%x\n", vget_lane_u8(a_uint8x8, 2));
std::printf("a_uint8x8: lane3=0x%x\n", vget_lane_u8(a_uint8x8, 3));
std::printf("a_uint8x8: lane4=0x%x\n", vget_lane_u8(a_uint8x8, 4));
std::printf("a_uint8x8: lane5=0x%x\n", vget_lane_u8(a_uint8x8, 5));
std::printf("a_uint8x8: lane6=0x%x\n", vget_lane_u8(a_uint8x8, 6));
std::printf("a_uint8x8: lane7=0x%x\n", vget_lane_u8(a_uint8x8, 7));
```

输出结果为：

```
a_uint8x8: lane0=0xff
a_uint8x8: lane1=0xff
a_uint8x8: lane2=0xff
a_uint8x8: lane3=0xff
a_uint8x8: lane4=0xff
a_uint8x8: lane5=0xff
a_uint8x8: lane6=0xff
a_uint8x8: lane7=0xff
```



------

### 存储 uint8x8_t 的值

- 函数原型

```
void vst1_u8 (uint8_t *a, uint8x8_t b);
```

- 作用
  - 将类型为`uint8x8_t`向量（由参数`b`确定）的值存储到内存起始地址为`a`后面八字节的内存中。
- 注意事项
  - 参数`a`所指向的合法内存必须不小于八字节，否则会产生非法写。
- 代码示例

```
void store_uint8x8_t(uint8x8_t val_uint8x8)
{
    {
        // 目的数据的元素数量等于8，合法
        uint8_t f_arr_uint8[] = {0x1, 0x2, 0x3, 0x4, 0x5, 0x6, 0x7, 0x8};
        std::printf("f_arr_uint8 ...\n");
        vst1_u8(f_arr_uint8, val_uint8x8);
    }

    {
        // 目的数据的元素数量等于8，但参数不是首元素的地址，最后一个元素为非法写
        uint8_t a_arr_uint8[] = {0xa1, 0xa2, 0xa3, 0xa4, 0xa5, 0xa6, 0xa7, 0xa8};
        std::printf("a_arr_uint8 ...\n");
        vst1_u8(&a_arr_uint8[1], val_uint8x8);
    }

    {
        // 目的数据的元素数量大于8，只写数组b_arr_uint8的前八个元素，合法
        uint8_t b_arr_uint8[] = {0xb0, 0xb1, 0xb2, 0xb3, 0xb4, 0xb5, 0xb6, 0xb7, 
                                    0xb8, 0xb9, 0xba, 0xbb, 0xbc, 0xbd, 0xbe, 0xbf};
        std::printf("b_arr_uint8 ...\n");
        vst1_u8(b_arr_uint8, val_uint8x8);
    }

    {
        // 目的数据的元素数量小于8，最后四个元素为非法写
        uint8_t c_arr_uint8[] = {0xc1, 0xc2, 0xc3, 0xc4};
        std::printf("c_arr_uint8 ...\n");
        vst1_u8(c_arr_uint8, val_uint8x8);
    }

    {
        // 目的数据是标量，最后七个元素为非法写
        uint8_t d_arr_uint8 = 0xd1;
        std::printf("d_arr_uint8 ...\n");
        vst1_u8(&d_arr_uint8, val_uint8x8);
    }
}
```

注：参数`val_uint8x8`的值为：{0xf1, 0xf2, 0xf3, 0xf4, 0xf5, 0xf6, 0xf7, 0xf8}，参数`val_uint8x8`的值为：{0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 0x08}。

- 代码调试

```
(gdb) p/x &f_arr_uint8[0]
$1 = 0x7ffffff2c8
(gdb) display/9ubx 0x7ffffff2c8
1: x/9xb 0x7ffffff2c8
0x7ffffff2c8:	0x01	0x02	0x03	0x04	0x05	0x06	0x07	0x08
0x7ffffff2d0:	0xf1
(gdb) n
f_arr_uint8 ...
34	        vst1_u8(f_arr_uint8, val_uint8x8);
1: x/9xb 0x7ffffff2c8
0x7ffffff2c8:	0x01	0x02	0x03	0x04	0x05	0x06	0x07	0x08
0x7ffffff2d0:	0xf1
(gdb) n
39	        uint8_t a_arr_uint8[] = {0xa1, 0xa2, 0xa3, 0xa4, 0xa5, 0xa6, 0xa7, 0xa8};
1: x/9xb 0x7ffffff2c8
0x7ffffff2c8:	0xf1	0xf2	0xf3	0xf4	0xf5	0xf6	0xf7	0xf8
0x7ffffff2d0:	0xf1
; 可以看出，“第九个元素”（0x7ffffff2d0处的内容）没有被修改，验证了目的数据的元素数量等于8，合法。


(gdb) n
40	        std::printf("a_arr_uint8 ...\n");
1: x/9xb 0x7ffffff2c8
0x7ffffff2c8:	0xa1	0xa2	0xa3	0xa4	0xa5	0xa6	0xa7	0xa8
0x7ffffff2d0:	0xf1
(gdb) 
a_arr_uint8 ...
41	        vst1_u8(&a_arr_uint8[1], val_uint8x8);
1: x/9xb 0x7ffffff2c8
0x7ffffff2c8:	0xa1	0xa2	0xa3	0xa4	0xa5	0xa6	0xa7	0xa8
0x7ffffff2d0:	0xf1
(gdb) 
47	                                    0xb8, 0xb9, 0xba, 0xbb, 0xbc, 0xbd, 0xbe, 0xbf};
1: x/9xb 0x7ffffff2c8
0x7ffffff2c8:	0xa1	0xf1	0xf2	0xf3	0xf4	0xf5	0xf6	0xf7
0x7ffffff2d0:	0xf8
; 可以看出，“第八个元素”（0x7ffffff2d0处的内容）的值由0xf1变成了0xf8，但这个元素不是有效的，发生了非法写。


(gdb) n
48	        std::printf("b_arr_uint8 ...\n");
1: x/9xb 0x7ffffff2c8
0x7ffffff2c8:	0xb0	0xb1	0xb2	0xb3	0xb4	0xb5	0xb6	0xb7
0x7ffffff2d0:	0xb8
(gdb) 
b_arr_uint8 ...
49	        vst1_u8(b_arr_uint8, val_uint8x8);
1: x/9xb 0x7ffffff2c8
0x7ffffff2c8:	0xb0	0xb1	0xb2	0xb3	0xb4	0xb5	0xb6	0xb7
0x7ffffff2d0:	0xb8
(gdb) 
54	        uint8_t c_arr_uint8[] = {0xc1, 0xc2, 0xc3, 0xc4};
1: x/9xb 0x7ffffff2c8
0x7ffffff2c8:	0xf1	0xf2	0xf3	0xf4	0xf5	0xf6	0xf7	0xf8
0x7ffffff2d0:	0xb8
(gdb) x/16ubx 0x7ffffff2c8
0x7ffffff2c8:	0xf1	0xf2	0xf3	0xf4	0xf5	0xf6	0xf7	0xf8
0x7ffffff2d0:	0xb8	0xb9	0xba	0xbb	0xbc	0xbd	0xbe	0xbf
; 可以看出，只有b_arr_uint8的前八个元素被修改了，合法


(gdb) n
55	        std::printf("c_arr_uint8 ...\n");
1: x/9xb 0x7ffffff2c8
0x7ffffff2c8:	0xc1	0xc2	0xc3	0xc4	0xf5	0xf6	0xf7	0xf8
0x7ffffff2d0:	0xb8
(gdb) 
c_arr_uint8 ...
56	        vst1_u8(c_arr_uint8, val2_uint8x8);
1: x/9xb 0x7ffffff2c8
0x7ffffff2c8:	0xc1	0xc2	0xc3	0xc4	0xf5	0xf6	0xf7	0xf8
0x7ffffff2d0:	0xb8
(gdb) 
61	        uint8_t d_arr_uint8 = 0xd1;
1: x/9xb 0x7ffffff2c8
0x7ffffff2c8:	0x01	0x02	0x03	0x04	0x05	0x06	0x07	0x08
0x7ffffff2d0:	0xb8
; 可以看出，c_arr_uint8[3]后面的四个元素被修改了，但这几个元素不是有效的，发生了非法写。


(gdb) undisplay 
Delete all auto-display expressions? (y or n) y
(gdb) n
61	        uint8_t d_arr_uint8 = 0xd1;
(gdb) 
62	        std::printf("d_arr_uint8 ...\n");
(gdb) p/x &d_arr_uint8
$1 = 0x7ffffff277
(gdb) display/8ubx 0x7ffffff277
1: x/8xb 0x7ffffff277
0x7ffffff277:	0xd1	0xc9	0xf2	0xff	0xff	0x7f	0x00	0x00
(gdb) n
d_arr_uint8 ...
63	        vst1_u8(&d_arr_uint8, val_uint8x8);
1: x/8xb 0x7ffffff277
0x7ffffff277:	0xd1	0xc9	0xf2	0xff	0xff	0x7f	0x00	0x00
(gdb) 
65	}
1: x/8xb 0x7ffffff277
0x7ffffff277:	0xf1	0xf2	0xf3	0xf4	0xf5	0xf6	0xf7	0xf8
; 可以看出，&d_arr_uint8后面的七个元素被修改了，但这几个元素不是有效的，发生了非法写。
```





------

### 向量算术逻辑运算

- [向量加法运算](https://blog.csdn.net/wohenfanjian/article/details/103283319#jump向量加法运算)

------

### 向量加法运算

- [两个 uint8x8_t 相加](https://blog.csdn.net/wohenfanjian/article/details/103283319#jump两个uint8x8_t相加)

------

### 两个 uint8x8_t 相加

- 函数原型

```
uint8x8_t vadd_u8(uint8x8_t a, uint8x8_t b);
```

- 作用
  - 将两个类型为`uint8x8_t`向量的对应位置元素相加，任意对应位置元素相加的结果（可能会进位或溢出）不会影响其他位置相加的结果。
- 代码示例

```
uint8x8_t a_uint8x8, b_uint8x8;
a_uint8x8 = vdup_n_u8(0x12);
b_uint8x8 = vdup_n_u8(0x34);

uint8x8_t sum_uint8x8;
sum_uint8x8 = vadd_u8(a_uint8x8, b_uint8x8);

// 分别打印变量 sum_uint8x8 的八个元素的值（十六进制）
std::printf("sum_uint8x8: lane0=0x%x\n", vget_lane_u8(sum_uint8x8, 0));
std::printf("sum_uint8x8: lane1=0x%x\n", vget_lane_u8(sum_uint8x8, 1));
std::printf("sum_uint8x8: lane2=0x%x\n", vget_lane_u8(sum_uint8x8, 2));
std::printf("sum_uint8x8: lane3=0x%x\n", vget_lane_u8(sum_uint8x8, 3));
std::printf("sum_uint8x8: lane4=0x%x\n", vget_lane_u8(sum_uint8x8, 4));
std::printf("sum_uint8x8: lane5=0x%x\n", vget_lane_u8(sum_uint8x8, 5));
std::printf("sum_uint8x8: lane6=0x%x\n", vget_lane_u8(sum_uint8x8, 6));
std::printf("sum_uint8x8: lane7=0x%x\n", vget_lane_u8(sum_uint8x8, 7));
```

输出结果为：

```
sum_uint8x8: lane0=0x46
sum_uint8x8: lane1=0x46
sum_uint8x8: lane2=0x46
sum_uint8x8: lane3=0x46
sum_uint8x8: lane4=0x46
sum_uint8x8: lane5=0x46
sum_uint8x8: lane6=0x46
sum_uint8x8: lane7=0x46
```