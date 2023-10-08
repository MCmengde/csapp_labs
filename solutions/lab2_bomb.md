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

## 备查

### 寄存器和汇编指令

[x64_cheatsheet](https://cs.brown.edu/courses/cs033/docs/guides/x64_cheatsheet.pdf)

### gdb相关指令

[gdb_refcard](https://web.stanford.edu/class/archive/cs/cs143/cs143.1128/documents/gdbref.pdf)

## bomb 1

第一个炸弹的判定函数是phase_1，其反汇编代码如下

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
