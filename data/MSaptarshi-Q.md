# [QA-1] No use of upgradeable SafeERC20 contract in the project
`WellUpgradeable.sol` makes use of Open Zeppelins OwnableUpgradeable.sol in the file but does not use an upgradeable version of SafeERC20.sol
https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/src/WellUpgradeable.sol#L8
## Recommended Mitigation Steps
Make use of Open Zeppelins upgradeable version of the SafeERC20.sol contract: https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/token/ERC20/utils/SafeERC20Upgradeable.sol
