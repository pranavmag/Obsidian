2026-04-04 22:41

Tags: [[increment]] [[decrement]] [[ternary operator]] [[if statements]]

### Increment/Decrement

Pre-Increment: ++x
Post-Increment: x++

Pre-Decrement: --x
Post-Decrement: x--

Each of these increment or decrement a value by 1 but the way that they go about it is different among some. Pre-Increment increments a value by 1 and returns that value whereas Post-Increment copies a current value, increments the real value by 1, and then returns the copy.

Make sure not to use increments within a statement with the same value there like in a function argument for example:

getMax(x, ++x), using something like this is unspecified behavior because the order or evaluation for function arguments in C++ is unspecified, it's up to the compiler. 