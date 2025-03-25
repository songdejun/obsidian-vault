# 环境配置
我使用的工具链是**WSL+VSCode**，主要参考了：[6.S081 / Fall 2020](https://pdos.csail.mit.edu/6.828/2020/tools.html)这个是官方网站，[操作系统实验Lab 0:实验环境搭建(MIT 6.S081 FALL 2020)-CSDN博客](https://blog.csdn.net/weixin_48283247/article/details/119611432?spm=1001.2014.3001.5502)。

按照官方网站配置环境我遇到了一些问题：
- xv6编译时报错
  ```shell
  user/sh.c:58:1: error: infinite recursion detected [-Werror=infinite-recursi
  ```
  解决方法是[MIT6.s081 编译QEMU中的错误_mit6.s081源文件报错-CSDN博客](https://blog.csdn.net/weixin_43647509/article/details/131953532)。d
- 执行 `make qemu` 卡住
  ```shell
 qemu-system-riscv64 -machine virt -bios none -kernel kernel/kernel -m 128M -smp 3-nographic -drive file=fs.img, if=none, format=raw, id=x0 -device virtio-blk-devi ce,drive=x0, bus=virtio-mmio-bus.0
  ```
  如果xv6源代码来自`git clone git://github.com/mit-pdos/xv6-riscv.git`，那解决方法就是将qemu版本更换置4.1.0，详情参考[XV6——运行make qemu时卡住问题_make qemu卡住-CSDN博客](https://blog.csdn.net/m0_68175178/article/details/137376004)。如果源代码来自`git clone https://github.com/mit-pdos/xv6-riscv.git`，那将qemu更新至7.2.0，详情参考[xv6 kernel doesn't boot: stuck at "make qemu". · Issue #262 · mit-pdos/xv6-riscv](https://github.com/mit-pdos/xv6-riscv/issues/262)。

总之最好先试一下完全参考[操作系统实验Lab 0:实验环境搭建(MIT 6.S081 FALL 2020)-CSDN博客](https://blog.csdn.net/weixin_48283247/article/details/119611432?spm=1001.2014.3001.5502)这个，我在Ubuntu 20.04和Debian 12.7参照这个教程都没问题。如果无法从`git://github.com/mit-pdos/xv6-riscv.git`克隆下源代码，那就从`https://github.com/mit-pdos/xv6-riscv.git`把源代码克隆下来并记得**更新qemu版本到7.2.0**。

在配置工具链时如果觉得翻墙下载太慢可以看[riscv-gnu-toolchain gitee镜像 国内镜像 加速下载-CSDN博客](https://blog.csdn.net/Galois4684/article/details/145218281?spm=1001.2014.3001.5502)，我把riscv-gnu-toolchain的子模块在gitee找了替代。

---
从[MIT 6.S081: Operating System Engineering - CS自学指南](https://csdiy.wiki/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/MIT6.S081/)还有许多其它可以参考的材料。

# Lab 1 Utilities

## 1. sleep
```c
#include "kernel/types.h"
#include "user/user.h"

int
main(int argc, char *argv[]){
	if (argc <  2){
		fprintf(2, "Usage: sleep...\n");
		exit(1);
	}

	sleep(atoi(argv[1]));

	exit(0);
}
```

## 2. pingpong
```c
#include "kernel/types.h"
#include "user/user.h"

int
main(int argc, char *argv[]){


	int pf[2], ps[2];

	pipe(pf);
	pipe(ps);

	int pid = fork();

	// 子进程
	if (pid == 0){
		// 关闭子->父管道读端口
		// 关闭父->子管道写端口
		close(ps[0]);
		close(pf[1]);

		char buffer[10];
        read(pf[0], buffer, 4);
        buffer[4] = 0;
        printf("%d: received %s\n", getpid(), buffer);
        write(ps[1], "pong", 4);
	}
	// 父进程
	else {
		// 关闭子->父管道写端口
		// 关闭父->子管道读端口
		close(ps[1]);
		close(pf[0]);
		
		char buffer[10];
        write(pf[1], "ping", 4);
        read(ps[0], buffer, 4);
        buffer[4] = 0;
        printf("%d: received %s\n", getpid(), buffer);
	}

	exit(0);
}
```

## 3. primes
```c
#include "kernel/types.h"
#include "user/user.h"

int decimal_atoi(char num[]) {
    return (num[0] - '0') * 10 + (num[1] - '0');
}

void decimal_itoa(char str[], int num) {
    str[0] = '0' + (num / 10);
    str[1] = '0' + (num % 10);
}

void mp_prime_sieve(int rfd, int process_cnt) {
    if (process_cnt >= 12) {
        close(rfd);
        exit(0);
    }

    int p[2];
    pipe(p);
    int pid = fork();

    if (pid == 0) {  // Child process
        close(p[1]);
        mp_prime_sieve(p[0], process_cnt + 1);
    } else {  // Parent process
        close(p[0]);
        char buf[2];
        while (1) {
            int n = read(rfd, buf, 2);
            if (n == 0) {
                close(p[1]);
                wait(0);
                exit(0);
            }
            int base = decimal_atoi(buf);
            fprintf(1, "prime %d\n", base);

            while (1) {
                n = read(rfd, buf, 2);
                if (n == 0) {
                    close(p[1]);
                    wait(0);
                    exit(0);
                }
                int cand = decimal_atoi(buf);
                if (cand % base != 0) {
                    write(p[1], buf, 2);
                }
            }
        }
    }
}

int main(int argc, char* argv[]) {
    if (argc != 1) {
        fprintf(2, "Usage: primes...\n");
        exit(1);
    }

    int p[2];
    pipe(p);
    char buf[2];

    for (int i = 2; i <= 35; i++) {
        decimal_itoa(buf, i);
        write(p[1], buf, 2);
    }

    int pid = fork();
    if (pid == 0) {
        close(p[1]);  // Close write end
        mp_prime_sieve(p[0], 1);
    } else {
        close(p[0]);
        close(p[1]);
    }

    wait(0);
    exit(0);
}
```

## 4. find
```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"
#include "kernel/fs.h"
#include "kernel/fcntl.h"

void
find(char dpath[], char fname[])
{
    char buf[512], *p;
    struct stat st;
    struct dirent de;
    int fd;

    if(strlen(dpath) + 1 + DIRSIZ + 1 > sizeof buf){
      fprintf(2, "find: path too long\n");
      return;
    }

    if ((fd = open(dpath, O_RDONLY)) < 0) {
        fprintf(2, "find: cannot open %s\n", dpath);
        return;
    }

    if (fstat(fd, &st) < 0) {
        fprintf(2, "find: cannot stat %s - a", dpath);
        close(fd);
        return;
    }

    strcpy(buf, dpath);
    p = buf + strlen(buf);
    *p++ = '/';
    // fprintf(1, "prefix is %s\n", buf);

    while (read(fd, &de, sizeof de) ==  sizeof de) {
        // inum = 0 表示一个错误或无效的状态, 比如试图访问一个不存在的文件或
        // 目录, 可能会返回一个 inum = 0 的 inode 引用
        if (de.inum == 0 || strcmp(de.name, ".") == 0 
                                        || strcmp(de.name, "..") == 0)
            continue;
        
        memmove(p, de.name, DIRSIZ);
        p[DIRSIZ] = 0;
        // fprintf(2, "this file path is %s\n", buf);
        if (stat(buf, &st) < 0) {
            fprintf(2, "find: cannot stat %s - b\n", buf);
            return;
        }
        switch (st.type) {
            case T_DIR: 
                // fprintf(1, "%s is a directory\n", buf);
                find(buf, fname);
                break;
            default:
                // fprintf(1, "%s %s\n", fname, de.name);
                if (strcmp(fname, de.name) == 0) {
                    fprintf(1, "%s\n", buf);
                }
        }

    }

    return;
}

int
main (int argc, char *argv[]) {
    if (argc != 3) {
        fprintf(2, "Usage: find...\n");
        exit(1);
    }

    if (argv[1][0] == '.')

    find(argv[1], argv[2]);

    exit(0);
}
```

## 5. xargs
```c
// Lab Xv6 and Unix utilities
// xargs.c

#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"
#include "kernel/param.h"

#define MAXN 1024

int
main(int argc, char *argv[])
{
    // 如果参数个数小于 2
    if (argc < 2) {
        // 打印参数错误提示
        fprintf(2, "usage: xargs command\n");
        // 异常退出
        exit(1);
    }
    // 存放子进程 exec 的参数
    char * argvs[MAXARG];
    // 索引
    int index = 0;
    // 略去 xargs ，用来保存命令行参数
    for (int i = 1; i < argc; ++i) {
        argvs[index++] = argv[i];
    }
    // 缓冲区存放从管道读出的数据
    char buf[MAXN] = {"\0"};
    
    int n;
	// 0 代表的是管道的 0，也就是从管道循环读取数据
    while((n = read(0, buf, MAXN)) > 0 ) {
        // 临时缓冲区存放追加的参数
		char temp[MAXN] = {"\0"};
        // xargs 命令的参数后面再追加参数
        argvs[index] = temp;
        // 内循环获取追加的参数并创建子进程执行命令
        for(int i = 0; i < strlen(buf); ++i) {
            // 读取单个输入行，当遇到换行符时，创建子线程
            if(buf[i] == '\n') {
                // 创建子线程执行命令
                if (fork() == 0) {
                    exec(argv[1], argvs);
                }
                // 等待子线程执行完毕
                wait(0);
            } else {
                // 否则，读取管道的输出作为输入
                temp[i] = buf[i];
            }
        }
    }
    // 正常退出
    exit(0);
}
```

# Lab 2 Utilities

## 0. Xv6 启动分析

entry.S: 设置每个硬件线程(hart)的 sp(stack pointer, 栈顶指针) 寄存器, 每个栈帧 4096 B.

start.c: 

## 1. System call tracing

