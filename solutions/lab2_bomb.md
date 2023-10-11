# Bomb lab

本实验主要涉及原书第三章知识点，实验为一个拆炸弹的游戏，共有六个炸弹

## 准备

1. 使用工具将可执行文件反汇编，方便查看

```shell
objdump -d bomb > bomb.s
```

2. 使用gdb调试

```shell
gdb bomb
```

进入gdb之后，设置参数，也就是答案的文本文件

```shell
(gdb) set args rst.txt
```

然后就可以使用`run`命令跑起来了，同时设置显示汇编代码和寄存器的窗口，方便查看，使用`ctrl+x o`快捷键可以切换窗口聚焦

```shell
(gdb) layout asm
(gdb) layout regs
```

## Cheat Sheet

### 寄存器和汇编指令

[x64_cheatsheet](https://cs.brown.edu/courses/cs033/docs/guides/x64_cheatsheet.pdf)

### gdb相关指令

[gdb_refcard](https://web.stanford.edu/class/archive/cs/cs143/cs143.1128/documents/gdbref.pdf)

## bomb 1

第一个炸弹的判定函数是phase_1，其汇编代码如下

```assembly
0000000000400ee0 <phase_1>:
  ; 压栈
  400ee0:	48 83 ec 08          	sub    $0x8,%rsp
  ; 设置第二个参数，第二个参数用rsi寄存器传递，
  ; 第一个参数在rdi中，也就是phase_1函数的入参，即读入的字符串的地址
  400ee4:	be 00 24 40 00       	mov    $0x402400,%esi
  ; 调用strings_not_equal，比较读入的字符串与$0x402400处的字符串
  400ee9:	e8 4a 04 00 00       	callq  401338 <strings_not_equal>
  ; 判断返回值是否为0，返回值用rax传递
  400eee:	85 c0                	test   %eax,%eax
  ; 等于0则跳过explode_bomb，否则调用explode_bomb，程序结束
  400ef0:	74 05                	je     400ef7 <phase_1+0x17>
  400ef2:	e8 43 05 00 00       	callq  40143a <explode_bomb>
  ; 出栈
  400ef7:	48 83 c4 08          	add    $0x8,%rsp
  400efb:	c3                   	retq
```

根据以上分析，显然答案就在`$0x402400`处，直接查看可得

```shell
(gdb) x/s 0x402400
$0x402400:      "Border relations with Canada have never been better."
```

## bomb 2

phase_2的汇编代码解析如下：

```assembly
0000000000400efc <phase_2>:
  ; %rbp和%rbx是callee函数负责保存的寄存器，如果在函数中用到他们，先压栈
  400efc:	55                   	push   %rbp
  400efd:	53                   	push   %rbx
  400efe:	48 83 ec 28          	sub    $0x28,%rsp

  ; 调用read_six_numbers方法，从输入的字符串中依次读取6个整数到栈中
  ; read_six_numbers调用时，第一个参数在rdi中，即phase_2的入参，也是就输入字符串的首地址
  ; 第二个参数在rsi中，即为栈顶地址
  400f02:	48 89 e6             	mov    %rsp,%rsi
  400f05:	e8 52 05 00 00       	callq  40145c <read_six_numbers>

  ; 将栈顶的整数与1进行比较，即输入的第一个数，不等则爆炸；可得答案中第一个数是1
  400f0a:	83 3c 24 01          	cmpl   $0x1,(%rsp)
  400f0e:	74 20                	je     400f30 <phase_2+0x34>
  400f10:	e8 25 05 00 00       	callq  40143a <explode_bomb>

  400f15:	eb 19                	jmp    400f30 <phase_2+0x34>
  ; 循环开始
  ; 将 %rbx-4 地址处的一个32位整数放到rax中，即输入的上一个数(第一个循环时为读入的第一个数)
  400f17:	8b 43 fc             	mov    -0x4(%rbx),%eax
  ; 将上一个数*2
  400f1a:	01 c0                	add    %eax,%eax
  ; 比较当前的数与上一个数*2的结果
  400f1c:	39 03                	cmp    %eax,(%rbx)
  ; 不等则爆炸
  400f1e:	74 05                	je     400f25 <phase_2+0x29>
  400f20:	e8 15 05 00 00       	callq  40143a <explode_bomb>
  ; 相等则继续循环，并检查是否到达边界，到达则跳出循环
  400f25:	48 83 c3 04          	add    $0x4,%rbx
  400f29:	48 39 eb             	cmp    %rbp,%rbx
  400f2c:	75 e9                	jne    400f17 <phase_2+0x1b>
  400f2e:	eb 0c                	jmp    400f3c <phase_2+0x40>

  ; 设置循环初始状态
  ; rbx作为index指针，保存遍历的索引地址；rbp作为遍历的边界
  ; rbx指向rsp+4的位置，即读入的第二个数
  400f30:	48 8d 5c 24 04       	lea    0x4(%rsp),%rbx
  400f35:	48 8d 6c 24 18       	lea    0x18(%rsp),%rbp
  400f3a:	eb db                	jmp    400f17 <phase_2+0x1b>
  ; 循环结束

  400f3c:	48 83 c4 28          	add    $0x28,%rsp
  400f40:	5b                   	pop    %rbx
  400f41:	5d                   	pop    %rbp
  400f42:	c3                   	retq
```

由分析可知，答案为6个整数，第一个整数是1，后面依次位前一个的两倍，所以答案如下

```text
1 2 4 8 16 32
```

关于read_six_number函数的具体内容，可以参考[bomb.s](../lab2_bomb/bomb.s)中的注释

在gdb调试时，可以使用以下命令查看栈中的输入整数：
```shell
x/6wd $rsp
0x7fffffffdcb0: 1   2   4   8
0x7fffffffdcc0: 16  32
```