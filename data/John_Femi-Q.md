## Issue
WellUpgradeable inherits Well.sol, storage gaps should be used to avoid collision of slots on storage space which could cause an overwrite of data and spin into complex issues if not well handled

## Impact
https://github.com/code-423n4/2024-07-basin/blob/main/src/WellUpgradeable.sol#L16
https://github.com/code-423n4/2024-07-basin/blob/main/src/Well.sol#L35

Conflict with storage slots previously written in the proxy

## Recommended Mitigation Steps
Addition of storage gaps to the Wells.sol contract when inherited by WellUpgradeable.sol