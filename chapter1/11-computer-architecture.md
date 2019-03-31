# Simple Computer Architecture {#simple-computer-architecture}

In Von Neumann architecture, memory and CPU are the two main components. We will use two struct CPU and MEMORY to simulate these two components.

```c
typedef struct computer{
	CPU cpu ;
	MEMORY memory;
}COMPUTER;
```

Our simple computer has 256 byte memory, which is simulated by a uint32 array.

```c
#define MAX_MEM_SIZE  64  //The max memory size - 64 words (256 bytes)

typedef struct memory{
	uint32_t addr[MAX_MEM_SIZE];
}MEMORY;
```

The CPU contains 3 control registers: Program Counter, Instruction Register and Process Status Register. We will learn more about the usage of control registers when we talk about how CPU repeat the Fetch-Decode-Execute cycle. It also contains 4 general purpose registers to store data and computation results. For example, when we try to do val1 + val2, the CPU will load load val1 and val2 from memory and store them into two general purpose registers then do the execution. The CPU may write the intermediate result of val1 + val2 into a third general purpose register and finally write the value in the register back to memory.

```c
typedef struct cpu{
	//Control registers
	uint32_t PC;	//Program counter
	uint32_t IR; 	//Instruction regiser
	uint32_t PSR;	//Processor Status Register

	//General purpose register
	int R[4];	// 4 Registers 
}CPU;
```
