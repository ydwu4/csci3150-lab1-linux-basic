# CPU Cycle {#cpu-cycle}

Our simple computer starts executing a program by first load the program into memory and initialize all the cpu registers.

```c
int computer_load_init(COMPUTER * cp, char * file)
{
	//load the image file
	int fd;
	int ret;

	// open the file 
	if ( ( fd = open( file, O_RDONLY ) ) < 0 ){
		printf("Error: open().\n");
		exit(-1);
	}

	// load the program (the program file <= 256 bytes) into the memory
	if( ( ret = read( fd, &cp->memory, MAX_MEM_SIZE*4)) < 0 ){
		printf("Error: read().\n");
		exit(-1);
	}else if ( ret > (MAX_MEM_SIZE*4) ) {
		printf("Error: read() - Program is too big. \n");
		exit(-1);
	}  

	//Initialize control registers
	cp->cpu.PC=0;	//Program counter
	cp->cpu.IR=0; 	//Instruction regiser
	cp->cpu.PSR=0;	//Processor Status Register

	//General purpose register
	cp->cpu.R[0]=0;	// General register No. 0 
	cp->cpu.R[1]=0;	// General register No. 1 
	cp->cpu.R[2]=0;	// General register No. 2 
	cp->cpu.R[3]=0;	// General register No. 3

	return 0;
}
```

Then we can starts to execute the instructions stored in memory by repeating the Fetch-Decode-Execute cycle. The argument computer passed to the cpu\_cycle() function is the computer structure that we initialize just now.

```c
// Execute CPU cyles: fetch, decode, execution, and increment PC; Repeat
while(1) {
	if( cpu_cycle(&computer) < 0 )
		break;
} 

int cpu_cycle(COMPUTER * cp) {
	uint8_t opcode, sreg, treg; // operation code, source register, target register
	int8_t  imm; // immediate data

	if(fetch(cp) < 0)
		return -1;

	if(decode(cp->cpu.IR, &opcode, &sreg, &treg, &imm) < 0)
		return -1;

	if(execute(cp, &opcode, &sreg, &treg, &imm) < 0 )
		return -1;

	return 0;
}
```

Fetch function takes a pointer to computer as argument. The central step is to load the instruction(a unsigned 32-bit integer) stored in memory address indexed by the cpu program counter(memory.addr[pc]) into the intruction register of CPU(cpu.IR).

```c
int fetch(COMPUTER * cp)
{
	uint32_t pc = cp->cpu.PC;

	if (pc < 0 || pc > 63 ){
		printf("PC is not in 0-63.\n");
		return -1;
	}else{
		// load instuction into CPU register
		cp->cpu.IR = cp->memory.addr[pc];
		return 0;
	}
}
```

After we load the instruction into Intruction Register, we can decode the instruction. Decode takes more arguments. The first argument instr is the instruction itself stored in Instruction Register, which is 32-bit unsigned integer. Then the following 4 8-bit integer values is the results that this decode funtion will try to fill in. 

Basically the decode process involves segmenting the 32-bit intruction two 4 8-bit values and assign the 8-bit value to the decoded results by using "bitwise and" and "bitwise shift". Note that the immediate data can be negative(Recall 2' complement encoding).

For example, an instruction (0x02aabbcc) will be decoded into: 0x02(opcode), 0xaa(sreg), 0xbb(treg), 0xcc(imm).
```c
// p_opcode: intruction code
// p_sreg: source register index
// p_treg: target register index
// p_imm: immediate data
int decode(uint32_t instr, uint8_t * p_opcode, uint8_t * p_sreg, uint8_t * p_treg, int8_t * p_imm)
{
	*p_opcode = (instr & 0xFF000000) >> 24;
	if (*p_opcode > MAX_INSTRUCTION_CODE) {
		printf("Invalid instruction\n");
		return -1;
	}

	*p_sreg = (instr & 0x00FF0000) >> 16;
	*p_treg = (instr & 0x0000FF00) >> 8;
	*p_imm = instr & 0x000000FF;

	return 0;
}
```

The execute method takes computer as first argument and the decoded results as the remaining arguments. Basically, the computer tries to match the operation code first and then perform corrsponding modifications on the source register, target register and intermidiate data.

We take addi as an example. addi is the assmply representation of this instuction. This intruction takes three arguments x, y and z. the value of x will be decoded into p\_sreg which represents which general purpose register will be used. Simliar for y. z is decoded into p\_imm. Where it will be used as an value to be added. The whole assembly instruction "addi 0,1,2" which is encoded to an 32-bit ingeter "0x02000102" means "add the value stored in register 0 with the immediate value 2 together then store the value to register 1". After finish this instuction, we add the program counter by 1. So that the fetch function will try to fetch the next instruction from memory.

Try the small execersize of writing the logic for blez instruction.
```c
int execute(COMPUTER *cp, uint8_t * p_opcode, uint8_t * p_sreg, uint8_t * p_treg, int8_t * p_imm)
{	
	switch( * p_opcode){
                case halt:
                        // halt (0x00000000) Effect: Stop CPU and exit
                        return -1;
                case NOP:
                        // NOP  (0x01000000) Effect: PC <- PC+1
                        cp->cpu.PC +=1;
                        break;
                case addi:
                        //addi  (0x02xxyyzz) Effect: R[yy] <- R[xx] + zz; PC <- PC+1
                        cp->cpu.R[*p_treg] = cp->cpu.R[*p_sreg] + *p_imm;
                        cp->cpu.PC +=1;
                        break;
                case move_reg:
                        //move_reg (0x03xxyy00) Effect: R[yy] <- R[xx]; PC <- PC+1
                        cp->cpu.R[*p_treg] = cp->cpu.R[*p_sreg];
                        cp->cpu.PC +=1;
                        break;
                case movei:
                        //movei (0x0400yyzz) Effect: R[yy] <- zz; PC <- PC+1
                        cp->cpu.R[*p_treg] = *p_imm;
                        cp->cpu.PC +=1;
                        break;
                case lw:
                        //lw-load word: (0x05xxyyzz) Effect: R[yy] <- M{ R[xx] + zz}; PC <- PC+1
                        cp->cpu.R[*p_treg] = cp->memory.addr[*p_imm + cp->cpu.R[*p_sreg];
                        cp->cpu.PC +=1;
                        break;
                case sw:
                        //sw-store word: (0x06xxyyzz) Effect: M{R[xx]+zz} <- R[yy]; PC <- PC+1 
                        cp->memory.addr[*p_imm + cp->cpu.R[*p_sreg] = cp->cpu.R[*p_treg];
                        break;
                case blez:
                        //blez - Branch on less than or equal: (0x07xx00zz)
                        //Effect: if R[xx] == 0 PC <- PC + 1 + zz
                        //        else PC <- PC +1

                        /*your code*/

                        break;
                default:
                        printf("Illegal instruction\n");
                        return -1;

	}
	return 0;
}
```

Put all things together, we finishes a simple Von Neumann computer simulator. The full code is down below, you can compile it and run it against a file that is written in the machine code (32-bit unsigned integers for this simulator) the code can be found at https://github.com/ydwu4/simple-computer-simulator.git .
