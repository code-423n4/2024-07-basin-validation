Missing Revert Statement in the Functions calcLpTokenSupply, calcReserveAtRatioLiquidity, calcReserveAtRatioSwap


The functions calcLpTokenSupply, calcReserveAtRatioLiquidity, calcReserveAtRatioSwap in Stable2.sol are missing a return statement at the end of the functions. This causes the function to potentially exit without returning the expected value, leading to undefined behavior and possible issues in the contract's execution.