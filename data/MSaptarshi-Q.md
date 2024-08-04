# [QA-1] No use of upgradeable SafeERC20 contract in the project
`WellUpgradeable.sol` makes use of Open Zeppelins OwnableUpgradeable.sol in the file but does not use an upgradeable version of SafeERC20.sol
https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/src/WellUpgradeable.sol#L8
Some tokens like `TUSD`, which has a proxy and implementation contract, if the implementation behind the proxy is changed, it can introduce features which break the protocol, like choosing to not return a bool on transfer(), or changing the balance over time like a rebasing token. 
## Recommended Mitigation Steps
Make use of Open Zeppelins upgradeable version of the SafeERC20.sol contract: https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/token/ERC20/utils/SafeERC20Upgradeable.sol

# [QA-2] Protocol can break for a token with a proxy and implementation contract (like `TUSD`)
https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/src/WellUpgradeable.sol#L40
```
IERC20[] memory _tokens = tokens();
        uint256 tokensLength = _tokens.length;
        for (uint256 i; i < tokensLength - 1; ++i) {
            for (uint256 j = i + 1; j < tokensLength; ++j) {
                if (_tokens[i] == _tokens[j]) {
                    revert DuplicateTokens(_tokens[i]);
                }
            }
        }
```
Basin is supposed to work with tokens which are `upgradable` in nature
For a token like `TUSD` which has a proxy and implementation contract, if the implementation behind the proxy is changed, it can introduce features which break the protocol, like choosing to not return a bool on transfer(), or changing the balance over time like a rebasing token.
## Recommended Mitigation Steps
Developers integrating with upgradable tokens should consider introducing logic that will freeze interactions with the token in question if an upgrade is detected. (e.g. the TUSD adapter used by MakerDAO).
OR have a token whitelist which does not allow such tokens.