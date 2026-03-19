2026-03-06 00:11

Tags: [[register operands]]

### Register Operands

A CPU can only do one operation at a time so we have register operands. For example,
add a, b, c takes b + c and sets that to a.

A real example we have is f = (g + h) - (i + j).
add t0, g, h
add t1, i, j
sub f, t0, t1

t0 and t1 are temporary registers meanwhile f, g, h, i, and j are stored in saved registers.