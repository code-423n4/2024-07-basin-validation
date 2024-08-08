# Non Critical Impact Issues

## 1: Spelling Errors

Vulnerability details

## Proof of Concept

> ***File: WellUpgradeable.sol***

implmentation can be changed to implementation in the following comments.

https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/src/WellUpgradeable.sol#L57

https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/src/WellUpgradeable.sol#L59

https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/src/WellUpgradeable.sol#L63

https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/src/WellUpgradeable.sol#L74

https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/src/WellUpgradeable.sol#L77

https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/src/WellUpgradeable.sol#L80


aquifier can be changed to aquifer in the following comments.

https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/src/WellUpgradeable.sol#L63 

https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/src/WellUpgradeable.sol#L69 


delgate can be changed to delegate in the following comment.

https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/src/WellUpgradeable.sol#L115 

> ***File: Stable2.sol***

prohibtive can be changed to prohibitive in the following comment.

https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/src/functions/Stable2.sol#L151 


### Tools Used

Manual Analysis


## 2: FLOATING PRAGMA SHOULD BE AVOIDED

Vulnerability details

### Context:

### FLOATING PRAGMA SHOULD BE AVOIDED

### Proof of Concept

> ***3 Files, 3 Instances*** 

https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/src/WellUpgradeable.sol#L3
```solidity
	pragma solidity ^0.8.20;
```

https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/src/functions/Stable2.sol#L3
```solidity
	pragma solidity ^0.8.20;
```

https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/src/functions/StableLUT/Stable2LUT1.sol#L3
```solidity
	pragma solidity ^0.8.20;
```

### Tools Used

Manual Analysis

### Recommended Mitigation Steps

Lock solidity pragmas

## 2: In functions that accept an address as a parameter, there should be a zero address check to prevent bugs

Vulnerability details

### Context:

In Solidity, 'revert' statements are used to undo changes and throw an exception when certain conditions are not met. However, in public and external functions, improper use of `revert` can be exploited for Denial of Service (DoS) attacks. An attacker can intentionally trigger these 'revert' conditions, causing legitimate transactions to consistently fail. For example, if a function relies on specific conditions from user input or contract state, an attacker could manipulate these to continually force reverts, blocking the function's execution. Therefore, it's crucial to design contract logic to handle exceptions properly and avoid scenarios where `revert` can be predictably triggered by malicious actors. This includes careful input validation and considering alternative design patterns that are less susceptible to such abuses.

### Proof of Concept

### Contracts

> ***2 Files, 3 Instances*** 

https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/src/WellUpgradeable.sol#L33-L49
```solidity
   function init(string memory _name, string memory _symbol) external override reinitializer(2) {
       __ERC20Permit_init(_name);
       __ERC20_init(_name, _symbol);
       __ReentrancyGuard_init();
       __UUPSUpgradeable_init();
       __Ownable_init();


       IERC20[] memory _tokens = tokens();
       uint256 tokensLength = _tokens.length;
       for (uint256 i; i < tokensLength - 1; ++i) {
           for (uint256 j = i + 1; j < tokensLength; ++j) {
               if (_tokens[i] == _tokens[j]) {
                   revert DuplicateTokens(_tokens[i]); // found
               }
           }
       }
   }
```

https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/src/functions/Stable2.sol#L310-L325
```solidity
   function decodeWellData(bytes memory data) public view virtual returns (uint256[] memory decimals) {
       (uint256 decimal0, uint256 decimal1) = abi.decode(data, (uint256, uint256));


       // if well data returns 0, assume 18 decimals.
       if (decimal0 == 0) {
           decimal0 = 18;
       }
       if (decimal0 == 0) {
           decimal1 = 18;
       }
       if (decimal0 > 18 || decimal1 > 18) revert InvalidTokenDecimals(); // Found


       decimals = new uint256[](2);
       decimals[0] = decimal0;
       decimals[1] = decimal1;
   }
```
https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/src/functions/Stable2.sol#L114-L144
```solidity
   function calcReserve(
       uint256[] memory reserves,
       uint256 j,
       uint256 lpTokenSupply,
       bytes memory data
   ) public view returns (uint256 reserve) {
       uint256[] memory decimals = decodeWellData(data);
       uint256[] memory scaledReserves = getScaledReserves(reserves, decimals);


       // avoid stack too deep errors.
       (uint256 c, uint256 b) = getBandC(a * N * N, lpTokenSupply, j == 0 ? scaledReserves[1] : scaledReserves[0]);
       reserve = lpTokenSupply;
       uint256 prevReserve;


       for (uint256 i; i < 255; ++i) {
           prevReserve = reserve;
           reserve = _calcReserve(reserve, b, c, lpTokenSupply);
           // Equality with the precision of 1
           // scale reserve down to original precision
           if (reserve > prevReserve) {
               if (reserve - prevReserve <= 1) {
                   return reserve / (10 ** (18 - decimals[j]));
               }
           } else {
               if (prevReserve - reserve <= 1) {
                   return reserve / (10 ** (18 - decimals[j]));
               }
           }
       }
       revert("did not find convergence"); // Found
   }
```

### Tools Used

Manual Analysis


# Low Impact Vulnerabilities

## 1: Missing checks for address(0x0) when updating address state variables 

Num of instances: 2

### Findings 


<details><summary>Click to show findings</summary>

['[93](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/src/WellUpgradeable.sol#L93-L96)']
```solidity
           function upgradeTo(address newImplementation) public override {
       _authorizeUpgrade(newImplementation);
       _upgradeToAndCallUUPS(newImplementation, new bytes(0), false);
   }

```
[‘[104](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/src/WellUpgradeable.sol#L104-L107)’]

```solidity
   function upgradeToAndCall(address newImplementation, bytes memory data) public payable override {
       _authorizeUpgrade(newImplementation);
       _upgradeToAndCallUUPS(newImplementation, data, true);
   }
```


</details>

## 2: Solidity version 0.8.23 won't work on all chains due to MCOPY

Vulnerability details

### Context:

Solidity version 0.8.23 introduces the MCOPY opcode, this may not be implemented on all chains and L2 thus reducing the portability and compatibility of the code. Consider using an earlier solidity version.

### Proof of Concept

> ***3 Files, 3 Instances*** 

https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/src/WellUpgradeable.sol#L3
```solidity
	pragma solidity ^0.8.20;
```

https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/src/functions/Stable2.sol#L3
```solidity
	pragma solidity ^0.8.20;
```

https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/src/functions/StableLUT/Stable2LUT1.sol#L3
```solidity
	pragma solidity ^0.8.20;
```

### Tools Used

Manual Analysis

### Recommended Mitigation Steps

Consider using an earlier solidity version.

