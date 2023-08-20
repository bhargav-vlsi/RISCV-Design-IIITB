# ASIC Design


## RISCV based Myth

<details>
<summary>DAY-0</summary>

This section describes steps to install and configure RISCV tool chain

```
git clone https://github.com/kunalg123/riscv_workshop_collaterals.git
sudo apt install libboost-regex-dev
cd riscv_workshop_collaterals
chmod 755 run.sh
./run.sh
```

The above commands usually creates a folder called "riscv_toolchain" in home folder. Follow the next commands to access the tool chain from anywhere in terminal. Otherwise, path to bin folder of toolchain has to be provided to execute respective commands.

```
gedit .bashrc
```

At last line of .bashrc

```
export PATH=/home/<username>/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14/bin:$PATH
```

Save and close the .bashrc file. Then give following command to apply the changes of .bashrc file.

```
source .bashrc
```

![riscv_toolchain](./Images2/riscv_toolchain.png)

Riscv toolchain installed

</details>

<details>
<summary>DAY-1</summary>

### Overview
This section explains about development of applications on custom hardware architecture and RISCV toolchain.

### Introduction
RISC- Reduced Instruction Set Architecture Computer
We have hardware resources and software codes running on these resources. Compiler is a tool that convert high level code to assembly level code. Assembler is a tool that converts assembly level code to machine level code. The assembly level code is specific to type of architecture used. We will also look into several instructions and concepts such as psuedo instructions, integer RV64I, multiply extenstion RV64M, single & double precision floating point extension, application binary interface, memory allocation and stack pointer.


### GCC compiler and sample usage
Gcc compiler is used to convert C code into machine code for computer to execute. Here is sample commands to compile and execute sample c code.
```
gcc -o code.out code.c
./code.out
```

Following is sample c code for sum of 'n' numbers.
```
#include<stdio.h>
int main()
{
	int n,sum=0;
	printf("Enter n: ");
	scanf("%d",&n);
	for(int i=1;i<=n;i++) {
	sum=sum+i; }
	printf("Sum of %d numbers is %d\n",n,sum);
	return 1;
}
```

![sum1ton](./Images2/sum1ton.png)

### RISCV gcc compilation and assembly code
Folowing command describes the way to compile c code in riscv gcc compiler.
```
riscv64-unknown-elf-gcc -O1 -mabi=lp64 -march=rv64i -o <output>.o <inputfile>.c
```

Following represents way to observe object(compiled assembly code)
```
riscv64-unknown-elf-objdump -d <output>.o
```

If we want a bit optimized version of assembly code we use following option and use same command to observe object file.
```
riscv64-unknown-elf-gcc -Ofast -mabi=lp64 -march=rv64i -o <output>.o <inputfile>.c
```
### Execution of output file in RISCV tool chain
We use following command to execute object file using riscv tool chain.
```
spike pk <output>.o
```

![spike_pk_sum1ton](./Images2/spike_pk_sum1ton.png)

We use following command to debug the output
```
spike -d pk <output>.o
```
![spike_pk_debug](./Images2/spike_pk_debug.png)

We have few commands to execute and observe specific variables or registers during debugging session.
--To run code until a specific location
```
until pc 0 <memory_location>
```

--To observe contents of register in specific core
```
reg <core> <register>
```

Press "Enter" to execute line by line in assembly code.

Press "q" to to quit debugging session.

### Integer floating point representation
Human beings are accustomed to use decimal number system and computers are designed for binary number system. Hence, there is a requirement for conversion of decimal to binary system. Present day computers are designed to handle 64 bit numbers where we usually divide 64 bits into two 32 bits group, each 32 bit group is divided into four 8 bit group, each 8 bit group is divided into either 2 nibbles or simply considered doubleword.

The number of patterns for any 'n' bits is 2^(n).

Signed binary numbers are represented using 2's complement numbers. MSB of a binary number is 0 for positive number and 1 for negative number in any representation.

For unsigned numbers of n bit, range -> 0 to 2^(n)-1.
For signed numbers of n bit, range -> -(2^(n-1)) to (2^(n-1)-1).
Here is sample C code to understand floating representation and highest & lowest value possible in RISCV.

```
#include<stdio.h>
#include<math.h>

int main()
{
	unsigned long long int max=(unsigned long long int) (pow(2,64)-1);
	printf("Highest num represented by unsigned long long integer for 64 bit is %llu\n",max);
	
	max=(unsigned long long int) (pow(2,10)-1);
	printf("Highest num represented by unsigned long long integer for 10 bit is %llu\n",max);
	
	max=(unsigned long long int) (pow(2,127)-1);
	printf("Highest num represented by unsigned long long integer for 127 bit is %llu\n",max);
	
	unsigned long long int min=(unsigned long long int) (pow(2,64)*-1);
	printf("Lowest num represented by unsigned long long integer for 64 bit is %llu\n",min);
	
	long long int max2=(long long int) (pow(2,63)-1);// bug was here type long long int instead of just int in video
	printf("Highest num represented by signed long long integer for 64 bit is %lld\n",max2);
	
	long long int min2=(long long int) (pow(2,63)*-1);// bug was here type long long int instead of just int in video
	printf("Lowest num represented by signed long long integer for 64 bit is %lld\n",min2);
	return 1;
}
```

Following output represents the output for above code.

![unsigned_signed_output](./Images2/unsigned_signed_output.png)

</details>

<details>
<summary>DAY-2</summary>

### Application Binary Interface

Interface simply refers to appearance & functionality of a system without deeper understanding architecture & implementation.
Ex: Users need to know mainly about appearance of a building rather than construction of the same.

Ex: Programmer need to know strcuture or syntax of a application library rather than its internal implementation.

Application Binary interface uses registers to access hardware resources. 

RISC uses little endian architecture for storing data where most significant byte is in highest memory location.

RISC uses little endian architecture and stores 1 byte in each location. Instructions are 32 bit but data is 64 bit in 64 bit architeciture. It is a byte addressable memory. We specify -march=rv64i as architecture and hence set of integer base instructions.

We have 32 registers in RISCV architecture with x00, x01 and so on as representation.

![register_riscv](./Images2/register_riscv.png)

We have a sample for ABI system call. Following are two codes (C & ASM)

C code

```
#include <stdio.h>

extern int load(int x, int y); 

int main() {
	int result = 0;
       	int count = 3;
    	result = load(0x0, count+1);
    	printf("Sum of number from 1 to %d is %d\n", count, result); 
}
```

load.S

```
.section .text
.global load
.type load, @function

load:
	add 	a4, a0, zero //Initialize sum register a4 with 0x0
	add 	a2, a0, a1   // store count of 10 in register a2. Register a1 is loaded with 0xa (decimal 10) from main program
	add	a3, a0, zero // initialize intermediate sum register a3 by 0
loop:	add 	a4, a3, a4   // Incremental addition
	addi 	a3, a3, 1    // Increment intermediate register by 1	
	blt 	a3, a2, loop // If a3 is less than a2, branch to label named <loop>
	add	a0, a4, zero // Store final result to register a0 so that it can be read by main program
	ret
```

Following is output after compilation & execution of above codes.

![c_asm_output](./Images2/c_asm_output.png)


### Execution of C code on RISCV CPU Verilog

We have a RISCV design written in verilog. We convert our c code into hex code and simulate and execute it on RISCV CPU code and get back output on terminal. We have all necessary codes in labs folder.

![c_riscv_execution](./Images2/c_riscv_execution.png)

</details>

<details>
<summary>DAY-3</summary>

### Overview
This section describes about TL-verilog and Makerchip platform & its examples.

### Makerchip platform
Makerchip platform is a cloud based web application that takes design input in form of TL-verilog code and provides logical diagram & waveform as output without any testbench provided externally.

### Pythagorean example
We take pythagorean example from platform and execute it to understand its flow. We compile and observe the output on different windows as shown below.

![maker_chip_pythagorean](./Images2/maker_chip_pythagorean.png)

### Few other sample exercise in Maker chip platform (combinational circuits)
Here we consider simple logic gates as examples.

![maker_chip_gates](./Images2/maker_chip_gates.png)

Here we consider example of Full adder to understand use of vectors.

![maker_chip_adder](./Images2/maker_chip_adder.png)

Here we consider example of multiplexer.

![maker_chip_mux](./Images2/maker_chip_mux.png)

Here is example for calculator.

![maker_chip_calc1](./Images2/maker_chip_calc1.png)

### Sequential circuits in Maker chip platform

Here is example for free counter.

![maker_chip_counter](./Images2/maker_chip_counter.png)

Here is sequential calculator that remembers previous results for next calculation.

![maker_chip_calc2](./Images2/maker_chip_calc2.png)

### Pipeline 
Pipeline refers to executing in stages. We can divide entire calculation into several stages to have clock frequency unchanged. This is done in TL-verilog by putting @1, @2 before that stage instruction. This is seen in pythagorean example as shown before. 

![maker_chip_pipeline](./Images2/maker_chip_pipeline.png)

### Identifiers & tokens
Identifiers are elements used to refer some location or value or signal. There are some rules to be followed to name an identifier. 

-- They must start with two lower case letters. 
lower_case -> pipe signal

CamelCase -> state signal

UPPER_KEY -> keyword signal

-- They must not end with numbers.

-- >> refers to ahead of.

Here is sample pipeline given as lab exercise.

![maker_chip_error](./Images2/maker_chip_error.png)

### Calculator & counter pipeline lab

![maker_chip_calc_counter](./Images2/maker_chip_calc_counter.png)

### Cycle calculator

![maker_chip_cycle_calc](./Images2/maker_chip_cycle_calc.png)

### Validity
Validity is a feature in TL-verilog not available in other RTL languages. It is easier to debug, better error checking and automated clock gatig in TL-verilog. If we have multiple stages of computation, then we use validity to indicate the valid output available to consider it for next set of computations.

### Clock gating
In most of circuits, clock consumes more power as it is generated continuously from clock generator circuit. Clock gating avoids toggling of clock signals. We use condition of validity to perform clock gating.

### Distance calculator
Here we use pythagoras theorem to calculate distance between two points through a given path as shown.

![maker_chip_distance_calc](./Images2/maker_chip_distance_calc.png)

### Calculator with single value memory Lab
Here we modify and design previous calculator to have single value memory and recall use for successive computations.

![maker_chip_single_mem_calc](./Images2/maker_chip_single_mem_calc.png)

</details>

<details>
<summary>DAY-4</summary>

### Overview
This section describes about micr-architecture of single cycle RISCV CPU and its various stages which is main aspect of workshop.

### Simple RISCV Micro-architecture

Here,we implement the following basic blocks:

Program Counter (PC)
Imem-Rd ( Instruction Memory)
Instruction Decoder
Register File Read
Arithmatic Logic Unit (ALU)
Register File Write
Branch

### Next PC lab

```
$pc[31:0] = $reset ? 32'b0 : >>1$pc + 32'd4;
```
![maker_chip_next_pc](./Images2/maker_chip_next_pc.png)






</details>

<details>
<summary>REFERENCES</summary>

    
https://steveicarus.github.io/iverilog/

https://github.com/kunalg123/

https://github.com/stevehoover/RISC-V_MYTH_Workshop

</details>
