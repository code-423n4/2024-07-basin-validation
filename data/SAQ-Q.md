##  Inefficient Loop in `calcReserve` and `calcReserveAtRatioLiquidity`
https://github.com/code-423n4/2024-07-basin/blob/main/src/functions/Stable2.sol#L128-L142

https://github.com/code-423n4/2024-07-basin/blob/main/src/functions/Stable2.sol#L288-L303

## Description
The loop running 255 times to achieve convergence might be inefficient and could be optimized. If convergence is not found within a reasonable number of iterations, the function could be designed to exit early to save gas.


## recommended mitigation
Implement a more efficient convergence algorithm or set a lower iteration limit with early termination when convergence is achieved.