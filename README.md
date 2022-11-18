# RISCV-32I-AHD
Advanced Hardware Design Project


- All design files inside Final_project --> Final_project.srcs --> sources_1
- All testbench files inside Final_project --> Final_project.srcs --> sim_1
- All simulation waveforms inside Final_project --> Simulation_output
- All elaborated designs inside Final_project --> Elaborated_Design

## ALU
1. Design (alu.v)
- We designed an ALU which performs all the operations mentioned in the supplementary material. The ALU has 2 32 bit inputs, a 4 bit alu_select input and a 32 bit output
that stores the result of the operarion. The alu_select opcodes have been defined in a sepearte header file that hold all the opcodes for different components of the 
processor. For, example ADD instruction has the word ADD at the alu_select case instead of the actual binary number. This makes the code very legible and easy to 
understand. 

2. Testbench (alu_tb.v, alu_test.txt)
- The testbench for the ALU is pretty straightforward. It has registers that store the inputs and wire that holds the output. I chose to read a alu_test.txt file that
will consist of all my testcases. This way instead of writing each and every testcases, there will be a single file which will hold the testcases. I also added 2 counters, a pass count and a fail count. These counters will keep track of all the testcases whether they have passed or failed. 


## Program Counter
1. Design (program_counter.v)
- Program Counter is the register that stores and sends out address of instruction to execute. It sends out 32 bit address, however last two LSBs are tied to 0, so that only starting address of instruction word is generated. We have also implemented some additional functionality in the PC module itself, first, increment by 4 and second, branch/jump to an immediate value. The output is decided between the two by input imm, generated by control unit. A failsafe mechanism is built in to reset PC to 0x01000000 in case it reaches an out-of-bounds address not supported by IM.

2. Testbench (program_counter_tb.v, program_counter.csv)
- We are using file IO to drive inputs and compare outputs with a csv file containing all the testcases. The testcases cover the following scenarios:
	- Tests regular counter operation (increment by 4)
	- Tests immediate operation (update value as per immediate input)
	- Tests return to regular counter post immediate address update
	- Tests if out-of-bounds address leads to reset condition
		- If immediate address is out of bounds
		- If PC counts to last address in IM
	- Test external reset functionality


## Instruction Memory
1. Design (instruction_mem.v, imem.mem)
- Instruction memory has been implemented as a 2Kbyte instruction memory designed to store 512x4-byte long instruction words. The memory addresses start from 0x01000000 and goes to 0x010007FF with one byte of data stored at each address. Hence, one instruction word spans 4 addresses in little endian format. Only a complete instruction word can be read from the memory. 

2. Testbench (instruction_mem_tb.v, imem_testcases.csv)
- Design loads imem.mem file to memory. Testcases are written to randomly access all addresses and stored instruction value is compared. Hence all addresses in memory are tested.


## Register File
1. Design (register_file.v)
- Register file is a collection of 32 32-bit wide registers, where R0 is hardwired to 0. Reset signal is present to clear registers R1 to R31. Read and write functionality present (except for R0, which is read-only).

2. Testbench (register_file_tb.v, register_file.csv)
- Testcases are stored in a csv file and cover the following scenarios:
	- Tests register write operation when we signal is high
	- Tests if junk write is prevented when we signal is low
	- Tests rs1 and rs2 read
	- Tests read on R0 (should be 0 even without reset, hence first testcase)
	- Tests if write on R0 is prevented even if we signal is high
  - Tests reset functionality and read/write post reset


## Data Memory

1. Design (data_memory.v) 
he data_mem.v module contains the data memory for the processor. Data memory is 4 KB size. It begins at the address 0x80000000. The load from and store to the memory takes place in this module. It is tested by DMem_TB.v, the tests include reading the N-numbers of team members at special read-only locations address 0x00100000, 0x00100004, 0x00100008 in the data memory and outputs them one by one.

2. Testbench (data_memmory_tb.v):
When the read signal is low and reset (active low) is high, data is written into data memory at specified address byte-wise and the writing of byte-wise data is controlled by 4-bit write enable. The read is set, and stored data is read and verified. At locations 0x00100000, 0x00100004 and 0x00100008, when read signal is high and write enable, ‘we’ is 4’b0, the N- numbers of our team members is output in consecutive clock cycles.

## Data Extender 

1. Design (data_extender.v)
The data_ext.v depending on the type of load instruction being executed sign extends the incoming data bits of given length to 32-bits, using the MSB value of the incoming data bits. It is implemented with the help of a combinational MUX logic. Testing is done through data_ext_tb.v.

2. Testbench (data_extender_tb.v) 

In the testbench, the data memory is tested for diverse types of load instructions and output is verified against expected result values. The test bench is automatically able to throw errors and generate the test bench passsed successfully message when completed.

## Branch Comparator 

1. Design (branch_comparator.v)
The BranComp.v module has the logic to check if the branch will be taken for the given B- type instructions or Branch instructions. It is tested by BranComp_TB.v.

2. Testbench (branch_comparator_tb.v)
The testbench BranComp_TB.v verifies for all the branch operations supported by the processor whether the signed or unsigned branch is occurring properly. BranComp_TB.v takes 390 ns total run time. The test bench is automatically able to throw errors and generate the test bench passsed successfully message when completed.

## Control Unit

1. Design (Control_unit_FSM)
The control_unit_FSM.v module contains the Control unit of the processor. It generates the control signals for coordinating all the modules according to the instruction being executed and takes care of the multicycle state machine for current execution cycle (Instruction Fetch, Instruction decode and execution, Memory, Write back and PC update, and Halt). he behavior of the system is governed by an FSM that lists 5 states: fetch-access the memory to get the next instruction (activate the read signal, place the right memory address on address bus, read the content of memory pointed by the address), decoding (interpret the data bits of instruction word to identify the operation and its data which can be taken from memory or registers), execution ( perform specific operation using processor resources and write the result into memory and/or registers). Our Processor implements a Mealy machine where current output is dependent on current input and current state. The FSM states 

![FSM](https://github.com/naman-47/RISCV-32I-AHD/blob/main/FSM%20(Control%20Unit).drawio.png)

2. Testbench (control_unit_FSM_tb.v)
The control unit testbench comprises test cases for all the instructions. It checks if the proper control signals for the current instruction being executed are generated for all the instructions. The testbench checks for all the instrcutions if the resulting states are as expected, verified through the control signals generated in those states.

## Immediate Extender

1. Design (Immediate_extender.v)
The module extends the immediate depending on the type of instruction being executed (J-type, I-type, S-type, B-type, U-type) based on the MSB value of the immediate. It then will feed the generated output into the ALU after begin selected by the MUX if the instruction is of I-Type.

2. Testbench (Immediate_extender.v)
immediate_extender_tb.v checks the DUT depending on the opcode if the data input is of given length is being completely extended to desired 32-bit value.


