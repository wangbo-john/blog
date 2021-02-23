# 性能优化篇（6）：NEON优化案例——低阶矩阵乘法（aarch64版）

Author:stormQ

Sunday, 15. December 2019 1 08:21AM

- 目录
  - 使用 Eigen 库实现一个 4 阶方阵的乘法运算
  - 使用 NEON 实现一个 4 阶方阵的乘法运算

------

### 使用 Eigen 库实现一个 4 阶方阵的乘法运算

首先，使用 Eigen 库实现一个 4 阶方阵的乘法运算，作为性能对比测试的参照。完整实现如下：

```
#define ELEMENT_TYPE int32_t
#define ROWS 4
#define COLUMNS 4

typedef Eigen::Matrix MATRIX_A;
typedef Eigen::Matrix MATRIX_B;
typedef Eigen::Matrix MATRIX_C;

void matirx_multi_eigen(const MATRIX_A &mat_1, const MATRIX_B &mat_2, 
                        MATRIX_C &mat_3)
{
    mat_3 = mat_1 * mat_2;
}
```

上述代码中以 4 阶方阵的乘法运算为例，同时也适用于预处理变量`ROWS`和`COLUMNS`的值不相等的情况。比如：`ROWS`的值为 4、`COLUMNS`的值为 5时，`MATRIX_A`是一个 4x5 的矩阵，`MATRIX_B`是一个 5x4 的矩阵，满足矩阵乘法的条件。所以，上述函数仍然适用。

另外，需要注意的是 Eigen 库中的矩阵是以列主序存储的。

------

### 使用 NEON 实现一个 4 阶方阵的乘法运算

使用 NEON 实现一个 4 阶方阵的乘法运算，完整实现如下：

```
void matirx_multi_4x4_int32_neon(const int *mat_1, const int *mat_2, 
                                 int *mat_3, int rows)
{
    int32x4_t a_1 = vld1q_s32(mat_1);
    int32x4_t a_2 = vld1q_s32(mat_1 + rows);
    int32x4_t a_3 = vld1q_s32(mat_1 + 2 * rows);
    int32x4_t a_4 = vld1q_s32(mat_1 + 3 * rows);

    int32x4_t b_1 = vld1q_s32(mat_2);
    int32x4_t b_2 = vld1q_s32(mat_2 + rows);
    int32x4_t b_3 = vld1q_s32(mat_2 + 2 * rows);
    int32x4_t b_4 = vld1q_s32(mat_2 + 3 * rows);

    int32x4_t c_1 = vmulq_n_s32(a_1, vgetq_lane_s32(b_1, 0));
    int32x4_t c_2 = vmulq_n_s32(a_2, vgetq_lane_s32(b_1, 1));
    int32x4_t c_3 = vmulq_n_s32(a_3, vgetq_lane_s32(b_1, 2));
    int32x4_t c_4 = vmulq_n_s32(a_4, vgetq_lane_s32(b_1, 3));
    int32x4_t c_5 = vaddq_s32(c_1, c_2);
    int32x4_t c_6 = vaddq_s32(c_3, c_4);
    int32x4_t c_7 = vaddq_s32(c_5, c_6);
    vst1q_s32(mat_3, c_7);

    c_1 = vmulq_n_s32(a_1, vgetq_lane_s32(b_2, 0));
    c_2 = vmulq_n_s32(a_2, vgetq_lane_s32(b_2, 1));
    c_3 = vmulq_n_s32(a_3, vgetq_lane_s32(b_2, 2));
    c_4 = vmulq_n_s32(a_4, vgetq_lane_s32(b_2, 3));
    c_5 = vaddq_s32(c_1, c_2);
    c_6 = vaddq_s32(c_3, c_4);
    c_7 = vaddq_s32(c_5, c_6);
    vst1q_s32(mat_3 + rows, c_7);

    c_1 = vmulq_n_s32(a_1, vgetq_lane_s32(b_3, 0));
    c_2 = vmulq_n_s32(a_2, vgetq_lane_s32(b_3, 1));
    c_3 = vmulq_n_s32(a_3, vgetq_lane_s32(b_3, 2));
    c_4 = vmulq_n_s32(a_4, vgetq_lane_s32(b_3, 3));
    c_5 = vaddq_s32(c_1, c_2);
    c_6 = vaddq_s32(c_3, c_4);
    c_7 = vaddq_s32(c_5, c_6);
    vst1q_s32(mat_3 + 2 * rows, c_7);

    c_1 = vmulq_n_s32(a_1, vgetq_lane_s32(b_4, 0));
    c_2 = vmulq_n_s32(a_2, vgetq_lane_s32(b_4, 1));
    c_3 = vmulq_n_s32(a_3, vgetq_lane_s32(b_4, 2));
    c_4 = vmulq_n_s32(a_4, vgetq_lane_s32(b_4, 3));
    c_5 = vaddq_s32(c_1, c_2);
    c_6 = vaddq_s32(c_3, c_4);
    c_7 = vaddq_s32(c_5, c_6);
    vst1q_s32(mat_3 + 3 * rows, c_7);
}
```

上述函数`matirx_multi_4x4_int32_neon()`的输入矩阵和输出矩阵都采用列主序的方式存储，与 Eigen 库保持一致。

该函数的实现用到了 5 个`NEON Intrinsics`，分别是`vld1q_s32`、`vgetq_lane_s32`、`vmulq_n_s32`、`vaddq_s32`和`vst1q_s32`，它们都定义在`arm_neon.h`文件中。

`vld1q_s32`的函数原型为`int32x4_t vld1q_s32 (const int32_t *a)`，作用：将内存起始地址为 a 的后面 16 字节的内容赋值给类型为 int32x4_t 的向量。也就是说，在上述函数中`int32x4_t a_1 = vld1q_s32(mat_1);`的作用为：将矩阵 mat_1 的第 1 列元素的值赋值给变量 a_1。以此类推，`int32x4_t a_2 = vld1q_s32(mat_1 + rows);`的作用为：将矩阵 mat_1 的第 2 列元素的值赋值给变量 a_2。

`vgetq_lane_s32`的函数原型为`int32_t vgetq_lane_s32 (int32x4_t __a, const int __b)`，作用：返回类型为 int32x4_t 向量的第n个元素的值。也就是说，在上述函数中`vgetq_lane_s32(b_2, 0)`的作用为：取矩阵 mat_2 中位于第 1 列第 1 行的元素。以此类推，`vgetq_lane_s32(b_2, 1)`的作用为：取矩阵 mat_2 中位于第 1 列第 2 行的元素。

`vmulq_n_s32`的函数原型为`int32x4_t vmulq_n_s32 (int32x4_t a, int32_t b)`，作用：将一个向量类型为 int32x4_t 的变量 a 中的每个元素分别乘以一个数，这个数由参数 b 指定。也就是说，在上述函数中`int32x4_t c_1 = vmulq_n_s32(a_1, vgetq_lane_s32(b_1, 0));`的作用为：将矩阵 mat_1 第 1 列的所有元素分别乘以矩阵 mat_2 的第 1 列第 1 行的元素，并将计算结果赋值给变量 c_1。以此类推，`int32x4_t c_2 = vmulq_n_s32(a_2, vgetq_lane_s32(b_1, 1));`的作用为：将矩阵 mat_1 第 2 列的所有元素分别乘以矩阵 mat_2 的第 1 列第 2 行的元素，并将计算结果赋值给变量 c_2。

`vaddq_s32`的函数原型为`int32x4_t vaddq_s32 (int32x4_t __a, int32x4_t __b)`，作用：将两个类型为 int32x4_t 向量的对应位置元素相加。

`vst1q_s32`的函数原型为`void vst1q_s32 (int32_t *a, int32x4_t b)`，作用：将类型为 int32x4_t 向量（由参数b确定）的值存储到内存起始地址为 a 后面十六字节的内存中。也就是说，在上述函数中`vst1q_s32(mat_3, c_7);`的作用为：设置矩阵 mat_3 第一列所有元素的值。以此类推，`vst1q_s32(mat_3 + rows, c_7);`的作用为：设置矩阵 mat_3 第二列所有元素的值。

下面讲解函数`matirx_multi_4x4_int32_neon()`每部分的实现。

1）加载 4 阶方阵 mat_1

```
int32x4_t a_1 = vld1q_s32(mat_1);
int32x4_t a_2 = vld1q_s32(mat_1 + rows);
int32x4_t a_3 = vld1q_s32(mat_1 + 2 * rows);
int32x4_t a_4 = vld1q_s32(mat_1 + 3 * rows);
```

由于函数`matirx_multi_4x4_int32_neon()`的输入矩阵采用列主序的方式存储。所以，这几行语句用于加载矩阵 mat_1 的每列元素到 4 个类型为 int32x4_t 的向量中，用于后面的计算。

2）加载 4 阶方阵 mat_2

```
int32x4_t b_1 = vld1q_s32(mat_2);
int32x4_t b_2 = vld1q_s32(mat_2 + rows);
int32x4_t b_3 = vld1q_s32(mat_2 + 2 * rows);
int32x4_t b_4 = vld1q_s32(mat_2 + 3 * rows);
```

同样地，这几行语句用于加载矩阵 mat_2 的每列元素到 4 个类型为 int32x4_t 的向量中，用于后面的计算。

3）计算并保存方阵 mat_1、mat_2 乘积的第一列元素

```
int32x4_t c_1 = vmulq_n_s32(a_1, vgetq_lane_s32(b_1, 0));
int32x4_t c_2 = vmulq_n_s32(a_2, vgetq_lane_s32(b_1, 1));
int32x4_t c_3 = vmulq_n_s32(a_3, vgetq_lane_s32(b_1, 2));
int32x4_t c_4 = vmulq_n_s32(a_4, vgetq_lane_s32(b_1, 3));
int32x4_t c_5 = vaddq_s32(c_1, c_2);
int32x4_t c_6 = vaddq_s32(c_3, c_4);
int32x4_t c_7 = vaddq_s32(c_5, c_6);
vst1q_s32(mat_3, c_7);
```

线性代数中矩阵乘法的运算规则为：矩阵 A 与矩阵 B 的乘积是一个矩阵 C。矩阵 C 的行数等于矩阵 A 的行数，矩阵 C 的列数等于矩阵 B 的列数，矩阵 C 的元素 cij（即矩阵 C 中第 i 行第 j 列的元素）等于矩阵 A 的第 i 行元素与矩阵 B 的第 j 列元素的乘积。

变量 c_1 中有 4 个元素，依次为：`a11 * b11`、`a21 * b11`、`a31 * b11`、`a41 * b11`。（aij 表示矩阵 mat_1 的第 i 行第 j 列的元素，bij 表示矩阵 mat_2 的第 i 行第 j 列的元素）

变量 c_2 中有 4 个元素，依次为：`a12 * b21`、`a22 * b21`、`a32 * b21`、`a42 * b21`。

变量 c_3 中有 4 个元素，依次为：`a13 * b31`、`a23 * b31`、`a33 * b31`、`a43 * b31`。

变量 c_4 中有 4 个元素，依次为：`a14 * b41`、`a24 * b41`、`a34 * b41`、`a44 * b41`。

变量 c_7 中有 4 个元素，依次为：`a11 * b11 + a12 * b21 + a13 * b31 + a14 * b41`、`a21 * b11 + a22 * b21 + a23 * b31 + a24 * b41`、`a31 * b11 + a32 * b21 + a33 * b31 + a34 * b41`、`a41 * b11 + a42 * b21 + a43 * b31 + a44 * b41`。

由于 `c11 = a11 * b11 + a12 * b21 + a13 * b31 + a14 * b41`，`c21 = a21 * b11 + a22 * b21 + a23 * b31 + a24 * b41`，`c31 = a31 * b11 + a32 * b21 + a33 * b31 + a34 * b41`，`c41 = a41 * b11 + a42 * b21 + a43 * b31 + a44 * b41`。所以，变量 c_7 中的 4 个元素即 c11、c21、c31、c41。也就是说， 矩阵变量 c_7 为方阵 mat_1、mat_2 乘积的第一列元素。矩阵乘积其他列元素的计算过程与之类似，后面不再赘述。

4）计算并保存方阵 mat_1、mat_2 乘积的第二列元素

```
c_1 = vmulq_n_s32(a_1, vgetq_lane_s32(b_2, 0));
c_2 = vmulq_n_s32(a_2, vgetq_lane_s32(b_2, 1));
c_3 = vmulq_n_s32(a_3, vgetq_lane_s32(b_2, 2));
c_4 = vmulq_n_s32(a_4, vgetq_lane_s32(b_2, 3));
c_5 = vaddq_s32(c_1, c_2);
c_6 = vaddq_s32(c_3, c_4);
c_7 = vaddq_s32(c_5, c_6);
vst1q_s32(mat_3 + rows, c_7);
```

5）计算并保存方阵 mat_1、mat_2 乘积的第三列元素

```
c_1 = vmulq_n_s32(a_1, vgetq_lane_s32(b_3, 0));
c_2 = vmulq_n_s32(a_2, vgetq_lane_s32(b_3, 1));
c_3 = vmulq_n_s32(a_3, vgetq_lane_s32(b_3, 2));
c_4 = vmulq_n_s32(a_4, vgetq_lane_s32(b_3, 3));
c_5 = vaddq_s32(c_1, c_2);
c_6 = vaddq_s32(c_3, c_4);
c_7 = vaddq_s32(c_5, c_6);
vst1q_s32(mat_3 + 2 * rows, c_7);
```

6）计算并保存方阵 mat_1、mat_2 乘积的第四列元素

```
c_1 = vmulq_n_s32(a_1, vgetq_lane_s32(b_4, 0));
c_2 = vmulq_n_s32(a_2, vgetq_lane_s32(b_4, 1));
c_3 = vmulq_n_s32(a_3, vgetq_lane_s32(b_4, 2));
c_4 = vmulq_n_s32(a_4, vgetq_lane_s32(b_4, 3));
c_5 = vaddq_s32(c_1, c_2);
c_6 = vaddq_s32(c_3, c_4);
c_7 = vaddq_s32(c_5, c_6);
vst1q_s32(mat_3 + 3 * rows, c_7);
```

完成程序为`main.cpp`：

```
#include "arm_neon.h"
#include 
#include 

#define ELEMENT_TYPE int32_t
#define ROWS 4
#define COLUMNS 4
#define ITERATION_COUNT 10000

typedef Eigen::Matrix MATRIX_A;
typedef Eigen::Matrix MATRIX_B;
typedef Eigen::Matrix MATRIX_C;

template 
void init_mat(MatrixT &mat, int offset)
{
    for (int i = 0; i < mat.cols(); i++)
    {
        for (int j = 0; j < mat.rows(); j++)
        {
            mat(j, i) = i + j + offset;
        }
    }
}

void init_mat(int *data, int rows, int cols, int offset)
{
    // 列主序
    for (int i = 0; i < cols; ++i)
    {
        for (int j = 0; j < rows; ++j)
        {
            *(data + i * rows + j) = i + j + offset;
        }
    }
}

void matirx_multi_eigen(const MATRIX_A &mat_1, const MATRIX_B &mat_2, 
                        MATRIX_C &mat_3)
{
    mat_3 = mat_1 * mat_2;
}

void matirx_multi_4x4_int32_neon(const int *mat_1, const int *mat_2, 
                                 int *mat_3, int rows)
{
    int32x4_t a_1 = vld1q_s32(mat_1);
    int32x4_t a_2 = vld1q_s32(mat_1 + rows);
    int32x4_t a_3 = vld1q_s32(mat_1 + 2 * rows);
    int32x4_t a_4 = vld1q_s32(mat_1 + 3 * rows);

    int32x4_t b_1 = vld1q_s32(mat_2);
    int32x4_t b_2 = vld1q_s32(mat_2 + rows);
    int32x4_t b_3 = vld1q_s32(mat_2 + 2 * rows);
    int32x4_t b_4 = vld1q_s32(mat_2 + 3 * rows);

    int32x4_t c_1 = vmulq_n_s32(a_1, vgetq_lane_s32(b_1, 0));
    int32x4_t c_2 = vmulq_n_s32(a_2, vgetq_lane_s32(b_1, 1));
    int32x4_t c_3 = vmulq_n_s32(a_3, vgetq_lane_s32(b_1, 2));
    int32x4_t c_4 = vmulq_n_s32(a_4, vgetq_lane_s32(b_1, 3));
    int32x4_t c_5 = vaddq_s32(c_1, c_2);
    int32x4_t c_6 = vaddq_s32(c_3, c_4);
    int32x4_t c_7 = vaddq_s32(c_5, c_6);
    vst1q_s32(mat_3, c_7);

    c_1 = vmulq_n_s32(a_1, vgetq_lane_s32(b_2, 0));
    c_2 = vmulq_n_s32(a_2, vgetq_lane_s32(b_2, 1));
    c_3 = vmulq_n_s32(a_3, vgetq_lane_s32(b_2, 2));
    c_4 = vmulq_n_s32(a_4, vgetq_lane_s32(b_2, 3));
    c_5 = vaddq_s32(c_1, c_2);
    c_6 = vaddq_s32(c_3, c_4);
    c_7 = vaddq_s32(c_5, c_6);
    vst1q_s32(mat_3 + rows, c_7);

    c_1 = vmulq_n_s32(a_1, vgetq_lane_s32(b_3, 0));
    c_2 = vmulq_n_s32(a_2, vgetq_lane_s32(b_3, 1));
    c_3 = vmulq_n_s32(a_3, vgetq_lane_s32(b_3, 2));
    c_4 = vmulq_n_s32(a_4, vgetq_lane_s32(b_3, 3));
    c_5 = vaddq_s32(c_1, c_2);
    c_6 = vaddq_s32(c_3, c_4);
    c_7 = vaddq_s32(c_5, c_6);
    vst1q_s32(mat_3 + 2 * rows, c_7);

    c_1 = vmulq_n_s32(a_1, vgetq_lane_s32(b_4, 0));
    c_2 = vmulq_n_s32(a_2, vgetq_lane_s32(b_4, 1));
    c_3 = vmulq_n_s32(a_3, vgetq_lane_s32(b_4, 2));
    c_4 = vmulq_n_s32(a_4, vgetq_lane_s32(b_4, 3));
    c_5 = vaddq_s32(c_1, c_2);
    c_6 = vaddq_s32(c_3, c_4);
    c_7 = vaddq_s32(c_5, c_6);
    vst1q_s32(mat_3 + 3 * rows, c_7);
}

int main()
{
    MATRIX_A mat_1;
    MATRIX_B mat_2;
    MATRIX_C mat_3;

    init_mat(mat_1, 0);
    init_mat(mat_2, 1);

    int *mat_4 = new int[ROWS * COLUMNS];
    int *mat_5 = new int[COLUMNS * ROWS];
    int *mat_6 = new int[ROWS * ROWS];

    init_mat(mat_4, ROWS, COLUMNS, 0);
    init_mat(mat_5, COLUMNS, ROWS, 1);

    for (int i = 0; i < ITERATION_COUNT; i++)
    {
        matirx_multi_eigen(mat_1, mat_2, mat_3);
    }

    for (int i = 0; i < ITERATION_COUNT; i++)
    {
        matirx_multi_4x4_int32_neon(mat_4, mat_5, mat_6, ROWS);
    }

    delete [] mat_6;
    delete [] mat_5;
    delete [] mat_4;
    return 0;
}
```

编译程序（On Jetson TX2）：

```
g++ -std=c++11 -I /home/test/eigen-3.3.7 -g -Og -o matrix_multi_Og matrix_multi.cpp
```

注：`/home/test/eigen-3.3.7`为 eigen-3.3.7 的源码路径。

统计两个版本的函数执行 10000 次的耗时，统计结果为：

| 启动程序方式 | 第一次执行耗时(us)                                      | 第二次执行耗时(us)                                      | 第三次执行耗时(us)                                      | 第四次执行耗时(us)                                      | 第五次执行耗时(us)                                      |
| :----------- | :------------------------------------------------------ | :------------------------------------------------------ | :------------------------------------------------------ | :------------------------------------------------------ | :------------------------------------------------------ |
| ./main_Og    | matirx_multi_eigen:3063matirx_multi_4x4_int32_neon:1507 | matirx_multi_eigen:3088matirx_multi_4x4_int32_neon:1504 | matirx_multi_eigen:2609matirx_multi_4x4_int32_neon:1026 | matirx_multi_eigen:3066matirx_multi_4x4_int32_neon:1447 | matirx_multi_eigen:2131matirx_multi_4x4_int32_neon:1002 |

从统计结果中可以看出，`matirx_multi_4x4_int32_neon()`函数的执行速度比`matirx_multi_eigen()`函数快 2 倍左右。

Egien 库中也使用了 NEON 指令，这一点可以通过查看 main_Og 的汇编代码得到验证。**那么导致这两个版本的执行速度不同的原因是什么呢？** 下面从 cache 的角度进行分析。

统计 cache 性能数据：

```
--------------------------------------------------------------------------------
I1 cache:         16384 B, 64 B, 4-way associative
D1 cache:         16384 B, 64 B, 4-way associative
LL cache:         262144 B, 64 B, 8-way associative
Command:          ./main_Og
Data file:        cachegrind.out.22152
Events recorded:  Ir I1mr ILmr Dr D1mr DLmr Dw D1mw DLmw
Events shown:     Ir I1mr ILmr Dr D1mr DLmr Dw D1mw DLmw
Event sort order: Ir I1mr ILmr Dr D1mr DLmr Dw D1mw DLmw
Thresholds:       0.1 100 100 100 100 100 100 100 100
Include dirs:     
User annotated:   main.cpp
Auto-annotation:  on

--------------------------------------------------------------------------------
       Ir  I1mr  ILmr        Dr   D1mr  DLmr      Dw  D1mw  DLmw 
--------------------------------------------------------------------------------
3,855,912 1,704 1,434 1,223,769 16,168 8,634 481,210 2,505 1,479  PROGRAM TOTALS

--------------------------------------------------------------------------------
     Ir I1mr ILmr      Dr  D1mr  DLmr     Dw  D1mw DLmw  file:function
--------------------------------------------------------------------------------
600,000    4    4 210,000     0     0 80,000     0    0  /usr/lib/gcc/aarch64-linux-gnu/5/include/arm_neon.h:matirx_multi_eigen(Eigen::Matrix const&, Eigen::Matrix const&, Eigen::Matrix&)
570,000    4    3  80,000     3     0 40,000     1    1  /usr/lib/gcc/aarch64-linux-gnu/5/include/arm_neon.h:matirx_multi_4x4_int32_neon(int const*, int const*, int*, int)
548,097   11   10 149,642 2,055 1,702 50,569    22    9  /build/glibc-BinVK7/glibc-2.23/elf/dl-lookup.c:_dl_lookup_symbol_x
540,520   45   41 192,656 5,120   951 94,214    35    8  /build/glibc-BinVK7/glibc-2.23/elf/dl-lookup.c:do_lookup_x
220,000    2    2 140,000     0     0 50,000     0    0  /home/test/eigen-3.3.7/Eigen/src/Core/CoreEvaluators.h:matirx_multi_eigen(Eigen::Matrix const&, Eigen::Matrix const&, Eigen::Matrix&)
211,533   28   28  48,140 3,389 2,964 20,629 1,745  887  /build/glibc-BinVK7/glibc-2.23/elf/../sysdeps/aarch64/dl-machine.h:_dl_relocate_object
190,079    8    8       9     2     0      7     0    0  /home/test3/main.cpp:main
160,000    1    1  60,000     0     0 40,000     0    0  /home/test3/main.cpp:matirx_multi_eigen(Eigen::Matrix const&, Eigen::Matrix const&, Eigen::Matrix&)
150,000    0    0 150,000     0     0      0     0    0  /home/test/eigen-3.3.7/Eigen/src/Core/arch/NEON/PacketMath.h:matirx_multi_eigen(Eigen::Matrix const&, Eigen::Matrix const&, Eigen::Matrix&)
130,000    1    1  80,000     0     0 40,000     0    0  /home/test/eigen-3.3.7/Eigen/src/Core/AssignEvaluator.h:matirx_multi_eigen(Eigen::Matrix const&, Eigen::Matrix const&, Eigen::Matrix&)
109,117    8    6  37,818   377   243 14,749    21    2  /build/glibc-BinVK7/glibc-2.23/elf/dl-lookup.c:check_match
103,170    7    7  14,700   250    90      0     0    0  /build/glibc-BinVK7/glibc-2.23/string/../sysdeps/aarch64/strcmp.S:strcmp
 56,746    3    3  16,086 2,185 1,019  3,210     2    0  /build/glibc-BinVK7/glibc-2.23/elf/do-rel.h:_dl_relocate_object
 50,000    0    0       0     0     0 30,000     0    0  /home/test/eigen-3.3.7/Eigen/src/Core/ProductEvaluators.h:matirx_multi_eigen(Eigen::Matrix const&, Eigen::Matrix const&, Eigen::Matrix&)
 50,000    0    0       0     0     0      0     0    0  /home/test3/main.cpp:matirx_multi_4x4_int32_neon(int const*, int const*, int*, int)
 44,112    8    8  13,412 1,022   985     12     1    0  /build/glibc-BinVK7/glibc-2.23/elf/dl-addr.c:_dl_addr
 10,800    8    8   3,270     8     0    360     0    0  ???:std::locale::_Impl::_M_install_facet(std::locale::id const*, std::locale::facet const*)
 10,000    0    0       0     0     0      0     0    0  /home/test/eigen-3.3.7/Eigen/src/Core/GeneralProduct.h:Eigen::Product, Eigen::Matrix, 1> const Eigen::MatrixBase >::lazyProduct >(Eigen::MatrixBase > const&) const
  7,273    6    5   3,012   423   142    624     3    1  /build/glibc-BinVK7/glibc-2.23/elf/dl-runtime.c:_dl_fixup
  6,004  236  193   1,528   217    31    574    26   18  ???:???
  5,740    2    2   2,286    74     6  1,136     2    2  /build/glibc-BinVK7/glibc-2.23/elf/dl-misc.c:_dl_name_match_p
  5,504    3    3   1,664     5     4    640     0    0  /build/glibc-BinVK7/glibc-2.23/wcsmbs/btowc.c:btowc

--------------------------------------------------------------------------------
-- Auto-annotated source: /usr/lib/gcc/aarch64-linux-gnu/5/include/arm_neon.h
--------------------------------------------------------------------------------
     Ir I1mr ILmr      Dr D1mr DLmr      Dw D1mw DLmw 

-- line 671 ----------------------------------------
      .    .    .       .    .    .       .    .    .  vaddq_s16 (int16x8_t __a, int16x8_t __b)
      .    .    .       .    .    .       .    .    .  {
      .    .    .       .    .    .       .    .    .    return __a + __b;
      .    .    .       .    .    .       .    .    .  }
      .    .    .       .    .    .       .    .    .  
      .    .    .       .    .    .       .    .    .  __extension__ static __inline int32x4_t __attribute__ ((__always_inline__))
      .    .    .       .    .    .       .    .    .  vaddq_s32 (int32x4_t __a, int32x4_t __b)
      .    .    .       .    .    .       .    .    .  {
120,000    1    0       0    0    0       0    0    0    return __a + __b;
      .    .    .       .    .    .       .    .    .  }
      .    .    .       .    .    .       .    .    .  
      .    .    .       .    .    .       .    .    .  __extension__ static __inline int64x2_t __attribute__ ((__always_inline__))
      .    .    .       .    .    .       .    .    .  vaddq_s64 (int64x2_t __a, int64x2_t __b)
      .    .    .       .    .    .       .    .    .  {
      .    .    .       .    .    .       .    .    .    return __a + __b;
      .    .    .       .    .    .       .    .    .  }
      .    .    .       .    .    .       .    .    .  
-- line 687 ----------------------------------------
-- line 1284 ----------------------------------------
      .    .    .       .    .    .       .    .    .  vmulq_s16 (int16x8_t __a, int16x8_t __b)
      .    .    .       .    .    .       .    .    .  {
      .    .    .       .    .    .       .    .    .    return __a * __b;
      .    .    .       .    .    .       .    .    .  }
      .    .    .       .    .    .       .    .    .  
      .    .    .       .    .    .       .    .    .  __extension__ static __inline int32x4_t __attribute__ ((__always_inline__))
      .    .    .       .    .    .       .    .    .  vmulq_s32 (int32x4_t __a, int32x4_t __b)
      .    .    .       .    .    .       .    .    .  {
 40,000    0    0       0    0    0       0    0    0    return __a * __b;
      .    .    .       .    .    .       .    .    .  }
      .    .    .       .    .    .       .    .    .  
      .    .    .       .    .    .       .    .    .  __extension__ static __inline float32x4_t __attribute__ ((__always_inline__))
      .    .    .       .    .    .       .    .    .  vmulq_f32 (float32x4_t __a, float32x4_t __b)
      .    .    .       .    .    .       .    .    .  {
      .    .    .       .    .    .       .    .    .    return __a * __b;
      .    .    .       .    .    .       .    .    .  }
      .    .    .       .    .    .       .    .    .  
-- line 1300 ----------------------------------------
-- line 2849 ----------------------------------------
      .    .    .       .    .    .       .    .    .  vgetq_lane_s16 (int16x8_t __a, const int __b)
      .    .    .       .    .    .       .    .    .  {
      .    .    .       .    .    .       .    .    .    return __aarch64_vget_lane_any (__a, __b);
      .    .    .       .    .    .       .    .    .  }
      .    .    .       .    .    .       .    .    .  
      .    .    .       .    .    .       .    .    .  __extension__ static __inline int32_t __attribute__ ((__always_inline__))
      .    .    .       .    .    .       .    .    .  vgetq_lane_s32 (int32x4_t __a, const int __b)
      .    .    .       .    .    .       .    .    .  {
160,000    2    2       0    0    0       0    0    0    return __aarch64_vget_lane_any (__a, __b);
      .    .    .       .    .    .       .    .    .  }
      .    .    .       .    .    .       .    .    .  
      .    .    .       .    .    .       .    .    .  __extension__ static __inline int64_t __attribute__ ((__always_inline__))
      .    .    .       .    .    .       .    .    .  vgetq_lane_s64 (int64x2_t __a, const int __b)
      .    .    .       .    .    .       .    .    .  {
      .    .    .       .    .    .       .    .    .    return __aarch64_vget_lane_any (__a, __b);
      .    .    .       .    .    .       .    .    .  }
      .    .    .       .    .    .       .    .    .  
-- line 2865 ----------------------------------------
-- line 6944 ----------------------------------------
      .    .    .       .    .    .       .    .    .  
      .    .    .       .    .    .       .    .    .  __extension__ static __inline int32x4_t __attribute__ ((__always_inline__))
      .    .    .       .    .    .       .    .    .  vmlaq_s32 (int32x4_t a, int32x4_t b, int32x4_t c)
      .    .    .       .    .    .       .    .    .  {
      .    .    .       .    .    .       .    .    .    int32x4_t result;
      .    .    .       .    .    .       .    .    .    __asm__ ("mla %0.4s, %2.4s, %3.4s"
      .    .    .       .    .    .       .    .    .             : "=w"(result)
      .    .    .       .    .    .       .    .    .             : "0"(a), "w"(b), "w"(c)
120,000    0    0       0    0    0       0    0    0             : /* No clobbers */);
      .    .    .       .    .    .       .    .    .    return result;
      .    .    .       .    .    .       .    .    .  }
      .    .    .       .    .    .       .    .    .  
      .    .    .       .    .    .       .    .    .  __extension__ static __inline uint8x16_t __attribute__ ((__always_inline__))
      .    .    .       .    .    .       .    .    .  vmlaq_u8 (uint8x16_t a, uint8x16_t b, uint8x16_t c)
      .    .    .       .    .    .       .    .    .  {
      .    .    .       .    .    .       .    .    .    uint8x16_t result;
      .    .    .       .    .    .       .    .    .    __asm__ ("mla %0.16b, %2.16b, %3.16b"
-- line 6960 ----------------------------------------
-- line 8478 ----------------------------------------
      .    .    .       .    .    .       .    .    .  
      .    .    .       .    .    .       .    .    .  __extension__ static __inline int32x4_t __attribute__ ((__always_inline__))
      .    .    .       .    .    .       .    .    .  vmulq_n_s32 (int32x4_t a, int32_t b)
      .    .    .       .    .    .       .    .    .  {
      .    .    .       .    .    .       .    .    .    int32x4_t result;
      .    .    .       .    .    .       .    .    .    __asm__ ("mul %0.4s,%1.4s,%2.s[0]"
      .    .    .       .    .    .       .    .    .             : "=w"(result)
      .    .    .       .    .    .       .    .    .             : "w"(a), "w"(b)
160,000    0    0       0    0    0       0    0    0             : /* No clobbers */);
      .    .    .       .    .    .       .    .    .    return result;
      .    .    .       .    .    .       .    .    .  }
      .    .    .       .    .    .       .    .    .  
      .    .    .       .    .    .       .    .    .  __extension__ static __inline uint16x8_t __attribute__ ((__always_inline__))
      .    .    .       .    .    .       .    .    .  vmulq_n_u16 (uint16x8_t a, uint16_t b)
      .    .    .       .    .    .       .    .    .  {
      .    .    .       .    .    .       .    .    .    uint16x8_t result;
      .    .    .       .    .    .       .    .    .    __asm__ ("mul %0.8h,%1.8h,%2.h[0]"
-- line 8494 ----------------------------------------
-- line 14183 ----------------------------------------
      .    .    .       .    .    .       .    .    .  vdupq_n_s16 (int32_t __a)
      .    .    .       .    .    .       .    .    .  {
      .    .    .       .    .    .       .    .    .    return (int16x8_t) {__a, __a, __a, __a, __a, __a, __a, __a};
      .    .    .       .    .    .       .    .    .  }
      .    .    .       .    .    .       .    .    .  
      .    .    .       .    .    .       .    .    .  __extension__ static __inline int32x4_t __attribute__ ((__always_inline__))
      .    .    .       .    .    .       .    .    .  vdupq_n_s32 (int32_t __a)
      .    .    .       .    .    .       .    .    .  {
160,000    2    2  10,000    0    0       0    0    0    return (int32x4_t) {__a, __a, __a, __a};
      .    .    .       .    .    .       .    .    .  }
      .    .    .       .    .    .       .    .    .  
      .    .    .       .    .    .       .    .    .  __extension__ static __inline int64x2_t __attribute__ ((__always_inline__))
      .    .    .       .    .    .       .    .    .  vdupq_n_s64 (int64_t __a)
      .    .    .       .    .    .       .    .    .  {
      .    .    .       .    .    .       .    .    .    return (int64x2_t) {__a, __a};
      .    .    .       .    .    .       .    .    .  }
      .    .    .       .    .    .       .    .    .  
-- line 14199 ----------------------------------------
-- line 15363 ----------------------------------------
      .    .    .       .    .    .       .    .    .  vld1q_s16 (const int16_t *a)
      .    .    .       .    .    .       .    .    .  {
      .    .    .       .    .    .       .    .    .    return __builtin_aarch64_ld1v8hi ((const __builtin_aarch64_simd_hi *) a);
      .    .    .       .    .    .       .    .    .  }
      .    .    .       .    .    .       .    .    .  
      .    .    .       .    .    .       .    .    .  __extension__ static __inline int32x4_t __attribute__ ((__always_inline__))
      .    .    .       .    .    .       .    .    .  vld1q_s32 (const int32_t *a)
      .    .    .       .    .    .       .    .    .  {
280,000    2    2 280,000    3    0       0    0    0    return __builtin_aarch64_ld1v4si ((const __builtin_aarch64_simd_si *) a);
      .    .    .       .    .    .       .    .    .  }
      .    .    .       .    .    .       .    .    .  
      .    .    .       .    .    .       .    .    .  __extension__ static __inline int64x2_t __attribute__ ((__always_inline__))
      .    .    .       .    .    .       .    .    .  vld1q_s64 (const int64_t *a)
      .    .    .       .    .    .       .    .    .  {
      .    .    .       .    .    .       .    .    .    return __builtin_aarch64_ld1v2di ((const __builtin_aarch64_simd_di *) a);
      .    .    .       .    .    .       .    .    .  }
      .    .    .       .    .    .       .    .    .  
-- line 15379 ----------------------------------------
-- line 22902 ----------------------------------------
      .    .    .       .    .    .       .    .    .  vst1q_s16 (int16_t *a, int16x8_t b)
      .    .    .       .    .    .       .    .    .  {
      .    .    .       .    .    .       .    .    .    __builtin_aarch64_st1v8hi ((__builtin_aarch64_simd_hi *) a, b);
      .    .    .       .    .    .       .    .    .  }
      .    .    .       .    .    .       .    .    .  
      .    .    .       .    .    .       .    .    .  __extension__ static __inline void __attribute__ ((__always_inline__))
      .    .    .       .    .    .       .    .    .  vst1q_s32 (int32_t *a, int32x4_t b)
      .    .    .       .    .    .       .    .    .  {
130,000    1    1       0    0    0 120,000    1    1    __builtin_aarch64_st1v4si ((__builtin_aarch64_simd_si *) a, b);
      .    .    .       .    .    .       .    .    .  }
      .    .    .       .    .    .       .    .    .  
      .    .    .       .    .    .       .    .    .  __extension__ static __inline void __attribute__ ((__always_inline__))
      .    .    .       .    .    .       .    .    .  vst1q_s64 (int64_t *a, int64x2_t b)
      .    .    .       .    .    .       .    .    .  {
      .    .    .       .    .    .       .    .    .    __builtin_aarch64_st1v2di ((__builtin_aarch64_simd_di *) a, b);
      .    .    .       .    .    .       .    .    .  }
      .    .    .       .    .    .       .    .    .  
-- line 22918 ----------------------------------------

--------------------------------------------------------------------------------
-- Auto-annotated source: /home/test3/main.cpp
--------------------------------------------------------------------------------
    Ir I1mr ILmr     Dr D1mr DLmr     Dw D1mw DLmw 

-- line 12 ----------------------------------------
     .    .    .      .    .    .      .    .    .  #define COLUMNS ROWS
     .    .    .      .    .    .      .    .    .  #define ITERATION_COUNT 10000
     .    .    .      .    .    .      .    .    .  
     .    .    .      .    .    .      .    .    .  typedef Eigen::Matrix MATRIX_A;
     .    .    .      .    .    .      .    .    .  typedef Eigen::Matrix MATRIX_B;
     .    .    .      .    .    .      .    .    .  typedef Eigen::Matrix MATRIX_C;
     .    .    .      .    .    .      .    .    .  
     .    .    .      .    .    .      .    .    .  template 
     4    1    1      0    0    0      4    0    0  void init_mat(MatrixT &mat, int offset)
     .    .    .      .    .    .      .    .    .  {
    46    0    0      0    0    0      0    0    0      for (int i = 0; i < mat.cols(); i++)
     .    .    .      .    .    .      .    .    .      {
   144    1    1      0    0    0      0    0    0          for (int j = 0; j < mat.rows(); j++)
     .    .    .      .    .    .      .    .    .          {
   160    0    0      0    0    0     32    0    0              mat(j, i) = i + j + offset;
     .    .    .      .    .    .      .    .    .          }
     .    .    .      .    .    .      .    .    .      }
     4    0    0      4    0    0      0    0    0  }
     .    .    .      .    .    .      .    .    .  
     .    .    .      .    .    .      .    .    .  void init_mat(int *data, int rows, int cols, int offset)
     .    .    .      .    .    .      .    .    .  {
     .    .    .      .    .    .      .    .    .      // 列主序
    48    0    0      0    0    0      0    0    0      for (int i = 0; i < cols; ++i)
     .    .    .      .    .    .      .    .    .      {
   144    0    0      0    0    0      0    0    0          for (int j = 0; j < rows; ++j)
     .    .    .      .    .    .      .    .    .          {
   192    1    1      0    0    0     32    0    0              *(data + i * rows + j) = i + j + offset;
     .    .    .      .    .    .      .    .    .          }
     .    .    .      .    .    .      .    .    .      }
     .    .    .      .    .    .      .    .    .  }
     .    .    .      .    .    .      .    .    .  
     .    .    .      .    .    .      .    .    .  void matirx_multi_eigen(const MATRIX_A &mat_1, const MATRIX_B &mat_2, 
     .    .    .      .    .    .      .    .    .                          MATRIX_C &mat_3)
80,000    1    1 10,000    0    0 40,000    0    0  {
     .    .    .      .    .    .      .    .    .      mat_3 = mat_1 * mat_2;
80,000    0    0 50,000    0    0      0    0    0  }
     .    .    .      .    .    .      .    .    .  
     .    .    .      .    .    .      .    .    .  void matirx_multi_4x4_int32_neon(const int *mat_1, const int *mat_2, 
     .    .    .      .    .    .      .    .    .                                   int *mat_3, int rows)
     .    .    .      .    .    .      .    .    .  {
     .    .    .      .    .    .      .    .    .      int32x4_t a_1 = vld1q_s32(mat_1);
20,000    0    0      0    0    0      0    0    0      int32x4_t a_2 = vld1q_s32(mat_1 + rows);
10,000    0    0      0    0    0      0    0    0      int32x4_t a_3 = vld1q_s32(mat_1 + 2 * rows);
20,000    0    0      0    0    0      0    0    0      int32x4_t a_4 = vld1q_s32(mat_1 + 3 * rows);
     .    .    .      .    .    .      .    .    .  
     .    .    .      .    .    .      .    .    .      int32x4_t b_1 = vld1q_s32(mat_2);
     .    .    .      .    .    .      .    .    .      int32x4_t b_2 = vld1q_s32(mat_2 + rows);
     .    .    .      .    .    .      .    .    .      int32x4_t b_3 = vld1q_s32(mat_2 + 2 * rows);
     .    .    .      .    .    .      .    .    .      int32x4_t b_4 = vld1q_s32(mat_2 + 3 * rows);
     .    .    .      .    .    .      .    .    .  
     .    .    .      .    .    .      .    .    .      int32x4_t c_1 = vmulq_n_s32(a_1, vgetq_lane_s32(b_1, 0));
     .    .    .      .    .    .      .    .    .      int32x4_t c_2 = vmulq_n_s32(a_2, vgetq_lane_s32(b_1, 1));
-- line 63 ----------------------------------------
-- line 92 ----------------------------------------
     .    .    .      .    .    .      .    .    .      c_4 = vmulq_n_s32(a_4, vgetq_lane_s32(b_4, 3));
     .    .    .      .    .    .      .    .    .      c_5 = vaddq_s32(c_1, c_2);
     .    .    .      .    .    .      .    .    .      c_6 = vaddq_s32(c_3, c_4);
     .    .    .      .    .    .      .    .    .      c_7 = vaddq_s32(c_5, c_6);
     .    .    .      .    .    .      .    .    .      vst1q_s32(mat_3 + 3 * rows, c_7);
     .    .    .      .    .    .      .    .    .  }
     .    .    .      .    .    .      .    .    .  
     .    .    .      .    .    .      .    .    .  int main()
     8    2    2      1    0    0      7    0    0  {
     .    .    .      .    .    .      .    .    .      MATRIX_A mat_1;
     .    .    .      .    .    .      .    .    .      MATRIX_B mat_2;
     .    .    .      .    .    .      .    .    .      MATRIX_C mat_3;
     .    .    .      .    .    .      .    .    .  
     2    0    0      0    0    0      0    0    0      init_mat(mat_1, 0);
     3    0    0      0    0    0      0    0    0      init_mat(mat_2, 1);
     .    .    .      .    .    .      .    .    .  
     3    0    0      0    0    0      0    0    0      int *mat_4 = new int[ROWS * COLUMNS];
     3    0    0      0    0    0      0    0    0      int *mat_5 = new int[COLUMNS * ROWS];
     3    0    0      0    0    0      0    0    0      int *mat_6 = new int[ROWS * ROWS];
     .    .    .      .    .    .      .    .    .  
     5    1    1      0    0    0      0    0    0      init_mat(mat_4, ROWS, COLUMNS, 0);
     5    0    0      0    0    0      0    0    0      init_mat(mat_5, COLUMNS, ROWS, 1);
     .    .    .      .    .    .      .    .    .  
50,004    1    1      0    0    0      0    0    0      for (int i = 0; i < ITERATION_COUNT; i++)
     .    .    .      .    .    .      .    .    .      {
40,000    0    0      0    0    0      0    0    0           matirx_multi_eigen(mat_1, mat_2, mat_3);
     .    .    .      .    .    .      .    .    .      }
     .    .    .      .    .    .      .    .    .  
50,004    1    1      0    0    0      0    0    0      for (int i = 0; i < ITERATION_COUNT; i++)
     .    .    .      .    .    .      .    .    .      {
50,000    0    0      0    0    0      0    0    0           matirx_multi_4x4_int32_neon(mat_4, mat_5, mat_6, ROWS);
     .    .    .      .    .    .      .    .    .      }
     .    .    .      .    .    .      .    .    .  
     3    0    0      0    0    0      0    0    0      delete [] mat_6;
     3    1    1      0    0    0      0    0    0      delete [] mat_5;
     3    0    0      0    0    0      0    0    0      delete [] mat_4;
     .    .    .      .    .    .      .    .    .      return 0;
    28    4    3     13    2    0      5    0    0  }

--------------------------------------------------------------------------------
-- User-annotated source: main.cpp
--------------------------------------------------------------------------------
  No information has been collected for main.cpp

--------------------------------------------------------------------------------
-- Auto-annotated source: /home/test/eigen-3.3.7/Eigen/src/Core/ProductEvaluators.h
--------------------------------------------------------------------------------
    Ir I1mr ILmr Dr D1mr DLmr     Dw D1mw DLmw 

-- line 389 ----------------------------------------
     .    .    .  .    .    .      .    .    .  {
     .    .    .  .    .    .      .    .    .    typedef typename Product::Scalar Scalar;
     .    .    .  .    .    .      .    .    .    
     .    .    .  .    .    .      .    .    .    template
     .    .    .  .    .    .      .    .    .    static EIGEN_STRONG_INLINE void evalTo(Dst& dst, const Lhs& lhs, const Rhs& rhs)
     .    .    .  .    .    .      .    .    .    {
     .    .    .  .    .    .      .    .    .      // Same as: dst.noalias() = lhs.lazyProduct(rhs);
     .    .    .  .    .    .      .    .    .      // but easier on the compiler side
10,000    0    0  0    0    0      0    0    0      call_assignment_no_alias(dst, lhs.lazyProduct(rhs), internal::assign_op());
     .    .    .  .    .    .      .    .    .    }
     .    .    .  .    .    .      .    .    .    
     .    .    .  .    .    .      .    .    .    template
     .    .    .  .    .    .      .    .    .    static EIGEN_STRONG_INLINE void addTo(Dst& dst, const Lhs& lhs, const Rhs& rhs)
     .    .    .  .    .    .      .    .    .    {
     .    .    .  .    .    .      .    .    .      // dst.noalias() += lhs.lazyProduct(rhs);
     .    .    .  .    .    .      .    .    .      call_assignment_no_alias(dst, lhs.lazyProduct(rhs), internal::add_assign_op());
     .    .    .  .    .    .      .    .    .    }
-- line 405 ----------------------------------------
-- line 443 ----------------------------------------
     .    .    .  .    .    .      .    .    .  
     .    .    .  .    .    .      .    .    .    EIGEN_DEVICE_FUNC EIGEN_STRONG_INLINE
     .    .    .  .    .    .      .    .    .    explicit product_evaluator(const XprType& xpr)
     .    .    .  .    .    .      .    .    .      : m_lhs(xpr.lhs()),
     .    .    .  .    .    .      .    .    .        m_rhs(xpr.rhs()),
     .    .    .  .    .    .      .    .    .        m_lhsImpl(m_lhs),     // FIXME the creation of the evaluator objects should result in a no-op, but check that!
     .    .    .  .    .    .      .    .    .        m_rhsImpl(m_rhs),     //       Moreover, they are only useful for the packet path, so we could completely disable them when not needed,
     .    .    .  .    .    .      .    .    .                              //       or perhaps declare them on the fly on the packet method... We have experiment to check what's best.
40,000    0    0  0    0    0 30,000    0    0        m_innerDim(xpr.lhs().cols())
     .    .    .  .    .    .      .    .    .    {
     .    .    .  .    .    .      .    .    .      EIGEN_INTERNAL_CHECK_COST_VALUE(NumTraits::MulCost);
     .    .    .  .    .    .      .    .    .      EIGEN_INTERNAL_CHECK_COST_VALUE(NumTraits::AddCost);
     .    .    .  .    .    .      .    .    .      EIGEN_INTERNAL_CHECK_COST_VALUE(CoeffReadCost);
     .    .    .  .    .    .      .    .    .  #if 0
     .    .    .  .    .    .      .    .    .      std::cerr << "LhsOuterStrideBytes=  " << LhsOuterStrideBytes << "\n";
     .    .    .  .    .    .      .    .    .      std::cerr << "RhsOuterStrideBytes=  " << RhsOuterStrideBytes << "\n";
     .    .    .  .    .    .      .    .    .      std::cerr << "LhsAlignment=         " << LhsAlignment << "\n";
-- line 459 ----------------------------------------

--------------------------------------------------------------------------------
-- Auto-annotated source: /home/test/eigen-3.3.7/Eigen/src/Core/AssignEvaluator.h
--------------------------------------------------------------------------------
    Ir I1mr ILmr     Dr D1mr DLmr     Dw D1mw DLmw 

-- line 603 ----------------------------------------
     .    .    .      .    .    .      .    .    .    typedef DstEvaluatorTypeT DstEvaluatorType;
     .    .    .      .    .    .      .    .    .    typedef SrcEvaluatorTypeT SrcEvaluatorType;
     .    .    .      .    .    .      .    .    .    typedef typename DstEvaluatorType::Scalar Scalar;
     .    .    .      .    .    .      .    .    .    typedef copy_using_evaluator_traits AssignmentTraits;
     .    .    .      .    .    .      .    .    .    typedef typename AssignmentTraits::PacketType PacketType;
     .    .    .      .    .    .      .    .    .    
     .    .    .      .    .    .      .    .    .    
     .    .    .      .    .    .      .    .    .    EIGEN_DEVICE_FUNC generic_dense_assignment_kernel(DstEvaluatorType &dst, const SrcEvaluatorType &src, const Functor &func, DstXprType& dstExpr)
50,000    1    1      0    0    0 40,000    0    0      : m_dst(dst), m_src(src), m_functor(func), m_dstExpr(dstExpr)
     .    .    .      .    .    .      .    .    .    {
     .    .    .      .    .    .      .    .    .      #ifdef EIGEN_DEBUG_ASSIGN
     .    .    .      .    .    .      .    .    .      AssignmentTraits::debug();
     .    .    .      .    .    .      .    .    .      #endif
     .    .    .      .    .    .      .    .    .    }
     .    .    .      .    .    .      .    .    .    
     .    .    .      .    .    .      .    .    .    EIGEN_DEVICE_FUNC Index size() const        { return m_dstExpr.size(); }
     .    .    .      .    .    .      .    .    .    EIGEN_DEVICE_FUNC Index innerSize() const   { return m_dstExpr.innerSize(); }
-- line 619 ----------------------------------------
-- line 644 ----------------------------------------
     .    .    .      .    .    .      .    .    .      Index col = colIndexByOuterInner(outer, inner); 
     .    .    .      .    .    .      .    .    .      assignCoeff(row, col);
     .    .    .      .    .    .      .    .    .    }
     .    .    .      .    .    .      .    .    .    
     .    .    .      .    .    .      .    .    .    
     .    .    .      .    .    .      .    .    .    template
     .    .    .      .    .    .      .    .    .    EIGEN_DEVICE_FUNC EIGEN_STRONG_INLINE void assignPacket(Index row, Index col)
     .    .    .      .    .    .      .    .    .    {
80,000    0    0 80,000    0    0      0    0    0      m_functor.template assignPacket(&m_dst.coeffRef(row,col), m_src.template packet(row,col));
     .    .    .      .    .    .      .    .    .    }
     .    .    .      .    .    .      .    .    .    
     .    .    .      .    .    .      .    .    .    template
     .    .    .      .    .    .      .    .    .    EIGEN_DEVICE_FUNC EIGEN_STRONG_INLINE void assignPacket(Index index)
     .    .    .      .    .    .      .    .    .    {
     .    .    .      .    .    .      .    .    .      m_functor.template assignPacket(&m_dst.coeffRef(index), m_src.template packet(index));
     .    .    .      .    .    .      .    .    .    }
     .    .    .      .    .    .      .    .    .    
-- line 660 ----------------------------------------

--------------------------------------------------------------------------------
-- Auto-annotated source: /home/test/eigen-3.3.7/Eigen/src/Core/CoreEvaluators.h
--------------------------------------------------------------------------------
    Ir I1mr ILmr     Dr D1mr DLmr     Dw D1mw DLmw 

-- line 147 ----------------------------------------
     .    .    .      .    .    .      .    .    .        m_outerStride(IsVectorAtCompileTime  ? 0 
     .    .    .      .    .    .      .    .    .                                             : int(IsRowMajor) ? ColsAtCompileTime 
     .    .    .      .    .    .      .    .    .                                             : RowsAtCompileTime)
     .    .    .      .    .    .      .    .    .    {
     .    .    .      .    .    .      .    .    .      EIGEN_INTERNAL_CHECK_COST_VALUE(CoeffReadCost);
     .    .    .      .    .    .      .    .    .    }
     .    .    .      .    .    .      .    .    .    
     .    .    .      .    .    .      .    .    .    EIGEN_DEVICE_FUNC explicit evaluator(const PlainObjectType& m)
80,000    1    1      0    0    0 50,000    0    0      : m_data(m.data()), m_outerStride(IsVectorAtCompileTime ? 0 : m.outerStride()) 
     .    .    .      .    .    .      .    .    .    {
     .    .    .      .    .    .      .    .    .      EIGEN_INTERNAL_CHECK_COST_VALUE(CoeffReadCost);
     .    .    .      .    .    .      .    .    .    }
     .    .    .      .    .    .      .    .    .  
     .    .    .      .    .    .      .    .    .    EIGEN_DEVICE_FUNC EIGEN_STRONG_INLINE
     .    .    .      .    .    .      .    .    .    CoeffReturnType coeff(Index row, Index col) const
     .    .    .      .    .    .      .    .    .    {
     .    .    .      .    .    .      .    .    .      if (IsRowMajor)
     .    .    .      .    .    .      .    .    .        return m_data[row * m_outerStride.value() + col];
     .    .    .      .    .    .      .    .    .      else
40,000    1    1 40,000    0    0      0    0    0        return m_data[row + col * m_outerStride.value()];
     .    .    .      .    .    .      .    .    .    }
     .    .    .      .    .    .      .    .    .  
     .    .    .      .    .    .      .    .    .    EIGEN_DEVICE_FUNC EIGEN_STRONG_INLINE
     .    .    .      .    .    .      .    .    .    CoeffReturnType coeff(Index index) const
     .    .    .      .    .    .      .    .    .    {
     .    .    .      .    .    .      .    .    .      return m_data[index];
     .    .    .      .    .    .      .    .    .    }
     .    .    .      .    .    .      .    .    .  
     .    .    .      .    .    .      .    .    .    EIGEN_DEVICE_FUNC EIGEN_STRONG_INLINE
     .    .    .      .    .    .      .    .    .    Scalar& coeffRef(Index row, Index col)
     .    .    .      .    .    .      .    .    .    {
     .    .    .      .    .    .      .    .    .      if (IsRowMajor)
     .    .    .      .    .    .      .    .    .        return const_cast(m_data)[row * m_outerStride.value() + col];
     .    .    .      .    .    .      .    .    .      else
50,032    1    1 50,000    0    0      0    0    0        return const_cast(m_data)[row + col * m_outerStride.value()];
     .    .    .      .    .    .      .    .    .    }
     .    .    .      .    .    .      .    .    .  
     .    .    .      .    .    .      .    .    .    EIGEN_DEVICE_FUNC EIGEN_STRONG_INLINE
     .    .    .      .    .    .      .    .    .    Scalar& coeffRef(Index index)
     .    .    .      .    .    .      .    .    .    {
     .    .    .      .    .    .      .    .    .      return const_cast(m_data)[index];
     .    .    .      .    .    .      .    .    .    }
     .    .    .      .    .    .      .    .    .  
     .    .    .      .    .    .      .    .    .    template
     .    .    .      .    .    .      .    .    .    EIGEN_STRONG_INLINE
     .    .    .      .    .    .      .    .    .    PacketType packet(Index row, Index col) const
     .    .    .      .    .    .      .    .    .    {
     .    .    .      .    .    .      .    .    .      if (IsRowMajor)
     .    .    .      .    .    .      .    .    .        return ploadt(m_data + row * m_outerStride.value() + col);
     .    .    .      .    .    .      .    .    .      else
50,000    0    0 50,000    0    0      0    0    0        return ploadt(m_data + row + col * m_outerStride.value());
     .    .    .      .    .    .      .    .    .    }
     .    .    .      .    .    .      .    .    .  
     .    .    .      .    .    .      .    .    .    template
     .    .    .      .    .    .      .    .    .    EIGEN_STRONG_INLINE
     .    .    .      .    .    .      .    .    .    PacketType packet(Index index) const
     .    .    .      .    .    .      .    .    .    {
     .    .    .      .    .    .      .    .    .      return ploadt(m_data + index);
     .    .    .      .    .    .      .    .    .    }
-- line 205 ----------------------------------------

--------------------------------------------------------------------------------
-- Auto-annotated source: /home/test/eigen-3.3.7/Eigen/src/Core/GeneralProduct.h
--------------------------------------------------------------------------------
    Ir I1mr ILmr Dr D1mr DLmr Dw D1mw DLmw 

-- line 443 ----------------------------------------
     .    .    .  .    .    .  .    .    .    //    * for a coeff-wise product use: v1.cwiseProduct(v2)
     .    .    .  .    .    .  .    .    .    EIGEN_STATIC_ASSERT(ProductIsValid || !(AreVectors && SameSizes),
     .    .    .  .    .    .  .    .    .      INVALID_VECTOR_VECTOR_PRODUCT__IF_YOU_WANTED_A_DOT_OR_COEFF_WISE_PRODUCT_YOU_MUST_USE_THE_EXPLICIT_FUNCTIONS)
     .    .    .  .    .    .  .    .    .    EIGEN_STATIC_ASSERT(ProductIsValid || !(SameSizes && !AreVectors),
     .    .    .  .    .    .  .    .    .      INVALID_MATRIX_PRODUCT__IF_YOU_WANTED_A_COEFF_WISE_PRODUCT_YOU_MUST_USE_THE_EXPLICIT_FUNCTION)
     .    .    .  .    .    .  .    .    .    EIGEN_STATIC_ASSERT(ProductIsValid || SameSizes, INVALID_MATRIX_PRODUCT)
     .    .    .  .    .    .  .    .    .  
     .    .    .  .    .    .  .    .    .    return Product(derived(), other.derived());
10,000    0    0  0    0    0  0    0    0  }
     .    .    .  .    .    .  .    .    .  
     .    .    .  .    .    .  .    .    .  } // end namespace Eigen
     .    .    .  .    .    .  .    .    .  
     .    .    .  .    .    .  .    .    .  #endif // EIGEN_PRODUCT_H

--------------------------------------------------------------------------------
-- Auto-annotated source: /home/test/eigen-3.3.7/Eigen/src/Core/arch/NEON/PacketMath.h
--------------------------------------------------------------------------------
     Ir I1mr ILmr      Dr D1mr DLmr Dw D1mw DLmw 

-- line 138 ----------------------------------------
      .    .    .       .    .    .  .    .    .  EIGEN_STRONG_INLINE void        vst1q_f32(float* to, float32x4_t from) { ::vst1q_f32((float32_t*)to,from); }
      .    .    .       .    .    .  .    .    .  EIGEN_STRONG_INLINE void        vst1_f32 (float* to, float32x2_t from) { ::vst1_f32 ((float32_t*)to,from); }
      .    .    .       .    .    .  .    .    .  #endif
      .    .    .       .    .    .  .    .    .  
      .    .    .       .    .    .  .    .    .  template<> struct unpacket_traits { typedef float   type; enum {size=4, alignment=Aligned16}; typedef Packet4f half; };
      .    .    .       .    .    .  .    .    .  template<> struct unpacket_traits { typedef int32_t type; enum {size=4, alignment=Aligned16}; typedef Packet4i half; };
      .    .    .       .    .    .  .    .    .  
      .    .    .       .    .    .  .    .    .  template<> EIGEN_STRONG_INLINE Packet4f pset1(const float&  from) { return vdupq_n_f32(from); }
150,000    0    0 150,000    0    0  0    0    0  template<> EIGEN_STRONG_INLINE Packet4i pset1(const int32_t&    from)   { return vdupq_n_s32(from); }
      .    .    .       .    .    .  .    .    .  
      .    .    .       .    .    .  .    .    .  template<> EIGEN_STRONG_INLINE Packet4f plset(const float& a)
      .    .    .       .    .    .  .    .    .  {
      .    .    .       .    .    .  .    .    .    const float f[] = {0, 1, 2, 3};
      .    .    .       .    .    .  .    .    .    Packet4f countdown = vld1q_f32(f);
      .    .    .       .    .    .  .    .    .    return vaddq_f32(pset1(a), countdown);
      .    .    .       .    .    .  .    .    .  }
      .    .    .       .    .    .  .    .    .  template<> EIGEN_STRONG_INLINE Packet4i plset(const int32_t& a)
-- line 154 ----------------------------------------

--------------------------------------------------------------------------------
The following files chosen for auto-annotation could not be found:
--------------------------------------------------------------------------------
  /build/glibc-BinVK7/glibc-2.23/elf/do-rel.h
  /build/glibc-BinVK7/glibc-2.23/wcsmbs/btowc.c
  /build/glibc-BinVK7/glibc-2.23/elf/dl-runtime.c
  /build/glibc-BinVK7/glibc-2.23/elf/dl-addr.c
  /build/glibc-BinVK7/glibc-2.23/elf/dl-lookup.c
  /build/glibc-BinVK7/glibc-2.23/string/../sysdeps/aarch64/strcmp.S
  /build/glibc-BinVK7/glibc-2.23/elf/dl-misc.c
  /build/glibc-BinVK7/glibc-2.23/elf/../sysdeps/aarch64/dl-machine.h

--------------------------------------------------------------------------------
Ir I1mr ILmr Dr D1mr DLmr Dw D1mw DLmw 
--------------------------------------------------------------------------------
55    2    2 59    0    0 58    0    0  percentage of events annotated
```

分析统计结果：

| 函数名称                    | 内存读操作数量(Dr列) | 一级数据缓存读不命中次数(D1mr列) | 最后一级数据缓存读不命中次数(DLmr列) | 内存写操作数量(Dw列) | 一级数据缓存写不命中次数(D1mw列) | 最后一级数据缓存写不命中次数(DLmw列) |
| :-------------------------- | :------------------- | :------------------------------- | :----------------------------------- | :------------------- | :------------------------------- | :----------------------------------- |
| matirx_multi_eigen          | 640,000              | 0                                | 0                                    | 240,000              | 0                                | 0                                    |
| matirx_multi_4x4_int32_neon | 80,000               | 3                                | 0                                    | 40,000               | 1                                | 1                                    |

可以看出，`matirx_multi_4x4_int32_neon`函数的内存读操作数量和内存写操作数量分别为`matirx_multi_eigen`函数的 0.125 倍、0.167 倍，其他的基本相同。因此，从 cache 的角度来看，`matirx_multi_4x4_int32_neon`函数比 Eigen 库的实现大大减少了内存读写操作数量，从而改善程序性能。