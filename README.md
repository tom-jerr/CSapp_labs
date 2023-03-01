## Arch Lab

## Part A

### sum_list()

#### 指令

- 在PartA目录下sum.ys中

#### 结果

~~~bash
(base) lzy@DESKTOP-PS05SI3:~/csapp/archlab/archlab-handout/sim/misc$ ./yis ../PartA/sum.yo
Stopped in 30 steps at PC = 0x13.  Status 'HLT', CC Z=1 S=0 O=0
Changes to registers:
%rax:   0x0000000000000000      0x0000000000000cba
%rsp:   0x0000000000000000      0x0000000000000200

Changes to memory:
0x01f0: 0x0000000000000000      0x000000000000005b
0x01f8: 0x0000000000000000      0x0000000000000013
~~~

***

### rsum_list

#### 指令

- 在PartA目录下rsum.ys中

#### 结果

~~~bash
(base) lzy@DESKTOP-PS05SI3:~/csapp/archlab/archlab-handout/sim/misc$ ./yis ../PartA/rsum.yo
Stopped in 44 steps at PC = 0x13.  Status 'HLT', CC Z=0 S=0 O=0
Changes to registers:
%rax:   0x0000000000000000      0x0000000000000cba
%rsp:   0x0000000000000000      0x0000000000000200

Changes to memory:
0x01b8: 0x0000000000000000      0x0000000000000c00
0x01c0: 0x0000000000000000      0x0000000000000093
0x01c8: 0x0000000000000000      0x00000000000000b0
0x01d0: 0x0000000000000000      0x0000000000000093
0x01d8: 0x0000000000000000      0x000000000000000a
0x01e0: 0x0000000000000000      0x0000000000000093
0x01f0: 0x0000000000000000      0x000000000000005b
0x01f8: 0x0000000000000000      0x0000000000000013
~~~

***

### copy()

#### 指令

- 在PartA目录下copy.ys中

#### 结果

~~~bash
(base) lzy@DESKTOP-PS05SI3:~/csapp/archlab/archlab-handout/sim/misc$ ./yis ../PartA/copy.yo
Stopped in 44 steps at PC = 0x13.  Status 'HLT', CC Z=0 S=1 O=0
Changes to registers:
%rax:   0x0000000000000000      0x0000000000000cba
%rcx:   0x0000000000000000      0xffffffffffffffff
%rsp:   0x0000000000000000      0x0000000000000200
%rsi:   0x0000000000000000      0x0000000000000048
%rdi:   0x0000000000000000      0x0000000000000030

Changes to memory:
0x0030: 0x0000000000000111      0x000000000000000a
0x0038: 0x0000000000000222      0x00000000000000b0
0x0040: 0x0000000000000333      0x0000000000000c00
0x01f0: 0x0000000000000000      0x000000000000006f
0x01f8: 0x0000000000000000      0x0000000000000013
~~~

***

## PartB

- 执行`iaddq`指令的的逻辑实现

### 操作

~~~verilog
#有效指令
bool instr_valid = icode in 
	{ INOP, IHALT, IRRMOVQ, IIRMOVQ, IRMMOVQ, IMRMOVQ,
	       IOPQ, IJXX, ICALL, IRET, IPUSHQ, IPOPQ, IIADDQ };
 
#需要寄存器
bool need_regids =
	icode in { IRRMOVQ, IOPQ, IPUSHQ, IPOPQ, 
		     IIRMOVQ, IRMMOVQ, IMRMOVQ, IIADDQ };
 
#有立即数valC
bool need_valC =
	icode in { IIRMOVQ, IRMMOVQ, IMRMOVQ, IJXX, ICALL, IIADDQ };
 
#srcB是寄存器值rB
word srcB = [
	icode in { IOPQ, IRMMOVQ, IMRMOVQ, IIADDQ  } : rB;
	icode in { IPUSHQ, IPOPQ, ICALL, IRET } : RRSP;
	1 : RNONE;  # Don't need register
];
#寄存器的写入位置为rB
word dstE = [
	icode in { IRRMOVQ } && Cnd : rB;
	icode in { IIRMOVQ, IOPQ, IIADDQ} : rB;
	icode in { IPUSHQ, IPOPQ, ICALL, IRET } : RRSP;
	1 : RNONE;  # Don't write any register
];
 
#ALU的计算符号位Add
word aluA = [
	icode in { IRRMOVQ, IOPQ } : valA;
	icode in { IIRMOVQ, IRMMOVQ, IMRMOVQ, IIADDQ } : valC;
	icode in { ICALL, IPUSHQ } : -8;
	icode in { IRET, IPOPQ } : 8;
	# Other instructions don't need ALU
];
#ALU的B值位valB，即为寄存器rB
word aluB = [
	icode in { IRMMOVQ, IMRMOVQ, IOPQ, ICALL, 
		      IPUSHQ, IRET, IPOPQ, IIADDQ } : valB;
	icode in { IRRMOVQ, IIRMOVQ } : 0;
	# Other instructions don't need ALU
];
 
#计算后需要设置条件码
bool set_cc = icode in { IOPQ, IIADDQ };
~~~

### 结果

~~~bash
32 instructions executed
Status = HLT
Condition Codes: Z=1 S=0 O=0
Changed Register State:
%rax:   0x0000000000000000      0x0000abcdabcdabcd
%rsp:   0x0000000000000000      0x0000000000000100
%rdi:   0x0000000000000000      0x0000000000000038
%r10:   0x0000000000000000      0x0000a000a000a000
Changed Memory State:
0x00f0: 0x0000000000000000      0x0000000000000055
0x00f8: 0x0000000000000000      0x0000000000000013
ISA Check Succeeds
~~~

***

## Part C

### 指令参考

- 同PartB相同设置iaddq指令

- 修改ncopy.ys文件，实现指令流水线

  ~~~ assembly
  ##################################################################
  # Do not modify this portion
  # Function prologue.
  # %rdi = src, %rsi = dst, %rdx = len
  ncopy:
  
  ##################################################################
  # You can modify this portion
  	# Loop header
  iaddq $-4, %rdx		# calculate len - 4
  jg Loop                 # if len >= 4 goto Loop
  
  RESTT:
  	iaddq $4, %rdx          # restore the old value of len
  REST:
  	jg NOT_FINISHED         # if %rdx is greater than 0,then not finished yet
  	ret
  NOT_FINISHED:
  	mrmovq (%rdi), %r10		# load *src
  	iaddq $8, %rdi
  	andq %r10, %r10			# test *src
  	jle ADD3				# if not greater than 0, jump
  	iaddq $1, %rax			# add the count of postive numbers
  ADD3:	
  	rmmovq %r10, (%rsi)		# save
  	iaddq $8, %rsi
  	iaddq $-1, %rdx
  	jmp REST
  Loop:
  	mrmovq (%rdi), %r10		# read val from src...
  	iaddq $40, %rdi			# src+=5
  	rmmovq %r10, (%rsi) 	# save *src to *rsi
  	andq %r10, %r10			# test %r10
  	jle ADD1				# if This number is not greater than 0, than not add %rax
  	iaddq $1, %rax			# add the number of postive numbers
  ADD1:	
  	mrmovq -32(%rdi), %r10	# read the second number
  	iaddq $40, %rsi			# dst += 5
  	rmmovq %r10, -32(%rsi)	# save *(src + 1) to *(dst + 1)
  	andq %r10, %r10			# val <= 0?
  	mrmovq -24(%rdi), %r10   # rearrange the instructions to avoid load/use hazzard
  	jle ADD2				# if not, add the count
  	iaddq $1, %rax			# count++
  ADD2:
  	rmmovq %r10, -24(%rsi)
  	andq %r10, %r10
  	mrmovq -16(%rdi), %r10
  	jle ADD4
  	iaddq $1, %rax
  ADD4:
  	rmmovq %r10, -16(%rsi)
  	andq %r10, %r10
  	mrmovq -8(%rdi), %r10
  	jle ADD5
  	iaddq $1, %rax
  ADD5:
  	rmmovq %r10, -8(%rsi)
  	andq %r10, %r10
  	jle ADD6
  	iaddq $1, %rax
  ADD6:
  	iaddq $-5, %rdx			# len -= 5 if(len - 4 > 0) that means len > 4
  	jg Loop					# if so, goto Loop:
  
  	jmp RESTT
  ##################################################################
  # Do not modify the following section of code
  # Function epilogue.
  Done:
  	ret
  ##################################################################
  # Keep the following label at the end of your function
  End:
  #/* $end ncopy-ys */
  ~~~

### Result

- 借鉴作者[代码](https://blog.csdn.net/kangyupl/article/details/106976780?ops_request_misc=&request_id=&biz_id=102&utm_term=archlab&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-5-106976780.142^v73^control,201^v4^add_ask,239^v2^insert_chatgpt&spm=1018.2226.3001.4187)，未测试。

