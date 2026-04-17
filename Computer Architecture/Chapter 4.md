2026-04-16 21:31

Tags: [[datapath]] [[control unit]] 

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