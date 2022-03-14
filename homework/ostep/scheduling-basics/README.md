# Scheduling basics

scheduler.py 这个程序允许你查看不同调度程序在调度指标（如响应时间、周转时间和总等待时间）下的执行情况。

## 问题 1

1、使用 SJF 和 FIFO 调度程序运行长度为 200 的 3 个作业时，计算响应时间和周转时间。

**答案：**

|调度程序|周转时间|响应时间|
|-|-|-|
|FIFO|400|200|
|SJF|400|200|

```shell
[root@3d4481b4da9f c-test]# ./scheduler.py -l 200,200,200 -p FIFO -c
...
```

```shell
[root@3d4481b4da9f c-test]# ./scheduler.py -l 200,200,200 -p SJF -c
...
```

## 问题 2

2、现在做同样的事情,但有不同长度的作业，即 100、200 和 300。

**答案：**

|调度程序|周转时间|响应时间|
|-|-|-|
|FIFO|333.33|133.33|
|SJF|333.33|133.33|

```shell
[root@3d4481b4da9f c-test]# ./scheduler.py -l 100,200,300 -p FIFO -c
...
```

```shell
[root@3d4481b4da9f c-test]# ./scheduler.py -l 100,200,300 -p SJF -c
...
```

## 问题 3

3、现在做同样的事情，但采用 RR 调度程序，时间片为 1。

**答案：**

200，200，200：

|调度程序|周转时间|响应时间|
|-|-|-|
|FIFO|599|1|

```shell
[root@3d4481b4da9f c-test]# ./scheduler.py -l 200,200,200 -p RR -q 1 -c
...

  [ time 597 ] Run job   0 for 1.00 secs ( DONE at 598.00 )
  [ time 598 ] Run job   1 for 1.00 secs ( DONE at 599.00 )
  [ time 599 ] Run job   2 for 1.00 secs ( DONE at 600.00 )

Final statistics:
  Job   0 -- Response: 0.00  Turnaround 598.00  Wait 398.00
  Job   1 -- Response: 1.00  Turnaround 599.00  Wait 399.00
  Job   2 -- Response: 2.00  Turnaround 600.00  Wait 400.00

  Average -- Response: 1.00  Turnaround 599.00  Wait 399.00
```

100，200，300：

|调度程序|周转时间|响应时间|
|-|-|-|
|FIFO|465.67|1|

```shell
[root@3d4481b4da9f c-test]# ./scheduler.py -l 200,200,200 -p RR -q 1 -c
...

  [ time 297 ] Run job   0 for 1.00 secs ( DONE at 298.00 )
...
  [ time 498 ] Run job   1 for 1.00 secs ( DONE at 499.00 )
...
  [ time 599 ] Run job   2 for 1.00 secs ( DONE at 600.00 )
  
Final statistics:
  Job   0 -- Response: 0.00  Turnaround 298.00  Wait 198.00
  Job   1 -- Response: 1.00  Turnaround 499.00  Wait 299.00
  Job   2 -- Response: 2.00  Turnaround 600.00  Wait 300.00

  Average -- Response: 1.00  Turnaround 465.67  Wait 265.67
```

## 问题 4

4、对于什么类型的工作负载，SJF 提供与 FIFO 相同的周转时间？

**答案：**

短任务先达到。

## 问题 5

5、对于什么类型的工作负载和量子长度（时间片长度），SJF 与 RR 提供相同的响应时间？

**答案：**

两种情况：

- 待运行的任务只有一个，也就是说，如果当前有任务未完成，不会有新任务达到。

- 所有任务（最长的任务除外）的运行时间至多为一个时间片。

## 问题 6

6、随着工作长度的增加，SJF 的响应时间会怎样？你能使用模拟程序来展示趋势吗？

**答案：**

响应时间会线性增加。

## 问题 7

7、随着量子长度（时间片长度）的增加，RR 的响应时间会怎样？你能写出一个方程？计算给定 N 个工作时,最坏情况的响应时间吗？

**答案：**

响应时间会线性增加。

假设时间片是 Q，每个任务最少需要一个时间片才能完成，那么平均响应时间是：Q(N-1)/2。
