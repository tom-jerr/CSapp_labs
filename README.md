# bomb_lab2

## 一些基本操作

### 反汇编文件生成

文件中只有bomb.c和bomb可执行文件，所以需要查看汇编代码，需要生成反汇编文件；

```bash
$ objdump -d bomb > bomb.asm
```

### gdb查看寄存器值

```shell
x/s $esi
```

## phase_1

```C
400ee0:	48 83 ec 08          	sub    $0x8,%rsp	/*设置栈空间*/
400ee4:	be 00 24 40 00       	mov    $0x402400,%esi
400ee9:	e8 4a 04 00 00       	callq  401338 <strings_not_equal>
400eee:	85 c0                	test   %eax,%eax	 /*test指令同逻辑与and运算，但只设置条件码	寄存器，不改变目的寄存器的值，test %eax,%eax用于测试寄存器%eax是否为空，由于寄存器%rax一般存放函数的返回值，此处应该存放的是函数 strings_not_equal的值，而%eax是%rax的低32位表示，所以不难分析出，当%eax值为0时，test的两个操作数相同且都为0，条件码ZF置位为1，即可满足下一行代码的跳转指令*/
400ef0:	74 05                	je     400ef7 <phase_1+0x17>
400ef2:	e8 43 05 00 00       	callq  40143a <explode_bomb>
400ef7:	48 83 c4 08          	add    $0x8,%rsp	/*回收栈空间*/
400efb:	c3                   	retq   
```
### Input
Border relations with Canada have never been better.

------

## phase_2

```c
  400efc:	55                   	push   %rbp	//把数据压入栈中
  400efd:	53                   	push   %rbx
  400efe:	48 83 ec 28          	sub    $0x28,%rsp
  400f02:	48 89 e6             	mov    %rsp,%rsi
  400f05:	e8 52 05 00 00       	callq  40145c <read_six_numbers>	/*调用																	read_six_number程序，读取6个数字*/
  400f0a:	83 3c 24 01          	cmpl   $0x1,(%rsp)	//将第一个输入数字与立即数1比较
  400f0e:	74 20                	je     400f30 <phase_2+0x34>
  400f10:	e8 25 05 00 00       	callq  40143a <explode_bomb>
  400f15:	eb 19                	jmp    400f30 <phase_2+0x34>
  400f17:	8b 43 fc             	mov    -0x4(%rbx),%eax	//把此时的%rsp值传递给%eax（循环															 //开始）
  400f1a:	01 c0                	add    %eax,%eax	//%eax = %eax * 2
  400f1c:	39 03                	cmp    %eax,(%rbx)	//比较新的输入数据与%eax中的数据
  400f1e:	74 05                	je     400f25 <phase_2+0x29>
  400f20:	e8 15 05 00 00       	callq  40143a <explode_bomb>
  400f25:	48 83 c3 04          	add    $0x4,%rbx
  400f29:	48 39 eb             	cmp    %rbp,%rbx	//判断是否已经判定6个数字
  400f2c:	75 e9                	jne    400f17 <phase_2+0x1b>
  400f2e:	eb 0c                	jmp    400f3c <phase_2+0x40>
  400f30:	48 8d 5c 24 04       	lea    0x4(%rsp),%rbx	//当前指针加4
  400f35:	48 8d 6c 24 18       	lea    0x18(%rsp),%rbp	//当前指针加24
  400f3a:	eb db                	jmp    400f17 <phase_2+0x1b>
  400f3c:	48 83 c4 28          	add    $0x28,%rsp
  400f40:	5b                   	pop    %rbx
  400f41:	5d                   	pop    %rbp
  400f42:	c3                   	retq   
```
### Input
1 2 4 8 16 32