## Title: `Stable2::getBandC` function is supposed to return `(uint256 c, uint256 b)`, but doesn't return anything.

## Summary: 
The `Stable2::getBandC` function is supposed to return `(uint256 c, uint256 b)` as we can see from function definition, here:
```js
    function getBandC(
        uint256 Ann,
        uint256 lpTokenSupply,
        uint256 reserves
    ) private pure returns (uint256 c, uint256 b) {
        c = lpTokenSupply * lpTokenSupply / (reserves * N) * lpTokenSupply * A_PRECISION / (Ann * N);
        b = reserves + (lpTokenSupply * A_PRECISION / Ann);
        //@audit - not returning anything
    }
```
But in the function it is just calculating value of c and b, without returning anything.
[link to code](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/src/functions/Stable2.sol#L375-L382)

## Recommended mitigation
Implement return statement.
```
    return(a,b);
```