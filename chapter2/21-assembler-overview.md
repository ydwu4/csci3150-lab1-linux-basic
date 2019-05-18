# Overview {#overview}
We would like to write a program that can help us to automatically translate a human-readable **text file** like following into an **executable** filled with instructions that can be recognized by our simple computer. This program is called **assembler**.
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
Specifically, recall that our simple computer executes instruction that is represented by **uint32\_t** and decodes each instruction into 3 offset of type uint8\_t and 1 immediate value (possbile to be negtive), therefore, we need to **translate each line into an uint32\_t instruction**. For each line in the text file, we find what are the 4 parts and combine them into an uint32\_t instruction using bitwise operations (|, &, << and >>).

For example, in the above text file, **".word 1"** is translated to **0x00000001**. **"lw R1, R1, 1"** will be translated to **0x05010101** where lw corresponds to operation code **0x05**, R1 coresponds to **0x01** register and 1 is the immediate value. When simple computer starts running this program the offset will be set to **3** and simple computer will finish when program counter reaches **"halt"**. 
