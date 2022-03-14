# Direct execution

## 问题

测量作业是小型练习。你可以编写代码在真实机器上运行，从而测量操作系统或硬件性能的某些方面。这样的作业背后的想法是给你一点实际操作系统的实践经验。

在这个作业中，你将测量系统调用和上下文切换的成本。测量系统调用的成本相对容易。例如，你可以重复调用一个简单的系统调用（例如，执行 0 字节读取）并记下所花的时间。将时间除以迭代次数，就可以估计系统调用的成本。

你必须考虑的一件事是时钟的精确性和准确性。你可以使用的典型时钟是 gettimeofday()。详细信息请阅读手册。你会看到，gettimeofday() 返回自 1970 年以来的微秒时间。然而，这并不意味着时钟精确到微秒。测量 gettimeofday() 的连续调用，以了解时钟的精确度。这会告诉你为了获得一个好的测量结果，需要让空系统调用测试的迭代运行多少次。如果 gettimeofday() 对你来说不够精确，可以考虑利用 x86 机器提供的 rdtsc 指令。

测量上下文切换的成本有点棘手。Imbench 基准测试的实现方法，是在单个 CPU 上运行两个进程并在它们之间设置两个 UNIX 管道。管道只是 UNIX 系统中的进程可以相互通信的许多方式之一。

- 第一个进程向第一个管道写入数据，然后等待第二个管道的读取。

- 由于看到第一个进程等待从第二个管道读取内容，OS 将第一个进程置于阻塞状态，并切换到另一个进程。该进程从第一个管道读取数据，然后写入第二个管道。

- 第二个进程再次尝试从第一个管道读取时，它会阻塞，从而继续进行通信的往返循环。

通过反复测量这种通信的成本，Imbench 可以很好地估计上下文切换的成本。你可以尝试使用管道或其他通信机制（例如 UNIX 套接字），重新创建类似的东西。

在具有多个 CPU 的系统中，测量上下文切换成本有一点困难。在这样的系统上，你需要确保你的上下文切换进程处于同一个处理器上。幸运的是，大多数操作系统都会提供系统调用，让一个进程绑定到特定的处理器。例如，在 Linux 上，sched_setaffinity() 调用就是你要查找的内容。通过确保两个进程位于同一个处理器上，你就能确保在测量操作系统停止一个进程并在同一个 CPU 上恢复另一个进程的成本。

**1、执行 gettimeofday() 的平均耗时**

执行次数达到 100,000 量级的时候，测量结果波动较小。执行 gettimeofday() 平均耗时 0.033630us。

```c
#include <stdio.h>
#include <sys/time.h>

#define MAX_NUM 100000

int main()
{
    struct timeval tv_begin, tv_end;
    gettimeofday(&tv_begin, NULL);

    for (int i = 0; i < MAX_NUM; i++)
    {
        gettimeofday(&tv_end, NULL);
    }

    double diff = (tv_end.tv_sec - tv_begin.tv_sec) * 1000000 + tv_end.tv_usec - tv_begin.tv_usec;
    printf("gettimeofday() 平均每次执行时间为: %fus\n", diff / MAX_NUM);

    return 0;
}

```

**2、执行 0 字节的平均耗时**

利用前面的经验，0 字节读取同样也执行 100,000 次。执行 0 字节读取平均耗时 0.764220us。

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/time.h>

#define MAX_NUM 100000

int main()
{
    struct timeval tv_begin, tv_end;
    gettimeofday(&tv_begin, NULL);

    char buf[10] = {};
    for (int i = 0; i < MAX_NUM; i++)
    {
        read(0, buf, sizeof(buf));
    }

    gettimeofday(&tv_end, NULL);
    double diff = (tv_end.tv_sec - tv_begin.tv_sec) * 1000000 + tv_end.tv_usec - tv_begin.tv_usec;
    printf("0 byte read 平均耗时为: %fus\n", diff / MAX_NUM);

    return 0;
}

```

**3、Context switch 的平均耗时**

同样循环 100,000次，Context switch 平均耗时 2.129440us。

```c
#include <stdio.h>
#include <unistd.h>
#include <errno.h>
#include <sys/time.h>

#define __USE_GNU
#include <sched.h>

#define MAX_NUM 100000

int bind_cpu(unsigned int cpu_index)
{
    int cpus = sysconf(_SC_NPROCESSORS_CONF);
    if (cpu_index > cpus)
    {
        printf("out of range, cpus: %d, cpu_index %d\n", cpus, cpu_index);
        return -1;
    }

    cpu_set_t mask = {}, get = {};
    CPU_ZERO(&mask);
    CPU_ZERO(&get);
    CPU_SET(cpu_index, &mask);

    sched_setaffinity(getpid(), sizeof(cpu_set_t), &mask);
    sched_getaffinity(getpid(), sizeof(cpu_set_t), &get);
    if (!CPU_ISSET(cpu_index, &get))
    {
        printf("cpu %d is not in cpu set\n", cpu_index);
        return -1;
    }

    return 0;
}

int main()
{
    // 进程绑定到 CPU2.
    if (0 != bind_cpu(2))
    {
        return -1;
    }

    struct timeval tv_begin, tv_end;
    char str[20] = "hello";

    // 两个管道.
    int fds1[2] = {}, fds2[2] = {};
    pipe(fds1);
    pipe(fds2);

    int ret = fork();
    if (ret < 0)
    {
        return -1;
    }
    else if (ret == 0)
    {
        close(fds1[1]);
        close(fds2[0]);

        read(fds1[0], str, sizeof(str));
        for (int i = 0; i < MAX_NUM; i++)
        {
            write(fds2[1], str, sizeof(str));
            read(fds1[0], str, sizeof(str));
        }
    }
    else
    {
        close(fds1[0]);
        close(fds2[1]);

        gettimeofday(&tv_begin, NULL);
        write(fds1[1], str, sizeof(str));
        for (int i = 0; i < MAX_NUM; i++)
        {
            read(fds2[0], str, sizeof(str));
            write(fds1[1], str, sizeof(str));
        }

        gettimeofday(&tv_end, NULL);
        double diff = (tv_end.tv_sec - tv_begin.tv_sec) * 1000000 + tv_end.tv_usec - tv_begin.tv_usec;
        
        // 在一次循环中, 进程 1 切换到 OS, OS 切换到 进程 2, 进程 2 切换回 OS, OS 再切换到 进程 1, 上下文共切换 4 次.
        printf("context switch 平均耗时为: %fus\n", diff / MAX_NUM / 4);
    }

    return 0;
}

```
