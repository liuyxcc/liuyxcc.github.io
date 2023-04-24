---
title: 内存映射的注意事项
updated: '2023-04-25 00:45:20'
excerpt: >-
  q_如果对mmap的返回值(ptr)做操作(ptr)munmap是否能够成功?a_voidptr=mmap()_ptr_可以对其进行操作munmap(ptrlen)_错误要保存原来的地址才可正确释放内存q_如果open时o_rdonlymmap时prot参数指定prot_read_prot_write会怎样?a_错误返回map_failed。open()函数中的权限建议和prot参数的权限保持一致。q_如果文件偏移量为会怎样?a_偏移量必须是k的整数倍返回map_failedq_mmap什么情况下会调用失
tags:
  - Linux
categories:
  - '04'
permalink: /post/precautions-for-memory-mapping-1oehwj.html
comments: true
---



1. Q：如果对mmap的返回值(ptr)做++操作(ptr++), munmap是否能够成功?

    A：  
    void * ptr = mmap(...);  
    ptr++;  可以对其进行++操作  
    munmap(ptr, len);   // 错误,要保存原来的地址才可正确释放内存

2. Q：如果open时O_RDONLY, mmap时prot参数指定PROT_READ | PROT_WRITE会怎样?  
    A：错误，返回MAP_FAILED。open()函数中的权限建议和prot参数的权限保持一致。

3. Q：如果文件偏移量为1000会怎样?  
    A：偏移量必须是4K的整数倍，返回MAP_FAILED

4. Q：mmap什么情况下会调用失败?

    A：

    * 第二个参数：length = 0
    * 第三个参数：prot

      * 只指定了写权限
      * prot PROT_READ | PROT_WRITE  
        第5个参数fd 通过open函数时指定的 O_RDONLY / O_WRONLY

5. Q：可以open的时候O_CREAT一个新文件来创建映射区吗?

    A：

    * 可以的，但是创建的文件的大小如果为0的话，肯定不行
    * 可以对新的文件进行扩展

      * lseek()
      * truncate()

6. Q：mmap后关闭文件描述符，对mmap映射有没有影响？

    A：

    ```c
    int fd = open("XXX");
    mmap(,,,,fd,0);
    close(fd);
    ```

    映射区还存在，创建映射区的fd被关闭，没有任何影响。

7. Q：对ptr越界操作会怎样？  
    A：void * ptr = mmap(NULL, 100,,,,,);  
    4K  
    越界操作操作的是非法的内存 -> 段错误
