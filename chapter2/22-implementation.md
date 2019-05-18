# Implementation {#implementation}

In C programming language courses, we've learned how to read from text file. We can make use of **"fscanf"** to read text of specific format into variables **opcode, sreg, dreg and immoff**. The line after ";" will be treated as comment and will not be translated into any instruction. We will omit this step.

After we get 4 parts of one instruction, we now need to combine them into one final uint32\_t instruction by the following approach:
```c
uint32_t inst_code = (opcode << 24) | (sreg << 16) | (dreg << 8) | (immoff & 0x000000ff);
```

We use opcode=5, sreg=1, dreg=1 and immoff=-2 as an example to illustrate. The binary representation of **opcode** is of 8 bits:**00000101**, **sreg** and **dreg** is **00000001**. For **immoff**, we following the 2' complement representation therefore it is **11111110**. 

The result of (opcode << 24) | (sreg << 16) | (dreg << 8) is 0x05010100 as expected. But we **cannot** directly do 0x05010100 | immoff directly, the reason is that for the "|" operation, the data type on its left is integer (because 24, 16, 8 are integers by default which makes result of "<<" an integer). If we directly "|" them together, **immoff will be upcast into an integer first whose representaion will be 0xfffffffe**. When we "|" 0xfffffffe with 0x05010100, the result will be 0xfffffffe. To avoid this senerio, we do **(immoff & 0x000000ff)** to only retrieve the last 8 bits of immoff. Then we can get the expected result: 0x050101fe.


After get inst\_code we can use: 
```c
fwrite(&inst_code, sizeof(inst_code),1, fout);
```
to write this instruction to file fout.

Excersize: write an assembler to translate the following text to an executable for our simple computer. You can refer to the specifications of instructions listed in the simple computer program. And use the simple computer to execute generated executable to find out the final state of all registers and word.
```c
.word 1
.word 100
lw R0, R0, 0      ; load word 0 to R0
lw R1, R1, 1      ; load word 1 to R1
addi R0, R0, 1    ; R0+=1
addi R1, R1, -1   ; R1-=1
addi R0, R2, -100 ; R2=R0-100
blez R2, -4       ; if R2 <=0 pc += (1+ -4) else pc += 1
sw R3, R0, 0      ; save R0 to word 0
halt              ;
```
