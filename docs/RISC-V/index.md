# RISC-V

## 编译工具链

- 工具链仓库
```
# 仓库地址
https://github.com/riscv-collab/riscv-gnu-toolchain

git clone https://github.com/riscv-collab/riscv-gnu-toolchain.git
```

- 显式添加默认的平台支持并附加特定的平台
```
./configure --prefix=/opt/riscv \
  --enable-multilib \
  --with-multilib-generator="rv64gc-lp64d--;rv64imac-lp64--;rv32gc-ilp32d--;rv32imac-ilp32--;rv32ima-ilp32--"
```

- 验证支持的多库组合
```
riscv64-unknown-elf-gcc --print-multi-lib
```

## RISC-V主要内容

1. ISA命名规范
```
RV[###][abc...xyz]
```
    - RV用于标识RISC-V 。
    - 【###】: {32, 64, 128} 用于标识处理器的字宽（寄存器的宽度）。
    - 【abc...xyz】: 标识该处理器支持的指令集模块集合。

2. 模块化的ISA

    - RISC-V ISA = 1个基本整数指令集 + 多个可选的扩展指令集。

3. 通用寄存器

    - 定义了32个通用寄存器以及一个PC。
    - 寄存器宽度由ISA指定。
    - 每个寄存器具体编程时有特定的用途以及各自的别名。由RISC-V ABI定义。

4. Hart

    - HARdware Thread（相当于一个基本的CPU单元）

5. 特权级别

    - 机器模式（M Machine Mode）：最高权限，用于启动、配置硬件和异常处理。
    - 监督模式（S Supervisor Mode）：运行操作系统内核。
    - 用户模式（U User/Application Mode）：运行应用程序，权限受限。
        （可选H Mode：支持虚拟化扩展的超监模式。）

6. Control and Status Registers(CSR)
    
    - 不同的特权级别下时分别对应各自的一套Register。
    - 高级别的特权级别下可以访问低级别的CSR。
    - RISC-V 定义了专门用于操作CSR的指令。
    - RISC-V 定义了特定的指令可以用于在不同特权级别之间进行切换。

7. 内存管理和保护（物理内存保护、虚拟内存）

    - 物理内存保护（Physical Memory Protection, PMP）
    - 虚拟内存
        RISC-V 的虚拟内存（Virtual Memory）既依赖硬件支持，也依赖软件实现。其 ISA（指令集架构）提供了明确的硬件机制，包括特权模式、控制状态寄存器（CSR）和地址转换机制，而操作系统（如 Linux）则负责页表管理、缺页处理等软件层面的实现。

8. 异常和中断

    risc-v的大多数异常会中断返回后会再执行一次异常指令。而中断后本身返回后会执行下一条语句。

## 主机字节序

- 二进制码： 左高右底
- 内存布局： 上高下底
- 大端序： 高位二进制码存入底地址
- 小端序： 高位二进制码存入高地址
- risc-v使用小端序
- **注意：字节序是否反转以字节为单位，一个字节8位，8位内部不反转**

## risc-v 的6种指令格式

- R-type
- I-type
- S-type
- B-type
- U-type
- J-type

## 汇编相关命令

1. 编写汇编 `nvim test.s`
2. 编译 `[rgcc] -march=[rv32im] -mabi=ilp32 -c test.s -o test.o`
3. 链接 `[rld] -m elf32lriscv -o test.elf test.o`
4. 查看段头信息 `[rreadelf] -S test.elf`
5. 查看 .text 段原始内容 `[rreadelf] -x .text test.elf`
6. 反汇编所有段 `[robjdump] -D test.elf`
7. 反汇编代码段 `[robjdump] -d test.elf`
8. 展开所有伪指令 `[robjdump] -d -M no-aliases test.elf`

## 模拟器运行与调试

1. 模拟器运行程序。启动时暂停CPU等待GDB连接，默认1234端口。 `qemu-system-riscv32 -machine virt -nographic -kernel test.elf -S -s`

    - `-s 是 -gdb tcp:127.0.0.1:1234 的缩写`
    
2. 运行dgb `[rgdb] test.elf`
3. dgb连接远程1234端口 `target remote : 1234`

## 示例代码

- makefile
```makefile
CROSS_COMPILE = riscv64-unknown-elf-
CFLAGS = -nostdlib -fno-builtin -march=rv32im -mabi=ilp32 -g -Wall

CC = ${CROSS_COMPILE}gcc
OBJDUMP = ${CROSS_COMPILE}objdump

QEMU = qemu-system-riscv32
QFLAGS = -nographic -smp 1 -machine virt -bios none

GDBINIT = gdbinit
GDB = ${CROSS_COMPILE}gdb

EXEC = test
SRC = ${EXEC}.s

all:
	@${CC} ${CFLAGS} ${SRC} -Ttext=0x80000000 -o ${EXEC}.elf
	
run: all
	@${QEMU} ${QFLAGS} -kernel ./${EXEC}.elf

debug: all
	@${QEMU} ${QFLAGS} -kernel ${EXEC}.elf -s -S &
	@${GDB} ${EXEC}.elf -q -x ${GDBINIT}

clean:
	rm -rf *.o *.bin *.elf
```

- gdbinit
```bash
display/z $x5
display/z $x6
display/z $x7

set disassemble-next-line on
b _start
target remote : 1234
c
```