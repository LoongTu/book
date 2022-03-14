# Process API

在这个作业中，你要熟悉一下刚读过的进程管理 API。别担心，它比听起来更有趣！如果你找到尽可能多的时间来编写代码，通常会增加成功的概率，为什么不现在就开始呢？

## 问题 1

1、编写一个调用 fork() 的程序。在调用之前，让主进程访问一个变量（例如 x）并将其值设置为某个值(例如 100)。子进程中的变量有什么值？当子进程和父进程都改变 x 的值时，变量会发生什么？

**答案：**

父进程和子进程修改变量不会互相影响。

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int main(int argc, char *argv[])
{
    int x = 100;

    int ret = fork();
    if (ret == -1)
    {
        return 1;
    }
    else if (ret == 0)
    {
        sleep(3);
        printf("i am child, pid: %d. x: %d\n", getpid(), x);

        x = 102;
        printf("i am child, pid: %d. x: %d\n", getpid(), x);
    }
    else
    {
        x = 1;
        printf("i am parent, pid: %d. my child: %d, x: %d\n", getpid(), ret, x);

        int wc = wait(NULL);
        printf("i am parent, pid: %d. my child: %d, wc: %d, x: %d\n", getpid(), ret, wc, x);
    }

    return 0;
}

```

输出：

```shell
[root@localhost ~]# ./main
i am parent, pid: 24706. my child: 24707, x: 1
i am child, pid: 24707. x: 100
i am child, pid: 24707. x: 102
i am parent, pid: 24706. my child: 24707, wc: 24707, x: 1
```

## 问题 2

2、编写一个打开文件的程序（使用 open 系统调用），然后调用 fork 创建一个新进程。子进程和父进程都可以访问 open() 返回的文件描述符吗？当它们并发（即同时）写入文件时，会发生什么？

**答案：**

父进程和子进程都可以访问文件描述符，并发写时，都可以写入。

```c
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>

int main(int argc, char *argv[])
{
    int fd = open("test.txt", O_RDWR | O_CREAT | O_TRUNC | O_APPEND, S_IRUSR | S_IWUSR);
    if (fd < 0)
    {
        return 1;
    }

    int ret = fork();
    if (ret == -1)
    {
        return 1;
    }
    else if (ret == 0)
    {
        write(fd, "i am child\n", 12);
    }
    else
    {
        write(fd, "i am parent\n", 13);
    }

    close(fd);
    return 0;
}

```

输出：

```shell
[root@localhost ~]# ./main
[root@localhost ~]# cat test.txt
i am parent
i am child
```

## 问题 3

3、使用 fork() 编写另一个程序。子进程应打印“hello”，父进程应打印“goodbye”。你应该尝试确保子进程始终先打印。你能否不在父进程调用 wait() 而做到这一点呢？

**答案：**

主进程调用 sleep。

## 问题 4

4、编写一个调用 fork() 的程序，然后调用某种形式的 exec() 来运行程序 /bin/ls。看看是否可以尝试 exec 的所有变体,包括 execl()、 execle()、 execlp()、 execv()、 execvp() 和  execve()，为什么同样的基本调用会有这么多变种？

**答案：**

不同的 exec() 的区别：

- l 代表 list（以 vararg 的形式传入，需要添加 NULL 作为参数结尾)；

- v 代表 vector（以 char * argv[] 的形式传入参数)；

- e 代表 environment；

- p 指定 filename，程序自动在 PATH 下找对应的程序；

```c
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <errno.h>

int main(int argc, char *argv[])
{
    char *const arg[] = {"ls", "-a", "-l", "-h", NULL}; // argv[0] 需要设置为将要运行的 executable filename.
    char *const envp[] = {"PATH=/bin:/usr/bin", NULL};

    int ret = fork();
    if (ret == -1)
    {
        return 1;
    }
    else if (ret == 0)
    {
        if (execl("/bin/ls", "ls", "-a", "-l", "-h", NULL) < 0)
        // if (execv("/bin/ls", arg) < 0)
        // if (execle("/bin/ls", "ls", "-a", "-l", "-h", NULL, envp) < 0)
        // if (execve("/bin/ls", arg, envp) < 0)
        // if (execlp("ls", "ls", "-a", "-l", "-h", NULL) < 0)
        // if (execvp("ls", arg) < 0)
        {
            printf("exec error: %s\n", strerror(errno));
        }
    }

    return 0;
}

```

## 问题 5

5、现在编写一个程序，在父进程中使用 wait()，等待子进程完成。wait() 返回什么？如果你在子进程中使用 wait() 会发生什么？

**答案：**

进程一旦调用了 wait，就会立即阻塞自己，由 wait 自动分析当前进程的某个子进程是否已经退出。如果让它找到了这样一个已经变成僵尸的子进程，wait 就会收集这个子进程的信息，并把它彻底销毁后返回；如果没有找到这样一个子进程，wait 就会一直阻塞在这里，直到有一个出现为止。

如果成功，wait 会返回被收集的子进程的 pid。如果调用进程没有子进程，调用就会失败，此时 wait 返回 -1，同时 errno 被置为 ECHILD。

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <unistd.h>
#include <errno.h>
#include <sys/wait.h>

int main(int argc, char *argv[])
{
    int ret = fork();
    if (ret == -1)
    {
        return 1;
    }
    else if (ret == 0)
    {
        sleep(1);
        int wc = wait(NULL);
        if (wc == -1)
        {
            printf("error: %s\n", strerror(errno));
        }
        printf("i am child, pid: %d, wc: %d\n", getpid(), wc);
        return 2;
    }
    else
    {
        int status = 0;
        int wc = wait(&status);
        printf("i am parent, pid: %d, my child: %d, wc: %d, status: %d\n",
               getpid(), ret, wc, WEXITSTATUS(status));
    }

    return 0;
}

```

输出：

```shell
error: No child processes
i am child, pid: 32427, wc: -1
i am parent, pid: 32426, my child: 32427, wc: 32427, status: 2
```

## 问题 6

6、对前一个程序稍作修改，这次使用 waitpid() 而不是 wait()。什么时候 waitpid() 会有用？

**答案：**

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>

int main(int argc, char *argv[])
{
    int ret = fork();
    if (ret == -1)
    {
        return 1;
    }
    else if (ret == 0)
    {
        sleep(1);
        printf("i am child, pid: %d\n", getpid());
        return 2;
    }
    else
    {
        int status = 0;
        int wc = waitpid(ret, &status, WUNTRACED);
        printf("i am parent, pid: %d, my child: %d, wc: %d, status: %d\n",
               getpid(), ret, wc, WEXITSTATUS(status));
    }

    return 0;
}

```

## 问题 7

7、编写一个创建子进程的程序，然后在子进程中关闭标准输出（STDOUT_FILENO）。如果子进程在关闭描述符后调用 printf() 打印输出，会发生什么？

**答案：**

关闭标准输出后看不到打印内容。

## 问题 8

8、编写一个程序，创建两个子进程，并使用 pipe() 系统调用，将一个子进程的标准输出连接到另一个子进程的标准输入。

**答案：**

调用 pipe 将当前进程空闲的 fd 最小的两个存储到fd[]中，默认情况下 > 2，因为 0、1、2 分配给了 stdin、stdout、stderr。

pipe 的两端需要分别关闭 fd[0] 和 fd[1]。

```c
#include <stdio.h>
#include <unistd.h>

int main(int argc, char *argv[])
{
    int fd[2];
    if (pipe(fd) < 0)
    {
        printf("pipe error\n");
    }
    printf("read_fd: %d, write fd: %d\n", fd[0], fd[1]);

    int ret = fork();
    if (ret == -1)
    {
        return 1;
    }
    else if (ret == 0)
    {
        close(fd[1]);
        char read_str[20] = {};
        int num = read(fd[0], read_str, sizeof(read_str));
        printf("read %d bytes from parent: %s\n", num, read_str);
    }
    else
    {
        close(fd[0]);
        char write_str[] = "hello, my child";
        write(fd[1], write_str, sizeof(write_str));
    }

    return 0;
}

```

输出：

```shell
read_fd: 3, write fd: 4
read 16 bytes from parent: hello, my child
```
