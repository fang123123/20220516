# 计算机的起源

图灵机\==>通用图灵机==>冯诺依曼存储程序结构

##### 图灵机

通过机器模拟人使用纸笔进行数学运算的过程

> 人眼看==>执行器读取指令、数据
>
> 人脑算==>根据特定指令，执行特定逻辑
>
> 写结果==>执行器写回数据

局限：执行器只能执行特定命令，即是一个==有限状态机==

##### 通用图灵机

可以模拟任意的图灵机。==只是模拟，由于现实中存储是有限的，所以无法跨越有限状态机的界限==

##### 冯诺依曼存储程序结构

将程序和数据放到计算机内部的存储器中，计算机在程序的控制下一步一步==取指执行==



#  计算机启动过程

##### 相关文档

[计算机启动](https://blog.csdn.net/weixin_46645613/article/details/114847373)

[全网最嘴硬讲解计算机的启动过程](https://blog.csdn.net/pdcfighting/article/details/109881716)

[计算机的启动过程(详细)](https://blog.csdn.net/enlaihe/article/details/84955026)



##### 计算机启动过程

对于x86系统而言，开机时，CPU处于实模式(只有1M大小，最初的计算机就只有1M)。设置`CS(段基址寄存器)==>0xFFFF,PC(偏移地址寄存器)==>0x0000`，由硬件实现。计算地址为`CS<<4+PC=0xFFFF0`

1. 读取BIOS(Basic Input Ouput System, 放入主板ROM中)
   1. POST(Power On Self Test，硬件自检)
   2. 加载I/O驱动、中断服务
   3. CMOS设置程序(通过特殊热键启动，猜测应该就是进入BIOS面板)
   4. 加载主引导记录
      1. BIOS根据`启动顺序`(BIOS面板里可以设置启动顺序)逐步加载外部设备的0盘0道0扇区，该扇区一共512B。当该扇区末尾两个字节分别为0x55和0xaa，BIOS认为该扇区为启动区，即主引导记录。
      2. ==将主引导记录加载到0x7c00处，0~0x7bFF放置操作系统==，设置`CS==>0x07c0,PC==>0x0000`
2. 读取MBR(Master boot record,主引导记录)

> MBR内容：
>
> 1. 446B的引导程序
> 2. 64B的分区表，分为4项，每个分区表16B(一个硬盘最多四个分区)
>    1. 分区表第一个字节只能是0x80和0。0x80表示为活动分区，只能有一个分区为活动分区，即主分区。==读取主分区==
>    2. 其余位置记录了分区实际位置
>    3. 最后四个字节记录主分区的扇区总数
>    4. $分区大小=扇区总数*扇区大小=2^{32}*512 B=2 TB$
> 3. 0x55、0xaa

3. 硬盘启动

> 方式一：卷引导记录
>
> 将控制权交给主分区，加载主分区的第一个扇区VBR(Volume boot record,卷引导记录),即操作系统内核加载器。
>
> 方式二：扩展分区和逻辑分区
>
> 当需要更多的分区时，将一个主分区作为扩展分区，扩展分区中可以包含多个逻辑分区。扩展分区的第一个扇区，叫做EBR(Extended boot record,扩展引导记录)，EBR包含一张64B的逻辑分区表，最多只有两项。
>
> 方式三：启动管理器
>
> 主引导记录前446B的引导程序中包含一个启动管理器，由用户选择启动特定的操作系统。Linux环境中，目前流行的启动管理器是Grub

boot、setup、system是连续的

<img src="D:/20220516/ComputerOS/ComputerOS.assets/image-20220517153029377.png" alt="image-20220517153029377" style="zoom: 33%;" />

4. 加载操作系统加载程序

> 1. 加载操作系统到0x0000处
> 2. 启动保护模式
> 3. 获取硬件相关参数
> 4. 跳转到0x0000处执行操作系统

<img src="D:/20220516/ComputerOS/ComputerOS.assets/image-20220517154559926.png" alt="image-20220517154559926" style="zoom: 33%;" />

5. 加载操作系统

操作系统的编译过程根据Makefile文件执行

<img src="D:/20220516/ComputerOS/ComputerOS.assets/image-20220517165814604.png" alt="image-20220517165814604" style="zoom:33%;" />



##### 实模式和保护模式

实模式切换为保护模式方式

<img src="D:/20220516/ComputerOS/ComputerOS.assets/image-20220517162941315.png" alt="image-20220517162941315" style="zoom:33%;" />

二者区别在于寻址方式不同

- 实模式寻址方式：`CX<<4+PC`，由于CX和PC都是16位寄存器，最大寻址空间为1MB。
- 保护模式寻址方式：CX为间接寻址，通过查询GDT(Global Description Table)获取实际段基址地址，CX和PC都是32位寄存器，最大寻址空间为4GB

<img src="D:/20220516/ComputerOS/ComputerOS.assets/image-20220517163043782.png" alt="image-20220517163043782" style="zoom:33%;" />

<center>保护模式寻址方式</center>

##### 不同类型的汇编

<img src="D:/20220516/ComputerOS/ComputerOS.assets/image-20220517170034457.png" alt="image-20220517170034457" style="zoom:33%;" />



##### 操作系统启动过程

1. 执行head.s初始化

使用32位汇编编写

<img src="D:/20220516/ComputerOS/ComputerOS.assets/image-20220517170859761.png" alt="image-20220517170859761" style="zoom:33%;" />

2. 执行main.c进入主逻辑

当main函数跳出后，操作系统死循环

<img src="D:/20220516/ComputerOS/ComputerOS.assets/image-20220517171243015.png" alt="image-20220517171243015" style="zoom:33%;" />

main函数中执行初始化逻辑

<img src="D:/20220516/ComputerOS/ComputerOS.assets/image-20220517171405532.png" alt="image-20220517171405532" style="zoom:33%;" />

 内存初始化

> 右移12，即4KB，一页

<img src="D:/20220516/ComputerOS/ComputerOS.assets/image-20220517171815057.png" alt="image-20220517171815057" style="zoom:33%;" />
