2026-05-13 19:32

Tags: [[control flow]] [[if statements]] [[switch statements]] [[goto statements]]
[[loops]] 

We talked about if statements in a previous chapter but now we are introducing more control flow here. I really liked seeing this table in LearnCPP, it brought an interesting perspective to how I see control flow.

![[Pasted image 20260514184119.png|697]]

### Switch vs If-Else

Switch statements are best used with single expressions (with a non-boolean integral or an enumerated type) that we want to evaluate for equality against a small number of values. Switch statements can get harder to read with more cases but that's up the programmer to see the tradeoffs they'd like to make.

If-else statements are more flexible in that they can test expressions for comparisons other than equality and they can test multiple conditions. They can also use any data type, test for ranges, or be used if the expression evaluates to a bool.

The text did mention about switch statements being able to be optimized into jump tables which are O(1) compared to the if-else statements that go sequentially which is O(n). I have heard about this a lot in there sources as well and researched about it a while back. It may be that compilers without optimizations would yield faster with switch statements but modern compilers should be able to optimize it either into O(1) jump tables or binary search trees of comparisons which should be O(log n). I have seen some assembly outputs that look almost identical across switch statements and if-else statements, but that was only a couple of cases. I would assume as the amount of comparisons increase (like if you are checking x, y, z, or more in your if statement) then it would get harder to optimize. 

To be honest, it really depends on the situation and the programmer. In my RISC-V CPU emulator I couldn't check the opcode, funct7, and funct3 together in the switch statements because you can only check one condition. My first iteration of the code was using a long if-else chain of checking opcode, funct7, and funct3 to identify the assembly instruction but I decided to refactor it into a nested switch statement with opcode first and then funct3 and then an if-else chain for the funct7. I felt that 3 checks for if-else chain may or may not have been difficult for the compiler to optimize so i decided for essentially an O(1) operation for the nested switch.


