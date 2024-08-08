## Description
1. The function `calcReserveAtRatioSwap` intially checks the value of j and sets i the opposite value. Instead of using the conditional operator, an xor operator will be more gas efficient as conditional operator requires multiple JUMP opcodes in its execution while xor does not.
`uint256 i = j == 1 ? 0 : 1` can be changed to `i = j ^ 1`

2. the function `decodeWellData` has an unnecessary if statment which can be removed.
```
if (decimal0 == 0) {
  decimal0 = 18;
}
if (decimal0 == 0) {
  decimal1 = 18;
}
```

can be replaced with
```
if (decimal0 == 0) {
  decimal0 = 18;
  decimal1 = 18;
}
This will save some gas wasted on same if check.

```