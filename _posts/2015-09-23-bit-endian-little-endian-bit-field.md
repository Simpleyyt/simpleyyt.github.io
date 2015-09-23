---
published: false
---

有如下位域结构体：

```cpp
struct {
    int a:4;
    int b:16;
    int c:12;
};
```

<!--more-->

**小端：**

在寄存器中分布如下：

```
C11 C10 C9 C8 C7 C6 C5 C4 C3 C2 C1 C0 B15 B14 B13 B12 B11 B10 B9 B8 B7 B6 B5 B4 B3 B2 B1 B0 A3 A2 A1 A0
```

在内存中存放格式为：

```
0xXXXX0020: B3 B2 B1 B0 A3 A2 A1 A0
0xXXXX0021: B11 B10 B9 B8 B7 B6 B5 B4
0x XXXX0022: C3 C2 C1 C0 B15 B14 B13 B12
0x XXXX0023: C11 C10 C9 C8 C7 C6 C5 C4
```

**大端：**

在寄存器中分布如下：

```
A3 A2 A1 A0 B15 B14 B13 B12 B11 B10 B9 B8 B7 B6 B5 B4 B3 B2 B1 B0 C11 C10 C9 C8 C7 C6 C5 C4 C3 C2 C1 C0
```

在内存中存放如下：

```
0xXXXX0020: A3 A2 A1 A0 B15 B14 B13
0xXXXX0021: B11 B10 B9 B8 B7 B6 B5 B4
0x XXXX0022: B3 B2 B1 B0 C11 C10 C9 C8
0x XXXX0023: C7 C6 C5 C4 C3 C2 C1 C0
C11 C10 C9 C8 C7 C6 C5 C4  B15 B14 B13 B12 C3 C2 C1 C0  B11 B10 B9 B8 B7 B6 B5 B4  A3 A2 A1 A0 B3 B2 B1 B0
```

---

本文转自：<http://blog.sina.com.cn/s/blog_6f611c300102uznw.html>