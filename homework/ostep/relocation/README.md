# Relocation

程序 relocation.py 让你看到，在带有基址和边界寄存器的系统中，如何执行地址转换。

## 问题 1

1、用种子 1、2 和 3 运行，并计算进程生成的每个虚拟地址是处于界限内还是界限外？如果在界限内，请计算地址转换。

**答案：**

```shell
[root@3d4481b4da9f c-test]# ./relocation.py -s 1

ARG seed 1
ARG address space size 1k
ARG phys mem size 16k

Base-and-Bounds register information:

  Base   : 0x0000363c (decimal 13884)
  Limit  : 290

Virtual Address Trace
  VA  0: 0x0000030e (decimal:  782) --> segmentation violation
  VA  1: 0x00000105 (decimal:  261) --> 0x00003741
  VA  2: 0x000001fb (decimal:  507) --> segmentation violation
  VA  3: 0x000001cc (decimal:  460) --> segmentation violation
  VA  4: 0x0000029b (decimal:  667) --> segmentation violation
```

## 问题 2

2、使用以下标志运行：-s 0 -n 10。为了确保所有生成的虚拟地址都处于边界内，要将 -l（界限寄存器）设置为什么值？

**答案：**

界限寄存器标识的是长度，因此要设置为 930。

```shell
[root@3d4481b4da9f c-test]# ./relocation.py -s 0 -n 10 -l 930 -c

ARG seed 0
ARG address space size 1k
ARG phys mem size 16k

Base-and-Bounds register information:

  Base   : 0x0000360b (decimal 13835)
  Limit  : 930

Virtual Address Trace
  VA  0: 0x00000308 (decimal:  776) --> VALID: 0x00003913 (decimal: 14611)
  VA  1: 0x000001ae (decimal:  430) --> VALID: 0x000037b9 (decimal: 14265)
  VA  2: 0x00000109 (decimal:  265) --> VALID: 0x00003714 (decimal: 14100)
  VA  3: 0x0000020b (decimal:  523) --> VALID: 0x00003816 (decimal: 14358)
  VA  4: 0x0000019e (decimal:  414) --> VALID: 0x000037a9 (decimal: 14249)
  VA  5: 0x00000322 (decimal:  802) --> VALID: 0x0000392d (decimal: 14637)
  VA  6: 0x00000136 (decimal:  310) --> VALID: 0x00003741 (decimal: 14145)
  VA  7: 0x000001e8 (decimal:  488) --> VALID: 0x000037f3 (decimal: 14323)
  VA  8: 0x00000255 (decimal:  597) --> VALID: 0x00003860 (decimal: 14432)
  VA  9: 0x000003a1 (decimal:  929) --> VALID: 0x000039ac (decimal: 14764)
```

## 问题 3

3、使用以下标志运行：-s 1 -n 10 -l 100。可以设置基址的最大值是多少，以便地址空间仍然完全放在物理内存中？

**答案：**

这题有点问题，应该理解成基址设置成多少，能仅容纳 100 的长度。

物理内存大小为 16KB，要使地址完全放在物理内存中，基址最大值为：物理内存大小 - 界限寄存器大小 = 16 * 1024 - 100 = 16284。

## 问题 4

4、运行和第 3 题相同的操作，但使用较大的地址空间（-a）和物理内存（-p）。

**答案：**

题目看不懂。

## 问题 5

5、作为界限寄存器的值的函数，随机生成的虚拟地址的哪一部分是有效的？画一个图，使用不同随机种子运行，限制值从 0 到最大地址空间大小。

**答案：**

不会。
