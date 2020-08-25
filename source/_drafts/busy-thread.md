---
title: CPU高负载
toc: true
tags: [线程,CPU Load]
---

在开发和运维系统过程中，难免会碰到CPU负载异常高的问题，第一印象是边界情况没处好，导致死循环。如何快速定位占用CPU的线程，甚至具体的代码块，对于解决线上问题非常关键。结合Linux自带的top命令和Java jstack可以轻松解决。

## 进程负载

Linux top 命令会展示系统当前的整体负载(load average)情况以及进程所占用的资源(PID、CPU、Memory等)，如下
```
top - 21:23:19 up 588 days,  3:17,  1 user,  load average: 3.28, 3.84, 4.55
Tasks: 417 total,   2 running, 415 sleeping,   0 stopped,   0 zombie
%Cpu(s):  8.4 us,  3.6 sy,  0.0 ni, 87.7 id,  0.0 wa,  0.0 hi,  0.3 si,  0.0 st
KiB Mem : 13141572+total,  4651624 free, 83008648 used, 43755452 buff/cache
KiB Swap:        0 total,        0 free,        0 used. 43701956 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 9161 admin     20   0  0.105t 0.065t  15768 S 1709.2 53.4 109998:58 java
17551 root      20   0 3690796  32624   2548 S  18.8  0.0 309:22.70 lsecd
26748 root      20   0  146408   2156   1376 R   6.2  0.0   0:00.01 top
    1 root      20   0 1403740 1.162g   1852 S   0.0  0.9 185:42.10 systemd
    2 root      20   0       0      0      0 S   0.0  0.0   5:18.52 kthreadd
    3 root      20   0       0      0      0 S   0.0  0.0  99:12.84 ksoftirqd/0
    5 root       0 -20       0      0      0 S   0.0  0.0   0:00.00 kworker/0:+
    8 root      rt   0       0      0      0 S   0.0  0.0  42:23.68 migration/0
    9 root      20   0       0      0      0 S   0.0  0.0   0:00.00 rcu_bh
   10 root      20   0       0      0      0 S   0.0  0.0   0:00.00 rcuob/0
   11 root      20   0       0      0      0 S   0.0  0.0   0:00.00 rcuob/1
   12 root      20   0       0      0      0 S   0.0  0.0   0:00.00 rcuob/2
```
### 线程负载

观察具体进程的线程负载，可以通过top -Hp pid。通过man top查看下 —H选项的含义:
```
-H  :Threads-mode operation
    Instructs top to display individual  threads.   Without  this
    command-line  option  a  summation  of  all  threads  in each
    process is shown.  Later this can be  changed  with  the  `H'
    interactive command.
```

top -Hp 9161 显示如下结果，通过交互命令P对线程按cpu占用排序
```
 PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
24030 admin     20   0 49.789g 0.021t   3208 R 70.8 17.4   8200:13 java
24027 admin     20   0 49.789g 0.021t   3208 R 70.1 17.4   8204:32 java
24020 admin     20   0 49.789g 0.021t   3208 R 69.4 17.4   8199:30 java
24034 admin     20   0 49.789g 0.021t   3208 R 69.4 17.4   8200:41 java
24018 admin     20   0 49.789g 0.021t   3208 R 69.1 17.4   8205:10 java
24024 admin     20   0 49.789g 0.021t   3208 R 69.1 17.4   8203:03 java
24023 admin     20   0 49.789g 0.021t   3208 R 68.4 17.4   8197:42 java
24036 admin     20   0 49.789g 0.021t   3208 R 68.4 17.4   8197:41 java
24037 admin     20   0 49.789g 0.021t   3208 R 68.4 17.4   8205:43 java
24029 admin     20   0 49.789g 0.021t   3208 R 68.1 17.4   8200:24 java
24026 admin     20   0 49.789g 0.021t   3208 R 67.8 17.4   8199:40 java
24019 admin     20   0 49.789g 0.021t   3208 R 67.1 17.4   8203:19 java
24032 admin     20   0 49.789g 0.021t   3208 R 66.8 17.4   8202:12 java
24016 admin     20   0 49.789g 0.021t   3208 R 66.4 17.4   8203:41 java
24025 admin     20   0 49.789g 0.021t   3208 R 66.4 17.4   8200:08 java
24031 admin     20   0 49.789g 0.021t   3208 R 66.1 17.4   8203:23 java
24028 admin     20   0 49.789g 0.021t   3208 R 65.4 17.4   8208:02 java
24017 admin     20   0 49.789g 0.021t   3208 R 65.1 17.4   8202:28 java
24022 admin     20   0 49.789g 0.021t   3208 R 64.8 17.4   8197:14 java
24033 admin     20   0 49.789g 0.021t   3208 R 64.8 17.4   8197:24 java
24038 admin     20   0 49.789g 0.021t   3208 R 62.8 17.4   8202:43 java
24035 admin     20   0 49.789g 0.021t   3208 R 62.5 17.4   8201:33 java
24021 admin     20   0 49.789g 0.021t   3208 R 61.5 17.4   8198:19 java
24039 admin     20   0 49.789g 0.021t   3208 S 23.6 17.4   4096:35 java
```

将高负载线程保存到tid.txt，利用单行命令
```
cat tid.txt|awk '{printf "%d,%x\n",$1,$1}'
```
将线程id 转化为十六进制，后续jstack tid采用hex 显示。结果如下:
```
24029,5ddd
24024,5dd8
24034,5de2
24022,5dd6
24030,5dde
24017,5dd1
24033,5de1
24018,5dd2
24032,5de0
24016,5dd0
24021,5dd5
24026,5dda
24038,5de6
24027,5ddb
24028,5ddc
24020,5dd4
24037,5de5
24019,5dd3
24023,5dd7
24035,5de3
24031,5ddf
24036,5de4
24025,5dd9
```

## 进程栈状态
利用jstack 获取进程的线程栈，命令jstack 9161 >stack.txt，利用grep 命令
```
 cat 9161.txt |grep -C 10 5ddd
```
查找目标线程栈如下:
```
"GC task thread#8 (ParallelGC)" os_prio=0 tid=0x00007f569002f800 nid=0x5dd8 runnable

"GC task thread#9 (ParallelGC)" os_prio=0 tid=0x00007f5690031000 nid=0x5dd9 runnable

"GC task thread#10 (ParallelGC)" os_prio=0 tid=0x00007f5690033000 nid=0x5dda runnable

"GC task thread#11 (ParallelGC)" os_prio=0 tid=0x00007f5690035000 nid=0x5ddb runnable

"GC task thread#12 (ParallelGC)" os_prio=0 tid=0x00007f5690036800 nid=0x5ddc runnable

"GC task thread#13 (ParallelGC)" os_prio=0 tid=0x00007f5690038800 nid=0x5ddd runnable

"GC task thread#14 (ParallelGC)" os_prio=0 tid=0x00007f569003a800 nid=0x5dde runnable

"GC task thread#15 (ParallelGC)" os_prio=0 tid=0x00007f569003c000 nid=0x5ddf runnable

"GC task thread#16 (ParallelGC)" os_prio=0 tid=0x00007f569003e000 nid=0x5de0 runnable

"GC task thread#17 (ParallelGC)" os_prio=0 tid=0x00007f5690040000 nid=0x5de1 runnable

"GC task thread#18 (ParallelGC)" os_prio=0 tid=0x00007f5690041800 nid=0x5de2 runnable
```

至次,可以确定CPU负载高是由于GC引起的，持续高说明可能存在内存泄漏/或JVM参数配置不合理的问题。

## Q&A

1. Jstack 会看到线程当前的状态，线程有哪些状态，状态之间如何转换?
 通过命令cat 9161.txt |grep ' java.lang.Thread.'，结果如下
 ```
  TIMED_WAITING (on object monitor),Object wait
  TIMED_WAITING (parking),Unsafe.park
  WAITING (parking)
  WAITING (on object monitor)
  RUNNABLE
  BLOCKED (on object monitor),synchronized 等待进入临界区
 ```
  所以常见的状态有:
   - TIME_WAITING
   - WAITING
   - RUNNABLE
   - BLOCKED
