# Paging

在这个作业中，你将使用一个简单的程序（名为 paging-linear-translate.py），来看看你是否理解了简单的虚拟--物理地址转换如何与线性页表一起工作。

## 问题 1

1、在做地址转换之前，让我们用模拟器来研究线性页表在给定不同参数的情况下如何改变大小。在不同参数变化时，计算线性页表的大小。一些建议输入如下，通过使用 -v 标志，你可以看到填充了多少个页表项。

首先，要理解线性页表大小如何随着地址空间的增长而变化：

```shell
paging-linear-translate.py -P 1k -a 1m -p 512m -v -n 0
paging-linear-translate.py -P 1k -a 2m -p 512m -v -n 0
paging-linear-translate.py -P 1k -a 4m -p 512m -v -n 0
```

然后，理解线性页面大小如何随页大小的增长而变化：

```shell
paging-linear-translate.py -P 1k -a 1m -p 512m -v -n 0
paging-linear-translate.py -P 2k -a 1m -p 512m -v -n 0
paging-linear-translate.py -P 4k -a 1m -p 512m -v -n 0
```

在运行这些命令之前，请试着想想预期的趋势。页表大小如何随地址空间的增长而改变？随着页大小的增长呢？为什么一般来说，我们不应该使用很大的页呢？

**答案：**

页大小 1k，地址空间 1m，所以页表大小为 1024。同理，当页大小不变时，页表大小随着地址空间增长而变大；当地址空间不变时，页表大小随着页大小增加而减少。

页过大，会造成浪费。

## 问题 2

2、现在让我们做一些地址转换。从一些小例子开始，使 -u 标志更改分配给地址空间的页数。例如：

```shell
paging-linear-translate.py -P 1k -a 16k -p 32k -v -u 0
paging-linear-translate.py -P 1k -a 16k -p 32k -v -u 25
paging-linear-translate.py -P 1k -a 16k -p 32k -v -u 50
paging-linear-translate.py -P 1k -a 16k -p 32k -v -u 75
paging-linear-translate.py -p 1k -a 16k -p 32k -v -u 100
```

如果增加每个地址空间中的页的百分比，会发生什么？

**答案：**

以 VA 0x00002bc6 (decimal:    11206) 为例，2bc6 用二进制表示为 00101011 11000110。

已知页大小为 1k，所以偏移用 10 位表示即可，又已知地址空间为 16k，所以 VPN 用 4 位表示即可。因此偏移为二进制 1111000110，VPN 为二进制 1010（十进制 10），对应 PFN 为十六进制 0x13，与偏移合在一起即为二进制 00010011 1111000110（十六进制 0x4fc6，十进制 20422）。

```shell
[root@3d4481b4da9f c-test]# ./paging-linear-translate.py -P 1k -a 16k -p 32k -v -u 25 -c
ARG seed 0
ARG address space size 16k
ARG phys mem size 32k
ARG page size 1k
ARG verbose True
ARG addresses -1


The format of the page table is simple:
The high-order (left-most) bit is the VALID bit.
  If the bit is 1, the rest of the entry is the PFN.
  If the bit is 0, the page is not valid.
Use verbose mode (-v) if you want to print the VPN # by
each entry of the page table.

Page Table (from entry 0 down to the max size)
  [       0]  0x80000018
  [       1]  0x00000000
  [       2]  0x00000000
  [       3]  0x00000000
  [       4]  0x00000000
  [       5]  0x80000009
  [       6]  0x00000000
  [       7]  0x00000000
  [       8]  0x80000010
  [       9]  0x00000000
  [      10]  0x80000013
  [      11]  0x00000000
  [      12]  0x8000001f
  [      13]  0x8000001c
  [      14]  0x00000000
  [      15]  0x00000000

Virtual Address Trace
  VA 0x00003986 (decimal:    14726) -->  Invalid (VPN 14 not valid)
  VA 0x00002bc6 (decimal:    11206) --> 00004fc6 (decimal    20422) [VPN 10]
  VA 0x00001e37 (decimal:     7735) -->  Invalid (VPN 7 not valid)
  VA 0x00000671 (decimal:     1649) -->  Invalid (VPN 1 not valid)
  VA 0x00001bc9 (decimal:     7113) -->  Invalid (VPN 6 not valid)
```

## 问题 3

3、现在让我们尝试一些不同的随机种子，以及一些不同的（有时相当疯狂的）地址空间参数：

```shell
paging-linear-translate.py -P 8  -a  32    -p 1024 -v -s 1
paging-linear-translate.py -P 8k -a  32k   -p 1m   -v -s 2
paging-linear-translate.py -P 1m -a  256m  -p 512m -v -s 3
```

哪些参数组合是不现实的？为什么？

**答案：**

第一个页大小太小，第三个太大。

## 问题 4

4、利用该程序尝试其他一些问题。你能找到让程序无法工作的限制吗？例如，如果地址空间大小大于物理内存，会发生什么情况？

**答案：**

略
