# Coreダンプ調査（デッドロック）

## 動機
プログラムがmutexによって複数のスレッドがデッドロックした。どのスレッドが最初にロックしたかを知りたい。

## 回答
gdbでプロセスにアタッチするかcoreダンプを調査し、該当mutexのpthread_mutex_tの__data.__ownerを確認する。
これがロックを保持しているLWP番号となっている。

もし、適切にglibcのdebuginfoがインストールされていればpthread_mutex_lock()の引数をprintコマンドで表示すればわかる。
インストールされていない場合でも、r8レジスタの値+0Chのアドレスの値で分かる（x86_64の場合）。

## glibcのdebuginfoがインストールされている場合
適切なglibcのdebuginfoがインストールされていると、pthreadのソースファイルなどもバックトレースに関連付けて表示できる。
該当のpthread_mutex_tにprintコマンドを使うと内容が見え、__data.__ownerが確認できる。

    (gdb) thread apply all bt

    Thread 4 (Thread 0x7f8181fd8700 (LWP 2000)):
    #0  __lll_lock_wait () at ../nptl/sysdeps/unix/sysv/linux/x86_64/lowlevellock.S:136
    #1  0x00007f81823765d8 in _L_lock_854 () from /lib64/libpthread.so.0
    #2  0x00007f81823764a7 in __pthread_mutex_lock (mutex=0x6b38e0) at pthread_mutex_lock.c:61
    #3  0x0000000000400760 in locker_thread_1 (arg=0x0) at testprogram6.c:9
    #4  0x00007f81825f43e4 in ?? () from /lib64/libglib-2.0.so.0
    #5  0x00007f8182374aa1 in start_thread (arg=0x7f8181fd8700) at pthread_create.c:301
    #6  0x00007f81820c1c4d in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:115

    Thread 3 (Thread 0x7f81815d7700 (LWP 2001)):
    #0  __lll_lock_wait () at ../nptl/sysdeps/unix/sysv/linux/x86_64/lowlevellock.S:136
    #1  0x00007f81823765d8 in _L_lock_854 () from /lib64/libpthread.so.0
    #2  0x00007f81823764a7 in __pthread_mutex_lock (mutex=0x6b38e0) at pthread_mutex_lock.c:61
    #3  0x000000000040078e in locker_thread_2 (arg=0x0) at testprogram6.c:17
    #4  0x00007f81825f43e4 in ?? () from /lib64/libglib-2.0.so.0
    #5  0x00007f8182374aa1 in start_thread (arg=0x7f81815d7700) at pthread_create.c:301
    #6  0x00007f81820c1c4d in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:115

    Thread 2 (Thread 0x7f8180bd6700 (LWP 2002)):
    #0  __lll_lock_wait () at ../nptl/sysdeps/unix/sysv/linux/x86_64/lowlevellock.S:136
    #1  0x00007f81823765d8 in _L_lock_854 () from /lib64/libpthread.so.0
    #2  0x00007f81823764a7 in __pthread_mutex_lock (mutex=0x6b38e0) at pthread_mutex_lock.c:61
    #3  0x00000000004007bc in locker_thread_3 (arg=0x0) at testprogram6.c:25
    #4  0x00007f81825f43e4 in ?? () from /lib64/libglib-2.0.so.0
    #5  0x00007f8182374aa1 in start_thread (arg=0x7f8180bd6700) at pthread_create.c:301
    #6  0x00007f81820c1c4d in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:115

    Thread 1 (Thread 0x7f8182ec1700 (LWP 1999)):
    #0  0x00007f81820b480d in read () at ../sysdeps/unix/syscall-template.S:82
    #1  0x00007f818204abc8 in _IO_new_file_underflow (fp=0x7f8182367f60) at fileops.c:598
    #2  0x00007f818204c6ce in _IO_default_uflow (fp=0x7f8182367f60) at genops.c:440
    #3  0x00007f8182047d0c in getchar () at getchar.c:38
    #4  0x0000000000400873 in main (argc=1, argv=0x7ffc19749798) at testprogram6.c:39
    (gdb) thread 4
    [Switching to thread 4 (Thread 0x7f8181fd8700 (LWP 2000))]#0  __lll_lock_wait () at ../nptl/sysdeps/unix/sysv/linux/x86_64/lowlevellock.S:136
    136     2:      movl    %edx, %eax
    (gdb) f 2
    #2  0x00007f81823764a7 in __pthread_mutex_lock (mutex=0x6b38e0) at pthread_mutex_lock.c:61
    61            LLL_MUTEX_LOCK (mutex);
    (gdb) p *mutex
    $2 = {__data = {__lock = 2, __count = 0, __owner = 2002, __nusers = 1, __kind = 0, __spins = 0, __list = {__prev = 0x0, __next = 0x0}}, __size = 
        "\002\000\000\000\000\000\000\000\322\a\000\000\001", '\000' <repeats 26 times>, __align = 2}

Thread 2～4で同じmutexに対してロックしているのでデッドロックしているのかもしれない。該当のmutexに対してprintコマンドを実行すると、__data.__ownerが2002であったので最初にロックし開放していないのはThread 2であることが分かる。

## debuginfoがインストールされていない場合
適切なglibcのdebuginfoがインストールされてい場合でも、pthread_mutex_lockが第１引数(rdiレジスタに入る)を、r8レジスタに退避しているのでこれを利用すれば概ねわかる。

    (gdb) thread apply all bt

    Thread 4 (Thread 0x7f8181fd8700 (LWP 2000)):
    #0  0x00007f818237b334 in __lll_lock_wait () from /lib64/libpthread.so.0
    #1  0x00007f81823765d8 in _L_lock_854 () from /lib64/libpthread.so.0
    #2  0x00007f81823764a7 in pthread_mutex_lock () from /lib64/libpthread.so.0
    #3  0x0000000000400760 in locker_thread_1 (arg=0x0) at testprogram6.c:9
    #4  0x00007f81825f43e4 in ?? () from /lib64/libglib-2.0.so.0
    #5  0x00007f8182374aa1 in start_thread () from /lib64/libpthread.so.0
    #6  0x00007f81820c1c4d in clone () from /lib64/libc.so.6

    Thread 3 (Thread 0x7f81815d7700 (LWP 2001)):
    #0  0x00007f818237b334 in __lll_lock_wait () from /lib64/libpthread.so.0
    #1  0x00007f81823765d8 in _L_lock_854 () from /lib64/libpthread.so.0
    #2  0x00007f81823764a7 in pthread_mutex_lock () from /lib64/libpthread.so.0
    #3  0x000000000040078e in locker_thread_2 (arg=0x0) at testprogram6.c:17
    #4  0x00007f81825f43e4 in ?? () from /lib64/libglib-2.0.so.0
    #5  0x00007f8182374aa1 in start_thread () from /lib64/libpthread.so.0
    #6  0x00007f81820c1c4d in clone () from /lib64/libc.so.6

    Thread 2 (Thread 0x7f8180bd6700 (LWP 2002)):
    #0  0x00007f818237b334 in __lll_lock_wait () from /lib64/libpthread.so.0
    #1  0x00007f81823765d8 in _L_lock_854 () from /lib64/libpthread.so.0
    #2  0x00007f81823764a7 in pthread_mutex_lock () from /lib64/libpthread.so.0
    #3  0x00000000004007bc in locker_thread_3 (arg=0x0) at testprogram6.c:25
    #4  0x00007f81825f43e4 in ?? () from /lib64/libglib-2.0.so.0
    #5  0x00007f8182374aa1 in start_thread () from /lib64/libpthread.so.0
    #6  0x00007f81820c1c4d in clone () from /lib64/libc.so.6

    Thread 1 (Thread 0x7f8182ec1700 (LWP 1999)):
    #0  0x00007f81820b480d in read () from /lib64/libc.so.6
    #1  0x00007f818204abc8 in _IO_new_file_underflow () from /lib64/libc.so.6
    #2  0x00007f818204c6ce in _IO_default_uflow_internal () from /lib64/libc.so.6
    #3  0x00007f8182047d0c in getchar () from /lib64/libc.so.6
    #4  0x0000000000400873 in main (argc=1, argv=0x7ffc19749798) at testprogram6.c:39

    (gdb) thread 4
    [Switching to thread 4 (Thread 0x7f8181fd8700 (LWP 2000))]#2  0x00007f81823764a7 in pthread_mutex_lock () from /lib64/libpthread.so.0

    (gdb) x/4wx $rdi
    0x6b38e0:       0x00000002      0x00000000      0x000007d2      0x00000001

適切なdebuginfoがなかったため、pthread_mutex_tを直接確認できなかった。
しかし、pthread_mutex_lock()が第１引数をr8に退避すること、__data.__ownerがオフセット0Chであることが分かっていれば、
どのスレッドがロックしたままなのか知ることができる。
pthread_mutex_lock()の挙動は、disassembleコマンドで、逆アセンブルできる。
__data.__ownerのオフセットは、別途pthreadのソースから調べられる。
上記の場合、0x07D2=2002なので、Thread2がロックを開放していないことが分かる。
