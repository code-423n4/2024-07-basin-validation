# Non Critical Impact Issues

# 1: Spelling Errors

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

