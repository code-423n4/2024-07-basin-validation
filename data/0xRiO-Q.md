# [L-1] Missing Length Check for `reserves` Array in `calcReserve()`
- **Description:** The `calcReserve()` function assumes that the `reserves` array always has exactly two elements, but it doesn't explicitly check for this condition. To avoid potential edge cases where the `reserves` array may have an unexpected length, a check should be added to ensure `reserves.length == 2`. This will safeguard against any unexpected inputs that could lead to incorrect calculations or potential security issues.

- **Impact:** Adding this check will handle edge cases where the `reserves` array does not have exactly two elements, making the function more robust and secure. This ensures that the function behaves as expected and avoids potential errors or vulnerabilities that could arise from incorrect input lengths.

- **Recommended Mitigation:** Add the following check at the beginning of the calcReserve() function:
``` solidity 
function calcReserve(
    uint256[] memory reserves,
    /*...*/
) public view returns (/*...*/) {
    //@audit Ensure reserves array has exactly two elements
    require(reserves.length == 2, "Invalid reserves length");
    
    uint256[] memory decimals = decodeWellData(data);
    uint256[] memory scaledReserves = getScaledReserves(reserves, decimals);

    //...
}
```
Checks like this can also be added in and in `calcReserveAtRatioSwap()` where check can be modified to `require(reserves.length == ratios.length && reserves.length ==2, "Invalid array lengths")` 

# [NC-1] Unused zero check can be removed
- **File:** [Stable2.sol](https://github.com/code-423n4/2024-07-basin/blob/main/src/functions/Stable2.sol#L86)

- **Description:** The function `calcLpTokenSupply()` contains a redundant check for `if (sumReserves == 0) return 0;`. This check is unnecessary because the `sumReserves` value is derived from the scaled reserves, calculated using the `getScaledReserves()` function. Since `getScaledReserves()` multiplies `reserves[0]` and `reserves[1]` by a factor of `10 ** (18 - decimals[i])`, `sumReserves` can only be zero if both `reserves[0]` and `reserves[1]` are zero. However, the function already includes an initial check for `if (reserves[0] == 0 && reserves[1] == 0) return 0;`, making the subsequent check for `sumReserves == 0` redundant.

- **Recommended Mitigation:** Remove check `if (sumReserves == 0) return 0;` check from the `calcLpTokenSupply()` function to streamline the code and avoid unnecessary operations. The initial check for `if (reserves[0] == 0 && reserves[1] == 0) return 0;` is sufficient to handle cases where both reserves are zero.