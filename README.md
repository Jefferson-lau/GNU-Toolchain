# PLCT--liujiawei

RISCV GNU GCC 回归测试

## 1.软件的构建和测试

（1）搭建测试环境

```
硬件环境：x86-64电脑

软件环境：虚拟机+ubuntu20.04-64

网络环境：能够非常稳定流畅的访问github

存储空间：大于100G（gnu源码包8G，单次构建超14G；llvm更大源码80G左右？，总之存储空间大点好）
```

1.1：在VMware上安装虚拟机做一些网络环境设置。

1.2：下载源码

### 方法一：

```RISCV$    git clone --recursive https://github.com/riscv/riscv-gnu-toolchain.git```

### 由于国内访问GitHub速度较慢，所以推荐使用方法二

### 方法二：

```
1. 不用git clone的方法下载第一份源码；而是从镜像网址下载历史版本的tar压缩包;镜像地址：https://mirror.iscas.ac.cn/plct/
2. 将压缩包解压的gnu的源设置为riscv-gnu-toolchain
3. 从riscv-gnu-toolchain源上获取最新的更新
```

## 如果使用镜像网站则需要执行以下操作：

```如果从mirror上下载riscv-gnu-toolchain，需要fetch和merge：
 $wget https://mirror.iscas.ac.cn/plct/riscv-gnu-toolchain.20210207.tbz
 $tar xjvf riscv-gnu-toolchain.20210207.tbz
 $cd riscv-gnu-toolchain
 $git fetch origin master
 $git merge origin/master
 $git submodule update --init --recursive
```

1.3：安装环境依赖(如果还缺少库，请自行进行添加)。
```
sudo apt install autoconf automake autotools-dev curl python3 libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo gperf libtool patchutils bc zlib1g-dev libexpat-dev cmake ninja-build pkg-config libglib2.0-dev libpixman-1-dev python git libfdt-dev libncurses5-dev libncursesw5-dev device-tree-compiler
```

1.4：选择一个目录，作为RISC-V编译程序的存放位置。然后设置环境变量，此处的~/RISCV可以替换。注意，此处的变量需要在每次登陆时重新设置（或设置环境变量，Ubuntu系统的配置）。

```
mkdir RISCV
RISCV=~/RISCV
cd RISCV
```

#### 注意：我们把解压后的riscv-gnu-toolchain放到该目录下。

1.5：构建newlib函数库版本

```
cd riscv-gun-toolchain
$ makdir build && cd build
$ mkdir newlib && cd newlib 
$ ../../configure --prefix=$RISCV/newlib
make -j $(nproc)
```

1.6：qemu模拟器的构建(回归测试好像不需要呀，不过没关系可以写一个程序跑一下看看gcc工具都弄好了没有)

使用下载解压的源码中的qemu文件改名为qemu-project移动到RISCV文件夹下。

```
RISCV$            cd qemu-project
qemu-project$     mkdir build
qemu-project$     cd build
build$            ../configure --target-list=riscv64-softmmu,riscv64-linux-user --prefix=$RISCV/qemu
build$            ninja
build$            ninja install
```

在```$RISCV/qemu/bin```里面，可以看到```qemu-riscv64```，可以运行上述交叉编译器生成的程序。

## 例程1：hello world!

### hello.s

```	.file	"hello.c"
	.option nopic
	.attribute arch, "rv64i2p0_m2p0_a2p0_f2p0_d2p0_c2p0"
	.attribute unaligned_access, 0
	.attribute stack_align, 16
	.text
	.section	.rodata
	.align	3
.LC0:
	.string	"Hello, world!"
	.text
	.align	1
	.globl	main
	.type	main, @function
main:
	addi	sp,sp,-16
	sd	ra,8(sp)
	sd	s0,0(sp)
	addi	s0,sp,16
	lui	a5,%hi(.LC0)
	addi	a0,a5,%lo(.LC0)
	call	puts
	li	a5,0
	mv	a0,a5
	ld	ra,8(sp)
	ld	s0,0(sp)
	addi	sp,sp,16
	jr	ra
	.size	main, .-main
	.ident	"GCC: (GNU) 10.2.0"
```

### riscv64-unknown-elf-gcc编译

```$RISCV/newlib/bin/riscv64-unknown-elf-g++ hello.s -o hello```

### qemu运行

```$RISCV/newlib/bin/spike $RISCV/newlib/riscv64-unknown-elf/bin/pk hello```



恭喜，至此，已经成功运行了你的第一个RISC-V程序！环境搭建至此结束！



## 2. 运行测试用例测试

### 2.1：执行gcc回归测试

```
#运行gcc回归测试用例进行测试
ljw@ubuntu:~/RISCV/riscv-gnu-toolchain/build/newlib$ make report-gcc-newlib 2>&1|tee b_rv64_newlib-reportgccnewlib-20210720.log

```

### 2.2gcc回归测试预期结果

当命令窗口输出停止刷新，并出现`========= Summary of gcc testsuite =========`表示回归测试已经执行完成。

翻看输出的 b_rv64_newlib-reportgccnewlib-20210720.log文档，搜索Summary关键词，能够找到gcc Summary、 g++ Summary 、Summary of gcc testsuite 等信息。如下所示：

```		log
		=== gcc Summary ===
# of expected passes		113298
# of unexpected failures	30
# of unexpected successes	3
# of expected failures		605
# of unsupported tests		2533

------------------------------------------------------

		=== g++ Summary ===

# of expected passes		172985
# of unexpected failures	26
# of expected failures		706
# of unsupported tests		9570

```

### 2.3执行binutils回归测试
运行binutils回归测试用例进行测试
```
ljw@ubuntu:~/RISCV/riscv-gnu-toolchain/build/newlib$ make report-binutils-newlib 2>&1|tee b_rv64_newlib-reportbinutilsnewlib-20210720.log
```
### 2.4binutils回归测试预期结果
当命令窗口输出停止刷新，并出现========= Summary of binutils testsuite =========表示回归测试已经执行完成。

翻看输出的 b_rv64_newlib-reportbinutilsnewlib-20210701.log文档，搜索Summary关键词，能够找到binutils Summary、 gas Summary 、ld Summary 、Summary of gcc testsuite 等信息。如下所示：
```
		=== binutils Summary ===

# of expected passes		220
# of expected failures		1
# of unsupported tests		9

--------------------------------------------------------------------------
		=== gas Summary ===

# of expected passes		337
# of expected failures		15
# of unsupported tests		13
-----------------------------------------------------------------------
		=== ld Summary ===

# of expected passes		546
# of unexpected failures	2
# of expected failures		11
# of unsupported tests		208
--------------------------------------------------------------------

               ========= Summary of binutils testsuite =========
                            | # of unexpected case
                            |     binutils |           ld |          gas |
 rv64imafdc/  lp64d/ medlow |            0 |            0 |            0 |


```
#### 至此，我已经大致熟悉了测试的流程和命令。

3.测试结果分析与测试报告，找到上一次的测试结果，将两次的测试结果按照《gnu-regression-b_rv64_newlib-report.docx》要求填写测试结果，并对比两次的测试结果数据，填写测试结果。

最后附上一个我个人遇到的问题：![image](https://user-images.githubusercontent.com/55994081/126284150-44ff6d19-bee4-4046-802d-c680622727cb.png)

解决办法：![image](https://user-images.githubusercontent.com/55994081/126284275-9aa899ed-feb7-4e13-b169-69b81015b652.png)
##### 本文到此结束！

