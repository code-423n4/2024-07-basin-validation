## Description

In the function `calcLpTokenSupply` when after all the 255 iterations, if the ideal  `lpTokenSupply` is not found then the function does not revert anything. It should revert with "did not find convergence" or any other Error code or message like in function `calcReserve`.

`revert("did not find convergence");`