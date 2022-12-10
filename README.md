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



------

## phase_3

```c
  400f43:	48 83 ec 18          	sub    $0x18,%rsp
  400f47:	48 8d 4c 24 0c       	lea    0xc(%rsp),%rcx	//存储第二个数
  400f4c:	48 8d 54 24 08       	lea    0x8(%rsp),%rdx	//存储第一个数
  400f51:	be cf 25 40 00       	mov    $0x4025cf,%esi	//存放输入数据（x/s 0x4025cf）
  400f56:	b8 00 00 00 00       	mov    $0x0,%eax
  400f5b:	e8 90 fc ff ff       	callq  400bf0 <__isoc99_sscanf@plt>
  400f60:	83 f8 01             	cmp    $0x1,%eax	//此时eax中存放scanf返回值，输入数据的														//个数
  400f63:	7f 05                	jg     400f6a <phase_3+0x27>	//大于1则跳转
  400f65:	e8 d0 04 00 00       	callq  40143a <explode_bomb>
  400f6a:	83 7c 24 08 07       	cmpl   $0x7,0x8(%rsp)	//判断num1是否大于7
  400f6f:	77 3c                	ja     400fad <phase_3+0x6a>	//大于则跳转执行																			//<explode_bomb>
  400f71:	8b 44 24 08          	mov    0x8(%rsp),%eax	//eax中存放第一个数
  400f75:	ff 24 c5 70 24 40 00 	jmpq   *0x402470(,%rax,8)	//switch判断(x/8xg 																		//0x402470查看case对应的内																//存)
  400f7c:	b8 cf 00 00 00       	mov    $0xcf,%eax	//case 0 
  400f81:	eb 3b                	jmp    400fbe <phase_3+0x7b>
  400f83:	b8 c3 02 00 00       	mov    $0x2c3,%eax	//case 2
  400f88:	eb 34                	jmp    400fbe <phase_3+0x7b>
  400f8a:	b8 00 01 00 00       	mov    $0x100,%eax	//case 3
  400f8f:	eb 2d                	jmp    400fbe <phase_3+0x7b>
  400f91:	b8 85 01 00 00       	mov    $0x185,%eax	//case 4
  400f96:	eb 26                	jmp    400fbe <phase_3+0x7b>
  400f98:	b8 ce 00 00 00       	mov    $0xce,%eax	//case 5
  400f9d:	eb 1f                	jmp    400fbe <phase_3+0x7b>
  400f9f:	b8 aa 02 00 00       	mov    $0x2aa,%eax	//case 6
  400fa4:	eb 18                	jmp    400fbe <phase_3+0x7b>
  400fa6:	b8 47 01 00 00       	mov    $0x147,%eax	//case 7
  400fab:	eb 11                	jmp    400fbe <phase_3+0x7b>
  400fad:	e8 88 04 00 00       	callq  40143a <explode_bomb>
  400fb2:	b8 00 00 00 00       	mov    $0x0,%eax
  400fb7:	eb 05                	jmp    400fbe <phase_3+0x7b>
  400fb9:	b8 37 01 00 00       	mov    $0x137,%eax	//case 1
  400fbe:	3b 44 24 0c          	cmp    0xc(%rsp),%eax	//判断num2与eax中的值是否相等
  400fc2:	74 05                	je     400fc9 <phase_3+0x86>
  400fc4:	e8 71 04 00 00       	callq  40143a <explode_bomb>
  400fc9:	48 83 c4 18          	add    $0x18,%rsp
  400fcd:	c3                   	retq   
```



x表明以十六进制的形式显示地址，g表示每8个字节的内存，因为这是x64平台，所以地址占8个字节

```shell
x/8xg 0x402470
```
### Input
**0 207	1 311	2 707	3 256	4 389	5 206	6 682	7 327**

