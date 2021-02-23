# 性能优化篇（2）：小心“STL 低效率用法”所带来的性能开销

Author:stormQ

Sunday, 17. November 2019 1 03:53PM

- 目录
  - [善用 reserve 预分配内存](https://blog.csdn.net/wohenfanjian/article/details/103244796#jump善用reserve预分配内存)
  - [善用 emplace_back 避免不必要的开销](https://blog.csdn.net/wohenfanjian/article/details/103244796#jump善用emplace_back避免不必要的开销)
  - [善用 std::move 避免不必要的拷贝开销](https://blog.csdn.net/wohenfanjian/article/details/103244796#jump善用std::move避免不必要的拷贝开销)

------

### 善用 reserve 预分配内存

- **reserve 函数的作用**

  将容器的容量（即可以容纳的最大元素数量）调整为指定的大小`n`，如果`n`小于容器的当前容量，那么直接忽略此操作。

- **适用场景**

  - 如果`std::vector`中要添加的元素数量已知，那么在添加元素前（通常是在循环中添加元素前）提前使用`reserve`函数预分配`std::vector`需要的内存。这样，可以避免由于临时申请内存而在`std::vector`内部发生数据迁移而带来的不必要的元素拷贝开销、创建新内存开销和释放旧内存开销，从而改善程序性能。

- **代码示例**

源码：

```
// main.cc


#include <vector>

#define N 1980*1024

class TestObject
{
public:
    TestObject(int a, int b, int c) : a_(a), b_(b), c_(c) {}

    int a_;
    int b_;
    int c_;
};

std::vector<TestObject> test_data_1;
std::vector<TestObject> test_data_2;

void poor(std::vector<TestObject> &data)
{
    for (int i = 0; i < N; i++)
    {
        data.push_back(TestObject(i, i+1, i+2));
    }
}

void better(std::vector<TestObject> &data)
{
    data.reserve(N);
    for (int i = 0; i < N; i++)
    {
        data.push_back(TestObject(i, i+1, i+2));
    }
}

int main()
{
    poor(test_data_1);
    better(test_data_2);
    return 0;
}
```

编译：

```
$ g++ -std=c++11 -g -Og -o main_Og main.cpp
```

函数耗时统计：

| 启动程序方式 | 第一次执行耗时(us)          | 第二次执行耗时(us)          | 第三次执行耗时(us)          | 第四次执行耗时(us)          | 第五次执行耗时(us)          |
| ------------ | --------------------------- | --------------------------- | --------------------------- | --------------------------- | --------------------------- |
| ./main_Og    | poor:43301<br/>better:16609 | poor:41402<br/>better:15822 | poor:60415<br/>better:15650 | poor:62450<br/>better:14970 | poor:41866<br/>better:16222 |

从统计结果中可以看出，示例中`better()`函数的执行速度比`poor()`函数至少快 `2`倍。

- **代码调试**

分析`poor()`函数添加前 9 个元素时，`std::vector`内部数据迁移情况。具体调试过程如下：

```
(gdb) l 25
20	
21	void poor(std::vector<TestObject> &data)
22	{
23	    for (int i = 0; i < N; i++)
24	    {
25	        data.push_back(TestObject(i, i+1, i+2));
26	    }
27	}
28	
29	void better(std::vector<TestObject> &data)
(gdb) b 25
Breakpoint 3 at 0x401065: file m.cpp, line 25.
(gdb) c
Continuing.

Breakpoint 3, poor (data=std::vector of length 0, capacity 0) at m.cpp:25
25	        data.push_back(TestObject(i, i+1, i+2));
(gdb) c
Continuing.

Breakpoint 3, poor (data=std::vector of length 1, capacity 1 = {...}) at m.cpp:25
25	        data.push_back(TestObject(i, i+1, i+2));
(gdb) display data.size()
1: data.size() = 1
(gdb) display &data[0]
2: &data[0] = (TestObject *) 0x614c20
(gdb) c
Continuing.

Breakpoint 3, poor (data=std::vector of length 2, capacity 2 = {...}) at m.cpp:25
25	        data.push_back(TestObject(i, i+1, i+2));
1: data.size() = 2
2: &data[0] = (TestObject *) 0x614c40
(gdb) 
Continuing.

Breakpoint 3, poor (data=std::vector of length 3, capacity 4 = {...}) at m.cpp:25
25	        data.push_back(TestObject(i, i+1, i+2));
1: data.size() = 3
2: &data[0] = (TestObject *) 0x614c60
(gdb) 
Continuing.

Breakpoint 3, poor (data=std::vector of length 4, capacity 4 = {...}) at m.cpp:25
25	        data.push_back(TestObject(i, i+1, i+2));
1: data.size() = 4
2: &data[0] = (TestObject *) 0x614c60
(gdb) 
Continuing.

Breakpoint 3, poor (data=std::vector of length 5, capacity 8 = {...}) at m.cpp:25
25	        data.push_back(TestObject(i, i+1, i+2));
1: data.size() = 5
2: &data[0] = (TestObject *) 0x614ca0
(gdb) 
Continuing.

Breakpoint 3, poor (data=std::vector of length 6, capacity 8 = {...}) at m.cpp:25
25	        data.push_back(TestObject(i, i+1, i+2));
1: data.size() = 6
2: &data[0] = (TestObject *) 0x614ca0
(gdb) 
Continuing.

Breakpoint 3, poor (data=std::vector of length 7, capacity 8 = {...}) at m.cpp:25
25	        data.push_back(TestObject(i, i+1, i+2));
1: data.size() = 7
2: &data[0] = (TestObject *) 0x614ca0
(gdb) 
Continuing.

Breakpoint 3, poor (data=std::vector of length 8, capacity 8 = {...}) at m.cpp:25
25	        data.push_back(TestObject(i, i+1, i+2));
1: data.size() = 8
2: &data[0] = (TestObject *) 0x614ca0
(gdb) 
Continuing.

Breakpoint 3, poor (data=std::vector of length 9, capacity 16 = {...}) at m.cpp:25
25	        data.push_back(TestObject(i, i+1, i+2));
1: data.size() = 9
2: &data[0] = (TestObject *) 0x614d10
```

可以看出，`poor()`函数在添加第 2、3、5、9 个元素时，第一个元素的地址——`&data[0]`发生了变化。即在这添加这些元素时，`std::vector`内部发生了数据迁移。在添加后面的某些元素时，也会导致`std::vector`内部数据迁移，这里不再赘述。

分析`better()`函数添加前 9 个元素时，`std::vector`内部是否有数据迁移情况。具体调试过程如下：

```
(gdb) l 34
29	void better(std::vector<TestObject> &data)
30	{
31	    data.reserve(N);
32	    for (int i = 0; i < N; i++)
33	    {
34	        data.push_back(TestObject(i, i+1, i+2));
35	    }
36	}
37	
38	int main()
(gdb) b 34
Breakpoint 4 at 0x400ffe: file m.cpp, line 34.
(gdb) c
Continuing.

Breakpoint 4, better (data=std::vector of length 0, capacity 2027520) at m.cpp:34
34	        data.push_back(TestObject(i, i+1, i+2));
(gdb) c
Continuing.

Breakpoint 4, better (data=std::vector of length 1, capacity 2027520 = {...}) at m.cpp:34
34	        data.push_back(TestObject(i, i+1, i+2));
(gdb) display data.size()
3: data.size() = 1
(gdb) display &data[0]
4: &data[0] = (TestObject *) 0x7ffff2ba6010
(gdb) c
Continuing.

Breakpoint 4, better (data=std::vector of length 2, capacity 2027520 = {...}) at m.cpp:34
34	        data.push_back(TestObject(i, i+1, i+2));
3: data.size() = 2
4: &data[0] = (TestObject *) 0x7ffff2ba6010
(gdb) 
Continuing.

Breakpoint 4, better (data=std::vector of length 3, capacity 2027520 = {...}) at m.cpp:34
34	        data.push_back(TestObject(i, i+1, i+2));
3: data.size() = 3
4: &data[0] = (TestObject *) 0x7ffff2ba6010
(gdb) 
Continuing.

Breakpoint 4, better (data=std::vector of length 4, capacity 2027520 = {...}) at m.cpp:34
34	        data.push_back(TestObject(i, i+1, i+2));
3: data.size() = 4
4: &data[0] = (TestObject *) 0x7ffff2ba6010
(gdb) 
Continuing.

Breakpoint 4, better (data=std::vector of length 5, capacity 2027520 = {...}) at m.cpp:34
34	        data.push_back(TestObject(i, i+1, i+2));
3: data.size() = 5
4: &data[0] = (TestObject *) 0x7ffff2ba6010
(gdb) 
Continuing.

Breakpoint 4, better (data=std::vector of length 6, capacity 2027520 = {...}) at m.cpp:34
34	        data.push_back(TestObject(i, i+1, i+2));
3: data.size() = 6
4: &data[0] = (TestObject *) 0x7ffff2ba6010
(gdb) 
Continuing.

Breakpoint 4, better (data=std::vector of length 7, capacity 2027520 = {...}) at m.cpp:34
34	        data.push_back(TestObject(i, i+1, i+2));
3: data.size() = 7
4: &data[0] = (TestObject *) 0x7ffff2ba6010
(gdb) 
Continuing.

Breakpoint 4, better (data=std::vector of length 8, capacity 2027520 = {...}) at m.cpp:34
34	        data.push_back(TestObject(i, i+1, i+2));
3: data.size() = 8
4: &data[0] = (TestObject *) 0x7ffff2ba6010
(gdb) 
Continuing.

Breakpoint 4, better (data=std::vector of length 9, capacity 2027520 = {...}) at m.cpp:34
34	        data.push_back(TestObject(i, i+1, i+2));
3: data.size() = 9
4: &data[0] = (TestObject *) 0x7ffff2ba6010
```

可以看出，`better()`函数在添加前 9 个元素时，第一个元素的地址——`&data[0]`都一直不变。即在这添加这些元素时，`std::vector`内部未发生数据迁移。在添加后面的其他元素时，也不会导致`std::vector`内部数据迁移，有兴趣的可以使用条件断点——`b 34 if &data[0]!=0x7ffff2ba6010`（0x7ffff2ba6010 为此处调试时 &data[0] 的值）验证这一点。这正是`使用 reserve 预分配内存`带来的效果。

上面示例中还有一处可以优化的地方，下文会详细说明。

------

### 善用 emplace_back 避免不必要的开销

- **emplace_back 函数的作用**

  相比于`push_back`函数，`emplace_back`函数可以直接构造元素，从而避免不必要的临时对象构造/析构开销。

- **适用场景**

  - 如果`std::vector`中要添加的元素需要传入参数构造时，用`emplace_back`而不是`push_back`函数来直接构造元素以避免不必要的开销，从而改善程序性能。

- **代码示例**

源码：

```
// main.cc


#include <vector>

#define N 1980*1024

class TestObject
{
public:
    TestObject(int a, int b, int c) : a_(a), b_(b), c_(c) {}

    int a_;
    int b_;
    int c_;
};

std::vector<TestObject> test_data_1;
std::vector<TestObject> test_data_2;

void poor(std::vector<TestObject> &data)
{
    data.reserve(N);
    for (int i = 0; i < N; i++)
    {
        data.push_back(TestObject(i, i+1, i+2));
    }
}

void better(std::vector<TestObject> &data)
{
    data.reserve(N);
    for (int i = 0; i < N; i++)
    {
        data.emplace_back(i, i+1, i+2);
    }
}

int main()
{
    poor(test_data_1);
    better(test_data_2);
    return 0;
}
```

编译：

```
$ g++ -std=c++11 -g -Og -o main_Og main.cpp
```

函数耗时统计：

| 启动程序方式 | 第一次执行耗时(us)          | 第二次执行耗时(us)          | 第三次执行耗时(us)          | 第四次执行耗时(us)          | 第五次执行耗时(us)          |
| ------------ | --------------------------- | --------------------------- | --------------------------- | --------------------------- | --------------------------- |
| ./main_Og    | poor:27588<br/>better:16733 | poor:26789<br/>better:17094 | poor:30049<br/>better:16380 | poor:28057<br/>better:16301 | poor:26616<br/>better:17038 |

从统计结果中可以看出，示例中`better()`函数的执行速度比`poor()`函数至少快 `1.5`倍。

- **代码调试**

为了验证`emplace_back`函数的作用，引入三个全局变量`g_count_1`、`g_count_2`和`g_count_3`用于统计构造函数、拷贝构造函数、析构函数被调用的次数。修改后的源码为：

```
// main.cc


#include <vector>

#define N 1980*1024

int g_count_1 = 0;
int g_count_2 = 0;
int g_count_3 = 0;

class TestObject
{
public:
    TestObject(int a, int b, int c) : a_(a), b_(b), c_(c)
    {
        g_count_1++;
    }

    TestObject(const TestObject &other)
    {
        if (this != &other)
        {
            this->a_ = other.a_;
            this->b_ = other.b_;
            this->c_ = other.c_;
        }
        g_count_2++;
    }

    ~TestObject()
    {
        g_count_3++;
    }

    int a_;
    int b_;
    int c_;
};

std::vector<TestObject> test_data_1;
std::vector<TestObject> test_data_2;

void poor(std::vector<TestObject> &data)
{
    data.reserve(N);
    for (int i = 0; i < N; i++)
    {
        data.push_back(TestObject(i, i+1, i+2));
    }
}

void better(std::vector<TestObject> &data)
{
    data.reserve(N);
    for (int i = 0; i < N; i++)
    {
        data.emplace_back(i, i+1, i+2);
    }
}

int main()
{
    g_count_1 = 0;
    g_count_2 = 0;
    g_count_3 = 0;
    poor(test_data_1);

    g_count_1 = 0;
    g_count_2 = 0;
    g_count_3 = 0;
    better(test_data_2);
    g_count_1 = 0;

    return 0;
}
```

具体调试过程如下：

```
(gdb) l
61	
62	int main()
63	{
64	    g_count_1 = 0;
65	    g_count_2 = 0;
66	    g_count_3 = 0;
67	    poor(test_data_1);
68	
69	    g_count_1 = 0;
70	    g_count_2 = 0;
71	    g_count_3 = 0;
72	    better(test_data_2);
73	    g_count_1 = 0;
74	
75	    return 0;
76	}
(gdb) b 69
Breakpoint 2 at 0x400b3e: file main.cpp, line 69.
(gdb) b 73
Breakpoint 3 at 0x400b66: file main.cpp, line 73.
(gdb) display g_count_1
1: g_count_1 = 0
(gdb) display g_count_2
2: g_count_2 = 0
(gdb) display g_count_3
3: g_count_3 = 0
(gdb) c
Continuing.

Breakpoint 2, main () at main.cpp:69
69	    g_count_1 = 0;
1: g_count_1 = 2027520
2: g_count_2 = 2027520
3: g_count_3 = 2027520
(gdb) n
70	    g_count_2 = 0;
1: g_count_1 = 0
2: g_count_2 = 2027520
3: g_count_3 = 2027520
(gdb) 
71	    g_count_3 = 0;
1: g_count_1 = 0
2: g_count_2 = 0
3: g_count_3 = 2027520
(gdb) 
72	    better(test_data_2);
1: g_count_1 = 0
2: g_count_2 = 0
3: g_count_3 = 0
(gdb) n

Breakpoint 3, main () at main.cpp:73
73	    g_count_1 = 0;
1: g_count_1 = 2027520
2: g_count_2 = 0
3: g_count_3 = 0
(gdb) n
76	}
1: g_count_1 = 0
2: g_count_2 = 0
3: g_count_3 = 0
```

可以看出，`poor`函数在执行过程中调用容器元素构造函数、析构函数和拷贝构造函数的次数都是 2027520（2027520 = 1980 x 1024），验证了`push_back`函数的三种开销：临时对象的构造/析构开销和拷贝开销。`better`函数在执行过程中调用容器元素构造函数、析构函数和拷贝构造函数的次数分别是 2027520、0、0，验证了`emplace_back`函数只有必要的构造元素开销，不会引入临时对象，避免了不必要的临时对象构造/析构开销，从而改善程序性能。

------

### 善用 std::move 避免不必要的拷贝开销

- **std::move 函数的作用**

  移动而非拷贝数据以避免不必要的拷贝开销，从而改善程序性能。

- **适用场景**

  - 构造函数的参数列表中有类型为`std::vector`的参数，并且该参数只在构造过程中使用。

- **代码示例**

源码：

```
// main.cpp


#include <list>
#include <vector>

#define N 1980*1024

class TestObject
{
public:
    TestObject(int a, int b, int c) : a_(a), b_(b), c_(c) {}

    int a_;
    int b_;
    int c_;
};

class Element
{
public:
    explicit Element(const std::vector<TestObject> &data)
        : data_(data)
    {
    }

    explicit Element(std::vector<TestObject> &&data)
        : data_(std::move(data))
    {
    }

private:
    std::vector<TestObject> data_;
};

std::list<Element> test_data_1;
std::list<Element> test_data_2;
Element *g_ele = nullptr;

void poor(std::list<Element> &data)
{
    std::vector<TestObject> vec;
    vec.reserve(N);
    for (int i = 0; i < N; i++)
    {
        vec.push_back(TestObject(i, i+1, i+2));
    }

    data.emplace_back(vec);
    g_ele = &data.front();
}

void better(std::list<Element> &data)
{
    std::vector<TestObject> vec;
    vec.reserve(N);
    for (int i = 0; i < N; i++)
    {
        vec.emplace_back(i, i+1, i+2);
    }

    data.emplace_back(std::move(vec));
    g_ele = &data.front();
}

int main()
{
    poor(test_data_1);
    better(test_data_2);
    return 0;
}
```

编译：

```
$ g++ -std=c++11 -g -Og -o main_Og main.cpp
```

函数耗时统计：

| 启动程序方式 | 第一次执行耗时(us)          | 第二次执行耗时(us)          | 第三次执行耗时(us)          | 第四次执行耗时(us)          | 第五次执行耗时(us)          |
| ------------ | --------------------------- | --------------------------- | --------------------------- | --------------------------- | --------------------------- |
| ./main_Og    | poor:37272<br/>better:13913 | poor:39693<br/>better:14832 | poor:37982<br/>better:14439 | poor:38427<br/>better:14590 | poor:38391<br/>better:14502 |

从统计结果中可以看出，示例中`better()`函数的执行速度比`poor()`函数至少快 `2`倍。

- **代码调试**

验证`data.emplace_back(vec); 会引发拷贝开销`，具体调试过程如下：

```
(gdb) l 50
35	
36	std::list<Element> test_data_1;
37	std::list<Element> test_data_2;
38	Element *g_ele = nullptr;
39	
40	void poor(std::list<Element> &data)
41	{
42	    std::vector<TestObject> vec;
43	    vec.reserve(N);
44	    for (int i = 0; i < N; i++)
45	    {
46	        vec.push_back(TestObject(i, i+1, i+2));
47	    }
48	
49	    data.emplace_back(vec);
50	    g_ele = &data.front();
51	}
52	
53	void better(std::list<Element> &data)
54	{
55	    std::vector<TestObject> vec;
56	    vec.reserve(N);
57	    for (int i = 0; i < N; i++)
58	    {
59	        vec.emplace_back(i, i+1, i+2);
60	    }
61	
62	    data.emplace_back(std::move(vec));
63	    g_ele = &data.front();
64	}
(gdb) b 50
Breakpoint 3 at 0x40131c: file p.cpp, line 50.
(gdb) c
Continuing.

Breakpoint 3, poor (data=std::__cxx11::list = {...}) at p.cpp:50
50	    g_ele = &data.front();
(gdb) display vec._M_impl
1: vec._M_impl = {<std::allocator<TestObject>> = {<__gnu_cxx::new_allocator<TestObject>> = {<No data fields>}, <No data fields>}, _M_start = 0x7ffff5a2b010, 
  _M_finish = 0x7ffff715f010, _M_end_of_storage = 0x7ffff715f010}
(gdb) n
42	    std::vector<TestObject> vec;
1: vec._M_impl = {<std::allocator<TestObject>> = {<__gnu_cxx::new_allocator<TestObject>> = {<No data fields>}, <No data fields>}, _M_start = 0x7ffff5a2b010, 
  _M_finish = 0x7ffff715f010, _M_end_of_storage = 0x7ffff715f010}
(gdb) display g_ele->data_._M_impl
2: g_ele->data_._M_impl = {<std::allocator<TestObject>> = {<__gnu_cxx::new_allocator<TestObject>> = {<No data fields>}, <No data fields>}, _M_start = 0x7ffff42f6010, 
  _M_finish = 0x7ffff5a2a010, _M_end_of_storage = 0x7ffff5a2a010}
```

可以看出，`poor`函数中的临时对象`vec`的内存地址范围为`0x7ffff5a2b010~0x7ffff715f010`（vec._M_impl._M_start 的值为 0x7ffff5a2b010，vec._M_impl.*M_end_of_storage 的值为 0x7ffff715f010）。全局变量`test_data_1`首元素的数据成员`data_`的内存地址范围为`0x7ffff42f6010~0x7ffff5a2a010`（g_ele->data*._M_impl.*M_start 的值为 0x7ffff42f6010，g_ele->data*._M_impl._M_end_of_storage 的值为 0x7ffff5a2a010）。两者的内存地址范围不相同。因此，全局变量`test_data_1`首元素的数据成员`data_`是从临时对象`vec`拷贝而来的，从而验证了`data.emplace_back(vec); 会引发拷贝开销。

验证`data.emplace_back(std::move(vec)); 不会引发拷贝开销`，具体调试过程如下：

```
(gdb) b 59
Breakpoint 4 at 0x4010ba: file p.cpp, line 59.
(gdb) b 63
Breakpoint 5 at 0x401147: file p.cpp, line 63.
(gdb) c
Continuing.
poor_function elapsed_time: 59091892 us

Breakpoint 4, better (data=empty std::__cxx11::list) at p.cpp:59
59	        vec.emplace_back(i, i+1, i+2);
2: g_ele->data_._M_impl = {<std::allocator<TestObject>> = {<__gnu_cxx::new_allocator<TestObject>> = {<No data fields>}, <No data fields>}, _M_start = 0x7ffff42f6010, 
  _M_finish = 0x7ffff5a2a010, _M_end_of_storage = 0x7ffff5a2a010}
(gdb) display vec._M_impl
3: vec._M_impl = {<std::allocator<TestObject>> = {<__gnu_cxx::new_allocator<TestObject>> = {<No data fields>}, <No data fields>}, _M_start = 0x616060, _M_finish = 0x616060, 
  _M_end_of_storage = 0x1d4a060}
(gdb) d 4
(gdb) c
Continuing.

Breakpoint 5, better (data=std::__cxx11::list = {...}) at p.cpp:63
63	    g_ele = &data.front();
2: g_ele->data_._M_impl = {<std::allocator<TestObject>> = {<__gnu_cxx::new_allocator<TestObject>> = {<No data fields>}, <No data fields>}, _M_start = 0x7ffff42f6010, 
  _M_finish = 0x7ffff5a2a010, _M_end_of_storage = 0x7ffff5a2a010}
3: vec._M_impl = {<std::allocator<TestObject>> = {<__gnu_cxx::new_allocator<TestObject>> = {<No data fields>}, <No data fields>}, _M_start = 0x0, _M_finish = 0x0, 
  _M_end_of_storage = 0x0}
(gdb) n
55	    std::vector<TestObject> vec;
2: g_ele->data_._M_impl = {<std::allocator<TestObject>> = {<__gnu_cxx::new_allocator<TestObject>> = {<No data fields>}, <No data fields>}, _M_start = 0x616060, _M_finish = 0x1d4a060, 
  _M_end_of_storage = 0x1d4a060}
3: vec._M_impl = {<std::allocator<TestObject>> = {<__gnu_cxx::new_allocator<TestObject>> = {<No data fields>}, <No data fields>}, _M_start = 0x0, _M_finish = 0x0, 
  _M_end_of_storage = 0x0}
```

可以看出，`better`函数中的临时对象`vec`的内存地址范围为`0x616060~0x1d4a060`（vec._M_impl._M_start 的值为 0x616060，vec._M_impl.*M_end_of_storage 的值为 0x1d4a060）。全局变量`test_data_2`首元素的数据成员`data_`的内存地址范围为`0x616060~0x1d4a060`（g_ele->data*._M_impl.*M_start 的值为 0x616060，g_ele->data*._M_impl._M_end_of_storage 的值为 0x1d4a060）。两者的内存地址范围相同。因此，全局变量`test_data_2`首元素的数据成员`data_`是直接将临时对象`vec`移动而来的，从而验证了`data.emplace_back(std::move(vec)); 不会引发拷贝开销`。另外，需要注意的是，临时对象`vec`在被移动之后将不再有效。