2026-04-16 21:31

Tags: [[datapath]] [[control unit]] [[pipelining]]

### Control & Datapath

The datapath is the hardware path that data flows through (registers, ALU, memory). The control unit tells the datapath what to do for each instruction. 

There are two steps every instruction uses,
Fetch: PC -> Instruction Memory -> get instruction
Register Read: Read rs1 and rs2 from register file
After this, behavior depends on instruction type


**Core Components:**
PC (Program Counter) -> holds address of current instruction
Instruction Memory -> gives instruction at PC
Register File -> reads 2 registers, writes 1 register
ALU -> does math and logic (Add, Sub, And, Or)
Data Memory -> used for lw/sw
ImmGen -> used for sign extension of immediate values
Mux (Multiplexer) -> chooses between inputs
Control Unit -> sets control signals, deciding which components should be used

Combinational Logic -> outputs depend on inputs, instant output (ALU, Mux)
State Elements -> stores values (registers, memory)
Only update on clock edge -> prevents data from being rewritten early.


**3 Key Muxes**

ALU Input Mux (ALUSrc): 
0 -> register (rs2)
1 -> immediate
used for lw/sw

Writeback Mux (MemtoReg):
0 -> ALU result
1 -> memory data
used for lw

PC Mux:
0 -> PC + 4
1 -> branch target
used in branch instructions like beq


**Control Signals**:
RegWrite -> write to register
ALUSrc -> choose immediate vs register
MemRead -> read memory
MemWrite -> write memory
MemtoReg -> memory to register
Branch -> enables branch
ALUOp -> tells ALU what operation to do


**ALU Control**:
ALUOp = 00 -> add (lw, sw)
ALUOp = 01 -> subtract (beq) 
ALUOp = 10 -> use funct fields (R-type)

Why does lw/sw add? This is because address offsets are calculated through addition, if you have like 12(x6) it will be equivalent to address = x6 + 12, so we need the ALU to add here to calculate that immediate value.

Why does beq subtract? This is because the best way to check if two values are equal is to subtract them and check if the value is 0.


All Instructions 
Fetch -> Read Registers -> use ALU
but the difference is where the ALU input comes from and what happens after ALU.

Control signals configure the datapath so the same hardware can execute different instructions.



### Pipelining

We want to optimize using pipelining to use clock cycles efficiently. Think of it from the perspective of laundry. One scenario is that you put the clothes in the washing machine then move them into the dryer and then fold them, then you do that cycle all over again with the next set of clothes. Another approach to that is putting the clothes in the washing machine, and when you move them to the dryer you put the next set of clothes in the washing machine so that all resources are being used efficiently. Whenever you move to the next step you add another set of clothes if possible.

The same can be done with these instructions. There are 5 different stages in pipelining. 

IF: Instruction Fetch -> grabbing the 32-bit chunk of binary data from memory. The CPU looks at PC, which holds a memory address and goes to that address in Instruction Memory, copies the 32-bit word, and pulls it into the CPU. Updates PC before cycle ends.

ID: Instruction Decode -> Figuring out what those 32 bits mean. The control unit figures out by looking at the opcode to determine the instruction type, identifying source registers and immediate values. It also may further check funct3, funct7, funct5 and stuff like that for further arithmetic operations and reads the values in the source registers as well. The ID/EX register now holds the decoded instruction type, the values pulled from rs1 and rs2, and any immediate values needed.

EX: Execute -> Performing the computation. The ALU takes over and does its job like we mentioned up there in datapath/control section. The EX/MEM register now has the computational result or memory address. 

MEM: Memory Access -> Read from or write to Data Memory (RAM). If the instruction is a Load, the CPU takes the address calculated in the EX stage, goes out to RAM, and retrieves the data. If it's a Store, then it takes a value from a register and pushes it out to RAM at that address. It passes through this stage if it's just a standard arithmetic operation or anything like that. The MEM/WB register now holds either the data pulled from memory or the result passed from the ALU. 

WB: Write-Back -> Save the final result. The CPU takes the final value and writes it to the destination register (rd). 



### Hazards

Data Hazards are when an instruction depends on the result of a previous instruction that hasn't completed yet. For example:

add x19, x0, x1 // writes x19 in cycle 5 (WB stage).
sub x2, x19, x3 // needs x19 in the cycle 3 (EX stage).

We don't want to use old values. 

Structural Hazards are when two instructions need the same hardware resource during the same cycle. If there were only one memory 


