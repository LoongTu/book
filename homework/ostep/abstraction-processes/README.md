# Abstraction processes

程序 process-run.py 让你查看程序运行时进程状态如何改变，是在使用 CPU（例如，执行相加指令）还是执行 I/O（例如，向磁盘发送请求并等待它完成）。

## 问题 1

1、用以下标志运行程序：./process-run.py -l 5:100,5:100。CPU 利用率（CPU 使用时间的百分比）应该是多少？为什么你知道这一点？利用 -c 标记查看你的答案是否正确。

**答案：**

CPU 利用率 100%。因为指令只使用了 CPU，没有进行 IO 等阻塞操作。

```shell
[root@3d4481b4da9f c-test]# ./process-run.py -l 5:100,5:100 -c -p
Time        PID: 0        PID: 1           CPU           IOs
  1        RUN:cpu         READY             1          
  2        RUN:cpu         READY             1          
  3        RUN:cpu         READY             1          
  4        RUN:cpu         READY             1          
  5        RUN:cpu         READY             1          
  6           DONE       RUN:cpu             1          
  7           DONE       RUN:cpu             1          
  8           DONE       RUN:cpu             1          
  9           DONE       RUN:cpu             1          
 10           DONE       RUN:cpu             1          

Stats: Total Time 10
Stats: CPU Busy 10 (100.00%)
Stats: IO Busy  0 (0.00%)
```

## 问题 2

2、现在用这些标志运行：./process-run.py -l 4:100,1:0。这些标志指定了一个包含 4 条指令的进程（都要使用 CPU），并且只是简单地发出 IO 并等待它完成。完成这两个进程需要多长时间？利用 -c 检查你的答案是否正确。**

**答案：**
完成这两个进程需要 11 个时钟周期。

```shell
[root@3d4481b4da9f c-test]# ./process-run.py -l 4:100,1:0 -c -p
Time        PID: 0        PID: 1           CPU           IOs
  1        RUN:cpu         READY             1          
  2        RUN:cpu         READY             1          
  3        RUN:cpu         READY             1          
  4        RUN:cpu         READY             1          
  5           DONE        RUN:io             1          
  6           DONE       WAITING                           1
  7           DONE       WAITING                           1
  8           DONE       WAITING                           1
  9           DONE       WAITING                           1
 10           DONE       WAITING                           1
 11*          DONE   RUN:io_done             1          

Stats: Total Time 11
Stats: CPU Busy 6 (54.55%)
Stats: IO Busy  5 (45.45%)
```

## 问题 3

3、现在交换进程的顺序：./process-run.py -l 1:0,4:100。现在发生了什么？交换顺序是否重要？为什么？同样，用 -c 看看你的答案是否正确。**

**答案：**

完成这两个进程用时 7 个时钟周期。重要，在进程 0 等待 IO 的时间里，CPU 可以执行进程 1。

```shell
[root@3d4481b4da9f c-test]# ./process-run.py -l 1:0,4:100 -c -p
Time        PID: 0        PID: 1           CPU           IOs
  1         RUN:io         READY             1          
  2        WAITING       RUN:cpu             1             1
  3        WAITING       RUN:cpu             1             1
  4        WAITING       RUN:cpu             1             1
  5        WAITING       RUN:cpu             1             1
  6        WAITING          DONE                           1
  7*   RUN:io_done          DONE             1          

Stats: Total Time 7
Stats: CPU Busy 6 (85.71%)
Stats: IO Busy  5 (71.43%)
```

## 问题 4

4、现在探索另一些标志。一个重要的标志是 -S，它决定了当进程发出 IO 时系统如何反应。将标志设置为 SWITCH_ON_END，在进程进行 I/O 操作时，系统将不会切换到另一个进程,而是等待进程完成。当你运行以下两个进程时，会发生什么情况？一个执行 I/O，另一个执行 CPU 工作。（-l 1:0,4:100 -c -S SWITCH_ON_END）**

**答案：**

进程 0 执行 IO 操作时，进程 1 等待。

```shell
[root@3d4481b4da9f c-test]# ./process-run.py -l 1:0,4:100 -S SWITCH_ON_END -c -p
Time        PID: 0        PID: 1           CPU           IOs
  1         RUN:io         READY             1          
  2        WAITING         READY                           1
  3        WAITING         READY                           1
  4        WAITING         READY                           1
  5        WAITING         READY                           1
  6        WAITING         READY                           1
  7*   RUN:io_done         READY             1          
  8           DONE       RUN:cpu             1          
  9           DONE       RUN:cpu             1          
 10           DONE       RUN:cpu             1          
 11           DONE       RUN:cpu             1          

Stats: Total Time 11
Stats: CPU Busy 6 (54.55%)
Stats: IO Busy  5 (45.45%)
```

## 问题 5

5、现在，运行相同的进程，但切换行为设置，在等待 IO 时切换到另一个进程（-l 1:0,4:100 -c -S SWITCH_ON_IO）。现在会发生什么？利用 -c 来确认你的答案是否正确。**

**答案：**

进程 0 执行 IO 操作时，CPU 会执行进程 1。

```shell
[root@3d4481b4da9f c-test]# ./process-run.py -l 1:0,4:100 -S SWITCH_ON_IO -c -p
Time        PID: 0        PID: 1           CPU           IOs
  1         RUN:io         READY             1          
  2        WAITING       RUN:cpu             1             1
  3        WAITING       RUN:cpu             1             1
  4        WAITING       RUN:cpu             1             1
  5        WAITING       RUN:cpu             1             1
  6        WAITING          DONE                           1
  7*   RUN:io_done          DONE             1          

Stats: Total Time 7
Stats: CPU Busy 6 (85.71%)
Stats: IO Busy  5 (71.43%)
```

## 问题 6

6、另一个重要的行为是 IO 完成时要做什么。利用 -I IO_RUN_LATER，当 IO 完成时，发出它的进程不一定马上运行。相反，当时运行的进程一直运行。当你运行这个进程组合时会发生什么？（./process-run.py -l 3:0,5:100,5:100 -S SWITCH_ON_IO -I IO_RUN_LATER -c -p）系统资源是否被有效利用？**

**答案：**

IO 完成时，CPU 并没有马上运行进程 0，而是执行了进程 2。系统资源没有被有效利用。

```shell
[root@3d4481b4da9f c-test]# ./process-run.py -l 3:0,5:100,5:100 -S SWITCH_ON_IO -I IO_RUN_LATER -c -p
Time        PID: 0        PID: 1        PID: 2           CPU           IOs
  1         RUN:io         READY         READY             1          
  2        WAITING       RUN:cpu         READY             1             1
  3        WAITING       RUN:cpu         READY             1             1
  4        WAITING       RUN:cpu         READY             1             1
  5        WAITING       RUN:cpu         READY             1             1
  6        WAITING       RUN:cpu         READY             1             1
  7*         READY          DONE       RUN:cpu             1          
  8          READY          DONE       RUN:cpu             1          
  9          READY          DONE       RUN:cpu             1          
 10          READY          DONE       RUN:cpu             1          
 11          READY          DONE       RUN:cpu             1          
 12    RUN:io_done          DONE          DONE             1          
 13         RUN:io          DONE          DONE             1          
 14        WAITING          DONE          DONE                           1
 15        WAITING          DONE          DONE                           1
 16        WAITING          DONE          DONE                           1
 17        WAITING          DONE          DONE                           1
 18        WAITING          DONE          DONE                           1
 19*   RUN:io_done          DONE          DONE             1          
 20         RUN:io          DONE          DONE             1          
 21        WAITING          DONE          DONE                           1
 22        WAITING          DONE          DONE                           1
 23        WAITING          DONE          DONE                           1
 24        WAITING          DONE          DONE                           1
 25        WAITING          DONE          DONE                           1
 26*   RUN:io_done          DONE          DONE             1          

Stats: Total Time 26
Stats: CPU Busy 16 (61.54%)
Stats: IO Busy  15 (57.69%)
```

## 问题 7

7、现在运行相同的进程，但使用 -I IO_RUN_IMMEDIATE 设置，该设置立即运行发出 IO 的进程。这种行为有何不同？为什么运行一个刚刚完成 IO 的进程会是一个好主意？**

**答案：**

刚完成 IO 的进程，很大可能会再次发出 IO，因此立即运行这样的进程，有利于提高 CPU 的利用率。

```shell
[root@3d4481b4da9f c-test]# ./process-run.py -l 3:0,5:100,5:100 -S SWITCH_ON_IO -I IO_RUN_IMMEDIATE -c -p
Time        PID: 0        PID: 1        PID: 2           CPU           IOs
  1         RUN:io         READY         READY             1          
  2        WAITING       RUN:cpu         READY             1             1
  3        WAITING       RUN:cpu         READY             1             1
  4        WAITING       RUN:cpu         READY             1             1
  5        WAITING       RUN:cpu         READY             1             1
  6        WAITING       RUN:cpu         READY             1             1
  7*   RUN:io_done          DONE         READY             1          
  8         RUN:io          DONE         READY             1          
  9        WAITING          DONE       RUN:cpu             1             1
 10        WAITING          DONE       RUN:cpu             1             1
 11        WAITING          DONE       RUN:cpu             1             1
 12        WAITING          DONE       RUN:cpu             1             1
 13        WAITING          DONE       RUN:cpu             1             1
 14*   RUN:io_done          DONE          DONE             1          
 15         RUN:io          DONE          DONE             1          
 16        WAITING          DONE          DONE                           1
 17        WAITING          DONE          DONE                           1
 18        WAITING          DONE          DONE                           1
 19        WAITING          DONE          DONE                           1
 20        WAITING          DONE          DONE                           1
 21*   RUN:io_done          DONE          DONE             1          

Stats: Total Time 21
Stats: CPU Busy 16 (76.19%)
Stats: IO Busy  15 (71.43%)
```

## 问题 8

8、现在运行一些随机生成的进程，例如 -s 1 -l 3:50,3:50、-s 2 -l 3:50,3:50、-s 3 -l 3:50,3:50。看看你是否能预测追踪记录会如何变化？当你使用 -I IO_RUN_IMMEDIATE 与 -I IO_RUN LATER 时会发生什么？当你使用 -S SWITCH_ON_IO 与 -S SWITCH_ON_END 时会发生什么？**

**答案：**

```shell
[root@3d4481b4da9f c-test]# ./process-run.py -s 1 -l 3:50,3:50 -c -p -I IO_RUN_IMMEDIATE
Time        PID: 0        PID: 1           CPU           IOs
  1        RUN:cpu         READY             1          
  2         RUN:io         READY             1          
  3        WAITING       RUN:cpu             1             1
  4        WAITING       RUN:cpu             1             1
  5        WAITING       RUN:cpu             1             1
  6        WAITING          DONE                           1
  7        WAITING          DONE                           1
  8*   RUN:io_done          DONE             1          
  9         RUN:io          DONE             1          
 10        WAITING          DONE                           1
 11        WAITING          DONE                           1
 12        WAITING          DONE                           1
 13        WAITING          DONE                           1
 14        WAITING          DONE                           1
 15*   RUN:io_done          DONE             1          

Stats: Total Time 15
Stats: CPU Busy 8 (53.33%)
Stats: IO Busy  10 (66.67%)


[root@3d4481b4da9f c-test]# ./process-run.py -s 1 -l 3:50,3:50 -c -p -I IO_RUN_LATER
Time        PID: 0        PID: 1           CPU           IOs
  1        RUN:cpu         READY             1          
  2         RUN:io         READY             1          
  3        WAITING       RUN:cpu             1             1
  4        WAITING       RUN:cpu             1             1
  5        WAITING       RUN:cpu             1             1
  6        WAITING          DONE                           1
  7        WAITING          DONE                           1
  8*   RUN:io_done          DONE             1          
  9         RUN:io          DONE             1          
 10        WAITING          DONE                           1
 11        WAITING          DONE                           1
 12        WAITING          DONE                           1
 13        WAITING          DONE                           1
 14        WAITING          DONE                           1
 15*   RUN:io_done          DONE             1          

Stats: Total Time 15
Stats: CPU Busy 8 (53.33%)
Stats: IO Busy  10 (66.67%)


[root@3d4481b4da9f c-test]# ./process-run.py -s 1 -l 3:50,3:50 -c -p -S SWITCH_ON_END
Time        PID: 0        PID: 1           CPU           IOs
  1        RUN:cpu         READY             1          
  2         RUN:io         READY             1          
  3        WAITING         READY                           1
  4        WAITING         READY                           1
  5        WAITING         READY                           1
  6        WAITING         READY                           1
  7        WAITING         READY                           1
  8*   RUN:io_done         READY             1          
  9         RUN:io         READY             1          
 10        WAITING         READY                           1
 11        WAITING         READY                           1
 12        WAITING         READY                           1
 13        WAITING         READY                           1
 14        WAITING         READY                           1
 15*   RUN:io_done         READY             1          
 16           DONE       RUN:cpu             1          
 17           DONE       RUN:cpu             1          
 18           DONE       RUN:cpu             1          

Stats: Total Time 18
Stats: CPU Busy 8 (44.44%)
Stats: IO Busy  10 (55.56%)


[root@3d4481b4da9f c-test]# ./process-run.py -s 1 -l 3:50,3:50 -c -p -S SWITCH_ON_IO
Time        PID: 0        PID: 1           CPU           IOs
  1        RUN:cpu         READY             1          
  2         RUN:io         READY             1          
  3        WAITING       RUN:cpu             1             1
  4        WAITING       RUN:cpu             1             1
  5        WAITING       RUN:cpu             1             1
  6        WAITING          DONE                           1
  7        WAITING          DONE                           1
  8*   RUN:io_done          DONE             1          
  9         RUN:io          DONE             1          
 10        WAITING          DONE                           1
 11        WAITING          DONE                           1
 12        WAITING          DONE                           1
 13        WAITING          DONE                           1
 14        WAITING          DONE                           1
 15*   RUN:io_done          DONE             1          

Stats: Total Time 15
Stats: CPU Busy 8 (53.33%)
Stats: IO Busy  10 (66.67%)
```
