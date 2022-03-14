# Segmentation

该程序允许你查看在具有分段的系统中如何执行地址转换。

## 问题 1

1、先让我们用一个小地址空间来转换一些地址。这里有一组简单的参数和几个不同的随机种子。你可以转换这些地址吗？

`segmentation.py -a 128 -p 512 -b 0 -l 20 -B 512 -L 20 -s 0`

`segmentation.py -a 128 -p 512 -b 0 -l 20 -B 512 -L 20 -s 1`

`segmentation.py -a 128 -p 512 -b 0 -l 20 -B 512 -L 20 -s 2`

**答案：**

```shell
[root@3d4481b4da9f c-test]# ./segmentation.py -a 128 -p 512 -b 0 -l 20 -B 512 -L 20 -s 0
ARG seed 0
ARG address space size 128
ARG phys mem size 512

Segment register information:

  Segment 0 base  (grows positive) : 0x00000000 (decimal 0)
  Segment 0 limit                  : 20

  Segment 1 base  (grows negative) : 0x00000200 (decimal 512)
  Segment 1 limit                  : 20

Virtual Address Trace
  VA  0: 0x0000006c (decimal:  108) --> PA or segmentation violation?
  VA  1: 0x00000061 (decimal:   97) --> PA or segmentation violation?
  VA  2: 0x00000035 (decimal:   53) --> PA or segmentation violation?
  VA  3: 0x00000021 (decimal:   33) --> PA or segmentation violation?
  VA  4: 0x00000041 (decimal:   65) --> PA or segmentation violation?
```

```shell
Virtual Address Trace
  VA  0: 0x00000011 (decimal:   17) --> PA or segmentation violation?
  VA  1: 0x0000006c (decimal:  108) --> PA or segmentation violation?
  VA  2: 0x00000061 (decimal:   97) --> PA or segmentation violation?
  VA  3: 0x00000020 (decimal:   32) --> PA or segmentation violation?
  VA  4: 0x0000003f (decimal:   63) --> PA or segmentation violation?
```

```shell
Virtual Address Trace
  VA  0: 0x0000007a (decimal:  122) --> PA or segmentation violation?
  VA  1: 0x00000079 (decimal:  121) --> PA or segmentation violation?
  VA  2: 0x00000007 (decimal:    7) --> PA or segmentation violation?
  VA  3: 0x0000000a (decimal:   10) --> PA or segmentation violation?
  VA  4: 0x0000006a (decimal:  106) --> PA or segmentation violation?
```

## 问题 2

2、现在，让我们看看是否理解了这个构建的小地址空间（使用上面问题的参数）。段 0 中最高的合法虚拟地址是什么？段 1 中最低的合法虚拟地址是什么？在整个地址空间中，最低和最高的非法地址是什么？最后，如何运行带有 A 标志的 segmentation.py 来测试你是否正确？

**答案：**

段 0 的界限寄存器是 20，因此段 0 最高的合法虚拟地址是 0x00000013（19）。

段 1 最高的合法虚拟地址是 0x0000007F（127），界限寄存器是 20，因此段 1 最低的合法虚拟地址是 0x0000006C（127 + 1 - 20 = 108）。

整个地址空间中，最低非法虚拟地址是 0x00000014（20），对应的物理地址是 0x00000014（20），最高非法虚拟地址 0x0000006B（107），对应的物理地址是 0x000001EB（491）。

```shell
[root@3d4481b4da9f c-test]# ./segmentation.py -a 128 -p 512 -b 0 -l 20 -B 512 -L 20 -A 19,108,20,107 -c
ARG seed 0
ARG address space size 128
ARG phys mem size 512

Segment register information:

  Segment 0 base  (grows positive) : 0x00000000 (decimal 0)
  Segment 0 limit                  : 20

  Segment 1 base  (grows negative) : 0x00000200 (decimal 512)
  Segment 1 limit                  : 20

Virtual Address Trace
  VA  0: 0x00000013 (decimal:   19) --> VALID in SEG0: 0x00000013 (decimal:   19)
  VA  1: 0x0000006c (decimal:  108) --> VALID in SEG1: 0x000001ec (decimal:  492)
  VA  2: 0x00000014 (decimal:   20) --> SEGMENTATION VIOLATION (SEG0)
  VA  3: 0x0000006b (decimal:  107) --> SEGMENTATION VIOLATION (SEG1)
```

## 问题 3

3、假设我们在一个 128 字节的物理内存中有一个很小的 16 字节地址空间。你会设置什么样的基址和界限，以便让模拟器为指定的地址流生成以下转换结果：有效，有效，违规……违规，有效，有效？假设用以下参数：

`./segmentation.py -a 16 -p 128 -A 0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15 --b0 ? --l0 ? --b1 ? --l1 ?`

**答案：**

题目要求 0，1，14，15 有效，其余无效。

因此答案是：

 `./segmentation.py -a 16 -p 128 -A 0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15 -b 0 -l 2 -B 16 -L 2`
 
 也就是说，只要 -l 和 -L 均为 2 即可。
 
 ## 问题 4
 
 4、假设我们想要生成一个问题，其中大约 90% 的随机生成的虚拟地址是有效的（即不产生段异常）。你应该如何配置模拟器来做到这一点？哪些参数很重要？
 
 **答案：**
 
 只要段 0 和段 1 的界限寄存器加一起达到整个地址空间的 90% 即可。
 
 `/segmentation.py -a 100 -p 128 -b 0 -l 45 -B 128 -L 45 -c`

## 问题 5

5、你可以运行模拟器，使所有虚拟地址无效吗？怎么做到？

**答案：**

`-l 0 -L 0`
