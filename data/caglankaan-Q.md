### Division before multiplication
When dealing with arithmetic operations, it's crucial to prioritize multiplication before division. This practice helps to reduce the risk of rounding errors and increases the precision of the calculations. By rearranging your mathematical operations to perform multiplications first, you can enhance the accuracy and efficiency of your contract's computational processes.

```solidity
Path: ./src/functions/Stable2.sol

380:        c = lpTokenSupply * lpTokenSupply / (reserves * N) * lpTokenSupply * A_PRECISION / (Ann * N);	// @audit-issue
```
[380](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L380-L380), 


#### Recommendation

Review your smart contract code to identify instances where division precedes multiplication. Refactor these calculations by rearranging the order, ensuring multiplication is performed first. This adjustment can significantly improve the precision of your results, especially in environments where every decimal point matters, like financial or token-related contracts.

### Solidity version 0.8.20 might not work on all chains due to `PUSH0`
The Solidity version 0.8.20 employs the recently introduced PUSH0 opcode in the Shanghai EVM. This opcode might not be universally supported across all blockchain networks and Layer 2 solutions. Thus, as a result, it might be not possible to deploy solution with version 0.8.20 >= on some blockchains.

```solidity
Path: ./src/WellUpgradeable.sol

3:pragma solidity ^0.8.20;	// @audit-issue
```
[3](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L3-L3), 


```solidity
Path: ./src/functions/Stable2.sol

3:pragma solidity ^0.8.20;	// @audit-issue
```
[3](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L3-L3), 


```solidity
Path: ./src/functions/StableLUT/Stable2LUT1.sol

3:pragma solidity ^0.8.20;	// @audit-issue
```
[3](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L3-L3), 


#### Recommendation

The Solidity version 0.8.20 employs the recently introduced PUSH0 opcode in the Shanghai EVM. This opcode might not be universally supported across all blockchain networks and Layer 2 solutions. Thus, as a result, it might be not possible to deploy solution with version 0.8.20 >= on some blockchains.

It is recommended to verify whether solution can be deployed on particular blockchain with the Solidity version 0.8.20 >=. Whenever such deployment is not possible due to lack of PUSH0 opcode support and lowering the Solidity version is a must, it is strongly advised to review all feature changes and bugfixes in [Solidity releases](https://soliditylang.org/blog/category/releases/). Some changes may have impact on current implementation and may impose a necessity of maintaining another version of solution.

### Upgradeable contract is missing `__gap[..]` storage variable
Storage gaps are needed to not break storage layouts when adding new variables to base contracts. Listed contracts may not be inherited from but it is good practice to add it now rather than forgetting later.


```solidity
Path: ./src/WellUpgradeable.sol

16:contract WellUpgradeable is Well, UUPSUpgradeable, OwnableUpgradeable {	// @audit-issue
```
[16](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L16-L16), 


#### Recommendation

When designing upgradeable contracts, it's important to include `__gap[..]` storage variables as storage gaps to prevent issues with storage layout changes when adding new variables to base contracts in the future. While the listed contracts may not be directly inherited from, it is a good practice to add `__gap[..]` storage variables now to ensure a robust and future-proof upgradeable contract design. This proactive approach helps avoid potential complications and storage layout conflicts in later stages of contract development.

### Contracts are designed to receive ETH but do not implement function for withdrawal
The following contracts can receive ETH but do not provide a function for withdrawal. This means that any ETH sent to these contracts will be permanently stuck, unable to be retrieved by the contract owner or any other party. Additionally, this issue can also apply to baseTokens, resulting in locked tokens and potential loss of funds.


```solidity
Path: ./src/WellUpgradeable.sol

104:    function upgradeToAndCall(address newImplementation, bytes memory data) public payable override {	// @audit-issue
```
[104](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L104-L104), 


#### Recommendation

To prevent ETH and token lock-up and potential loss of funds, ensure that contracts designed to receive ETH or tokens implement a function for withdrawal. This function should allow contract owners and users to retrieve their funds when needed. Failure to provide a withdrawal mechanism can lead to locked assets and permanent loss, posing a significant risk to contract users and owners.

### Non Disabled Implementation Contract
The upgradeable contracts do not disable initializers in the constructor, as recommended by the imported Initializable contract. This means that anyone can call the initializer on the implementation contract to set the contract variables and assign the roles. To reduce the potential attack surface, `_disableInitializers` in the constructor needs to be called.

```solidity
Path: ./src/WellUpgradeable.sol

16:contract WellUpgradeable is Well, UUPSUpgradeable, OwnableUpgradeable {	// @audit-issue
```
[16](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L16-L16), 


#### Recommendation


Build a constructor function in the upgradeable contracts that calls the disableInitializers() function.

### Excessive Gas Consumption Due to Nested Loops in Solidity
In the Solidity smart contract, there are instances where for-loops are nested within other for-loops. Nested loops in Solidity can lead to significantly increased gas consumption, especially when operating on large data sets. This is because the complexity of nested loops is multiplicative; a double nested loop results in \(O(n^2)\) complexity, and a triple nested loop leads to \(O(n^3)\) complexity. In the context of Ethereum, where each operation consumes a certain amount of gas, such complexity can make transactions prohibitively expensive and may even hit the block gas limit, causing transactions to fail.

```solidity
Path: ./src/WellUpgradeable.sol

43:            for (uint256 j = i + 1; j < tokensLength; ++j) {	// @audit-issue: This for is under another for loop on line 42
```
[43](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L43-L43), 


#### Recommendation


1. **Optimize Data Structures**: Where possible, use more efficient data structures that can reduce the need for nested iterations. This might involve restructuring how data is stored and accessed.
2. **Limit Loop Operations**: Implement checks or mechanisms to limit the number of iterations in each loop. For instance, setting a hard cap on array sizes or breaking out of loops early when a certain condition is met.


### Potential division by zero should have zero checks in place
In Solidity, division by zero is a critical issue that can lead to exceptions and disrupt the normal flow of a contract. It's essential to ensure that the divisor in any division operation is not zero before performing the operation. Failing to check for zero can result in transaction reversion or other unintended consequences. This is especially important in financial contracts or any contract where division operations are used to determine distributions, rewards, or similar calculations. Implementing checks to ensure the divisor is not zero before performing division enhances the robustness and reliability of the contract.

```solidity
Path: ./src/functions/Stable2.sol

381:        b = reserves + (lpTokenSupply * A_PRECISION / Ann);	// @audit-issue: Variable `Ann` not checked.
```
[381](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L381-L381), 


#### Recommendation

Before performing any division operation in Solidity, implement a check to ensure that the divisor is not zero. Use a `require()` statement or a similar assertion to validate the divisor. For example, use `require(divisor != 0, "Divisor cannot be zero");` before executing the division. This precaution prevents exceptions due to division by zero and ensures that your contract remains stable and predictable under all operational conditions. Regularly review your contract's logic to identify and safeguard against potential division by zero occurrences.

### Possible loss of precision
Division by large numbers may result in precision loss due to rounding down, or even the result being erroneously equal to zero. Consider adding checks on the numerator to ensure precision loss is handled appropriately.


```solidity
Path: ./src/functions/Stable2.sol

91:            dP = dP * lpTokenSupply / (scaledReserves[0] * N);	// @audit-issue

92:            dP = dP * lpTokenSupply / (scaledReserves[1] * N);	// @audit-issue

94:            lpTokenSupply = (Ann * sumReserves / A_PRECISION + (dP * N)) * lpTokenSupply	// @audit-issue
95:                / (((Ann - A_PRECISION) * lpTokenSupply / A_PRECISION) + ((N + 1) * dP));

94:            lpTokenSupply = (Ann * sumReserves / A_PRECISION + (dP * N)) * lpTokenSupply	// @audit-issue

95:                / (((Ann - A_PRECISION) * lpTokenSupply / A_PRECISION) + ((N + 1) * dP));	// @audit-issue

135:                    return reserve / (10 ** (18 - decimals[j]));	// @audit-issue

139:                    return reserve / (10 ** (18 - decimals[j]));	// @audit-issue

187:        pd.targetPrice = scaledRatios[i] * PRICE_PRECISION / scaledRatios[j];	// @audit-issue

201:            scaledReserves[i] = parityReserve * pd.lutData.lowPriceI / pd.lutData.precision;	// @audit-issue

202:            scaledReserves[j] = parityReserve * pd.lutData.lowPriceJ / pd.lutData.precision;	// @audit-issue

207:            scaledReserves[i] = parityReserve * pd.lutData.highPriceI / pd.lutData.precision;	// @audit-issue

208:            scaledReserves[j] = parityReserve * pd.lutData.highPriceJ / pd.lutData.precision;	// @audit-issue

215:            pd.maxStepSize = scaledReserves[j] * (pd.lutData.lowPriceJ - pd.lutData.highPriceJ) / pd.lutData.lowPriceJ;	// @audit-issue

217:            pd.maxStepSize = scaledReserves[j] * (pd.lutData.highPriceJ - pd.lutData.lowPriceJ) / pd.lutData.highPriceJ;	// @audit-issue

231:                    return scaledReserves[j] / (10 ** (18 - decimals[j]));	// @audit-issue

235:                    return scaledReserves[j] / (10 ** (18 - decimals[j]));	// @audit-issue

260:        pd.targetPrice = scaledRatios[i] * PRICE_PRECISION / scaledRatios[j];	// @audit-issue

269:            scaledReserves[j] = scaledReserves[i] * pd.lutData.lowPriceJ / pd.lutData.precision;	// @audit-issue

275:            scaledReserves[j] = scaledReserves[i] * pd.lutData.highPriceJ / pd.lutData.precision;	// @audit-issue

283:            pd.maxStepSize = scaledReserves[j] * (pd.lutData.lowPriceJ - pd.lutData.highPriceJ) / pd.lutData.lowPriceJ;	// @audit-issue

285:            pd.maxStepSize = scaledReserves[j] * (pd.lutData.highPriceJ - pd.lutData.lowPriceJ) / pd.lutData.highPriceJ;	// @audit-issue

296:                    return scaledReserves[j] / (10 ** (18 - decimals[j]));	// @audit-issue

300:                    return scaledReserves[j] / (10 ** (18 - decimals[j]));	// @audit-issue

372:        return (reserve * reserve + c) / (reserve * 2 + b - lpTokenSupply);	// @audit-issue

380:        c = lpTokenSupply * lpTokenSupply / (reserves * N) * lpTokenSupply * A_PRECISION / (Ann * N);	// @audit-issue

381:        b = reserves + (lpTokenSupply * A_PRECISION / Ann);	// @audit-issue

392:                - pd.maxStepSize * (pd.targetPrice - pd.currentPrice) / (pd.lutData.highPrice - pd.lutData.lowPrice);	// @audit-issue

397:                + pd.maxStepSize * (pd.currentPrice - pd.targetPrice) / (pd.lutData.highPrice - pd.lutData.lowPrice);	// @audit-issue
```
[91](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L91-L91), [92](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L92-L92), [94](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L94-L95), [94](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L94-L94), [95](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L95-L95), [135](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L135-L135), [139](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L139-L139), [187](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L187-L187), [201](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L201-L201), [202](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L202-L202), [207](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L207-L207), [208](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L208-L208), [215](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L215-L215), [217](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L217-L217), [231](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L231-L231), [235](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L235-L235), [260](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L260-L260), [269](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L269-L269), [275](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L275-L275), [283](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L283-L283), [285](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L285-L285), [296](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L296-L296), [300](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L300-L300), [372](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L372-L372), [380](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L380-L380), [381](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L381-L381), [392](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L392-L392), [397](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L397-L397), 


#### Recommendation

Incorporate strategies in your Solidity contracts to mitigate precision loss in division operations. This can include:
1. Performing checks on the numerator and denominator to ensure they are within a reasonable range to avoid significant rounding errors.
2. Considering the use of fixed-point arithmetic libraries or scaling factors to handle divisions with higher precision.
3. Clearly documenting any inherent limitations of your division logic and providing guidelines for inputs to minimize unexpected behavior.
Always thoroughly test division operations under various scenarios to ensure that the outcomes are consistent with your contract's intended logic and accuracy requirements.

### Floating Pragma
A "floating pragma" in Solidity refers to the practice of using a pragma statement that does not specify a fixed compiler version but instead allows the contract to be compiled with any compatible compiler version. This issue arises when pragma statements like `pragma solidity ^0.8.0;` are used without a specific version number, allowing the contract to be compiled with the latest available compiler version. This can lead to various compatibility and stability issues.


```solidity
Path: ./src/WellUpgradeable.sol

3:pragma solidity ^0.8.20;	// @audit-issue
```
[3](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L3-L3), 


```solidity
Path: ./src/functions/Stable2.sol

3:pragma solidity ^0.8.20;	// @audit-issue
```
[3](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L3-L3), 


```solidity
Path: ./src/functions/StableLUT/Stable2LUT1.sol

3:pragma solidity ^0.8.20;	// @audit-issue
```
[3](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L3-L3), 


#### Recommendation

Consider locking the pragma version whenever possible and avoid using a floating pragma in the final deployment. [Consider known bugs](https://github.com/ethereum/solidity/releases) for the compiler version that is chosen.

### Unneeded initializations of integer variable to `0`.
In Solidity, it is common practice to initialize variables with default values when declaring them. However, initializing integer variables to `0` when they are not subsequently used in the code can lead to unnecessary gas consumption and code clutter. This issue points out instances where such initializations are present but serve no functional purpose.

```solidity
Path: ./src/functions/Stable2.sol

88:        for (uint256 i = 0; i < 255; i++) {	// @audit-issue
```
[88](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L88-L88), 


#### Recommendation


It is recommended not to initialize integer variables to `0` to save some Gas.


### `else`-block not required
One level of nesting can be removed by not having an `else` block when the `if`-block returns, and `if (foo) { return 1; } else { return 2; }` becomes `if (foo) { return 1; } return 2;`


```solidity
Path: ./src/functions/Stable2.sol

388:        if (pd.targetPrice > pd.currentPrice) {
389:            // if the targetPrice is greater than the currentPrice,
390:            // the reserve needs to be decremented to increase currentPrice.
391:            return reserve
392:                - pd.maxStepSize * (pd.targetPrice - pd.currentPrice) / (pd.lutData.highPrice - pd.lutData.lowPrice);
393:        } else {	// @audit-issue
394:            // if the targetPrice is less than the currentPrice,
395:            // the reserve needs to be incremented to decrease currentPrice.
396:            return reserve
397:                + pd.maxStepSize * (pd.currentPrice - pd.targetPrice) / (pd.lutData.highPrice - pd.lutData.lowPrice);
398:        }
```
[393](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L388-L398), 


```solidity
Path: ./src/functions/StableLUT/Stable2LUT1.sol

47:                                if (price < 0.337394e6) {
48:                                    return PriceData(
49:                                        0.337394e6,
50:                                        0,
51:                                        7.689965795021471706e18,
52:                                        0.30624e6,
53:                                        0,
54:                                        8.612761690424049377e18,
55:                                        1e18
56:                                    );
57:                                } else {	// @audit-issue
58:                                    return PriceData(
59:                                        0.370355e6,
60:                                        0,
61:                                        6.866040888412029197e18,
62:                                        0.337394e6,
63:                                        0,
64:                                        7.689965795021471706e18,
65:                                        1e18
66:                                    );
67:                                }

77:                                if (price < 0.440934e6) {
78:                                    return PriceData(
79:                                        0.440934e6,
80:                                        0,
81:                                        5.473565759257038366e18,
82:                                        0.404944e6,
83:                                        0,
84:                                        6.130393650367882863e18,
85:                                        1e18
86:                                    );
87:                                } else {	// @audit-issue
88:                                    return PriceData(
89:                                        0.478063e6,
90:                                        0,
91:                                        4.887112285050926097e18,
92:                                        0.440934e6,
93:                                        0,
94:                                        5.473565759257038366e18,
95:                                        1e18
96:                                    );
97:                                }

104:                            if (price < 0.554558e6) {
105:                                return PriceData(
106:                                    0.554558e6, 0, 3.89597599254697613e18, 0.516039e6, 0, 4.363493111652613443e18, 1e18
107:                                );
108:                            } else {	// @audit-issue
109:                                return PriceData(
110:                                    0.59332e6, 0, 3.478549993345514402e18, 0.554558e6, 0, 3.89597599254697613e18, 1e18
111:                                );
112:                            }

119:                                if (price < 0.632052e6) {
120:                                    return PriceData(
121:                                        0.632052e6,
122:                                        0,
123:                                        3.105848208344209382e18,
124:                                        0.59332e6,
125:                                        0,
126:                                        3.478549993345514402e18,
127:                                        1e18
128:                                    );
129:                                } else {	// @audit-issue
130:                                    return PriceData(
131:                                        0.670518e6,
132:                                        0,
133:                                        2.773078757450186949e18,
134:                                        0.632052e6,
135:                                        0,
136:                                        3.105848208344209382e18,
137:                                        1e18
138:                                    );
139:                                }

146:                            if (price < 0.746003e6) {
147:                                return PriceData(
148:                                    0.746003e6, 0, 2.210681407406080101e18, 0.708539e6, 0, 2.475963176294809553e18, 1e18
149:                                );
150:                            } else {	// @audit-issue
151:                                return PriceData(
152:                                    0.782874e6, 0, 1.973822685183999948e18, 0.746003e6, 0, 2.210681407406080101e18, 1e18
153:                                );
154:                            }

159:                                if (price < 0.819199e6) {
160:                                    return PriceData(
161:                                        0.819199e6,
162:                                        0,
163:                                        1.762341683200000064e18,
164:                                        0.782874e6,
165:                                        0,
166:                                        1.973822685183999948e18,
167:                                        1e18
168:                                    );
169:                                } else {	// @audit-issue
170:                                    return PriceData(
171:                                        0.855108e6,
172:                                        0,
173:                                        1.573519359999999923e18,
174:                                        0.819199e6,
175:                                        0,
176:                                        1.762341683200000064e18,
177:                                        1e18
178:                                    );
179:                                }

186:                            if (price < 0.879393e6) {
187:                                return PriceData(
188:                                    0.879393e6, 0, 1.456811172527798348e18, 0.873157e6, 0, 1.485947395978354457e18, 1e18
189:                                );
190:                            } else {	// @audit-issue
191:                                return PriceData(
192:                                    0.885627e6, 0, 1.428246247576273165e18, 0.879393e6, 0, 1.456811172527798348e18, 1e18
193:                                );
194:                            }

203:                                if (price < 0.89081e6) {
204:                                    return PriceData(
205:                                        0.89081e6,
206:                                        0,
207:                                        1.404927999999999955e18,
208:                                        0.885627e6,
209:                                        0,
210:                                        1.428246247576273165e18,
211:                                        1e18
212:                                    );
213:                                } else {	// @audit-issue
214:                                    return PriceData(
215:                                        0.891863e6,
216:                                        0,
217:                                        1.400241419192424397e18,
218:                                        0.89081e6,
219:                                        0,
220:                                        1.404927999999999955e18,
221:                                        1e18
222:                                    );
223:                                }

231:                                if (price < 0.904344e6) {
232:                                    return PriceData(
233:                                        0.904344e6,
234:                                        0,
235:                                        1.345868338324129665e18,
236:                                        0.898101e6,
237:                                        0,
238:                                        1.372785705090612263e18,
239:                                        1e18
240:                                    );
241:                                } else {	// @audit-issue
242:                                    return PriceData(
243:                                        0.910594e6,
244:                                        0,
245:                                        1.319478763062872151e18,
246:                                        0.904344e6,
247:                                        0,
248:                                        1.345868338324129665e18,
249:                                        1e18
250:                                    );
251:                                }

261:                                if (price < 0.92312e6) {
262:                                    return PriceData(
263:                                        0.92312e6,
264:                                        0,
265:                                        1.268241794562545266e18,
266:                                        0.916852e6,
267:                                        0,
268:                                        1.293606630453796313e18,
269:                                        1e18
270:                                    );
271:                                } else {	// @audit-issue
272:                                    return PriceData(
273:                                        0.9266e6,
274:                                        0,
275:                                        1.254399999999999959e18,
276:                                        0.92312e6,
277:                                        0,
278:                                        1.268241794562545266e18,
279:                                        1e18
280:                                    );
281:                                }

288:                            if (price < 0.935697e6) {
289:                                return PriceData(
290:                                    0.935697e6, 0, 1.218994419994757328e18, 0.929402e6, 0, 1.243374308394652239e18, 1e18
291:                                );
292:                            } else {	// @audit-issue
293:                                return PriceData(
294:                                    0.94201e6, 0, 1.195092568622310836e18, 0.935697e6, 0, 1.218994419994757328e18, 1e18
295:                                );
296:                            }

303:                                if (price < 0.948343e6) {
304:                                    return PriceData(
305:                                        0.948343e6,
306:                                        0,
307:                                        1.171659381002265521e18,
308:                                        0.94201e6,
309:                                        0,
310:                                        1.195092568622310836e18,
311:                                        1e18
312:                                    );
313:                                } else {	// @audit-issue
314:                                    return PriceData(
315:                                        0.954697e6,
316:                                        0,
317:                                        1.14868566764928004e18,
318:                                        0.948343e6,
319:                                        0,
320:                                        1.171659381002265521e18,
321:                                        1e18
322:                                    );
323:                                }

330:                            if (price < 0.962847e6) {
331:                                return PriceData(
332:                                    0.962847e6, 0, 1.120000000000000107e18, 0.961075e6, 0, 1.12616241926400007e18, 1e18
333:                                );
334:                            } else {	// @audit-issue
335:                                return PriceData(
336:                                    0.96748e6, 0, 1.104080803200000016e18, 0.962847e6, 0, 1.120000000000000107e18, 1e18
337:                                );
338:                            }

343:                                if (price < 0.973914e6) {
344:                                    return PriceData(
345:                                        0.973914e6,
346:                                        0,
347:                                        1.082432159999999977e18,
348:                                        0.96748e6,
349:                                        0,
350:                                        1.104080803200000016e18,
351:                                        1e18
352:                                    );
353:                                } else {	// @audit-issue
354:                                    return PriceData(
355:                                        0.98038e6,
356:                                        0,
357:                                        1.061208000000000151e18,
358:                                        0.973914e6,
359:                                        0,
360:                                        1.082432159999999977e18,
361:                                        1e18
362:                                    );
363:                                }

370:                            if (price < 0.993421e6) {
371:                                return PriceData(
372:                                    0.993421e6, 0, 1.020000000000000018e18, 0.986882e6, 0, 1.040399999999999991e18, 1e18
373:                                );
374:                            } else {	// @audit-issue
375:                                return PriceData(
376:                                    1.006758e6, 0, 0.980000000000000093e18, 0.993421e6, 0, 1.020000000000000018e18, 1e18
377:                                );
378:                            }

389:                                if (price < 1.013564e6) {
390:                                    return PriceData(
391:                                        1.013564e6,
392:                                        0,
393:                                        0.960400000000000031e18,
394:                                        1.006758e6,
395:                                        0,
396:                                        0.980000000000000093e18,
397:                                        1e18
398:                                    );
399:                                } else {	// @audit-issue
400:                                    return PriceData(
401:                                        1.020422e6,
402:                                        0,
403:                                        0.941192000000000029e18,
404:                                        1.013564e6,
405:                                        0,
406:                                        0.960400000000000031e18,
407:                                        1e18
408:                                    );
409:                                }

417:                                if (price < 1.034307e6) {
418:                                    return PriceData(
419:                                        1.034307e6,
420:                                        0,
421:                                        0.903920796799999926e18,
422:                                        1.027335e6,
423:                                        0,
424:                                        0.922368159999999992e18,
425:                                        1e18
426:                                    );
427:                                } else {	// @audit-issue
428:                                    return PriceData(
429:                                        1.041342e6,
430:                                        0,
431:                                        0.885842380864000023e18,
432:                                        1.034307e6,
433:                                        0,
434:                                        0.903920796799999926e18,
435:                                        1e18
436:                                    );
437:                                }

447:                                if (price < 1.048443e6) {
448:                                    return PriceData(
449:                                        1.048443e6,
450:                                        0,
451:                                        0.868125533246720038e18,
452:                                        1.04366e6,
453:                                        0,
454:                                        0.880000000000000004e18,
455:                                        1e18
456:                                    );
457:                                } else {	// @audit-issue
458:                                    return PriceData(
459:                                        1.055613e6,
460:                                        0,
461:                                        0.8507630225817856e18,
462:                                        1.048443e6,
463:                                        0,
464:                                        0.868125533246720038e18,
465:                                        1e18
466:                                    );
467:                                }

474:                            if (price < 1.070179e6) {
475:                                return PriceData(
476:                                    1.070179e6, 0, 0.81707280688754691e18, 1.062857e6, 0, 0.833747762130149894e18, 1e18
477:                                );
478:                            } else {	// @audit-issue
479:                                return PriceData(
480:                                    1.077582e6, 0, 0.800731350749795956e18, 1.070179e6, 0, 0.81707280688754691e18, 1e18
481:                                );
482:                            }

489:                                if (price < 1.085071e6) {
490:                                    return PriceData(
491:                                        1.085071e6,
492:                                        0,
493:                                        0.784716723734800059e18,
494:                                        1.077582e6,
495:                                        0,
496:                                        0.800731350749795956e18,
497:                                        1e18
498:                                    );
499:                                } else {	// @audit-issue
500:                                    return PriceData(
501:                                        1.090025e6,
502:                                        0,
503:                                        0.774399999999999977e18,
504:                                        1.085071e6,
505:                                        0,
506:                                        0.784716723734800059e18,
507:                                        1e18
508:                                    );
509:                                }

516:                            if (price < 1.100323e6) {
517:                                return PriceData(
518:                                    1.100323e6, 0, 0.753641941474902044e18, 1.09265e6, 0, 0.769022389260104022e18, 1e18
519:                                );
520:                            } else {	// @audit-issue
521:                                return PriceData(
522:                                    1.108094e6, 0, 0.738569102645403985e18, 1.100323e6, 0, 0.753641941474902044e18, 1e18
523:                                );
524:                            }

529:                                if (price < 1.115967e6) {
530:                                    return PriceData(
531:                                        1.115967e6,
532:                                        0,
533:                                        0.723797720592495919e18,
534:                                        1.108094e6,
535:                                        0,
536:                                        0.738569102645403985e18,
537:                                        1e18
538:                                    );
539:                                } else {	// @audit-issue
540:                                    return PriceData(
541:                                        1.123949e6,
542:                                        0,
543:                                        0.709321766180645907e18,
544:                                        1.115967e6,
545:                                        0,
546:                                        0.723797720592495919e18,
547:                                        1e18
548:                                    );
549:                                }

556:                            if (price < 1.14011e6) {
557:                                return PriceData(
558:                                    1.14011e6, 0, 0.681471999999999967e18, 1.132044e6, 0, 0.695135330857033051e18, 1e18
559:                                );
560:                            } else {	// @audit-issue
561:                                return PriceData(
562:                                    1.140253e6, 0, 0.681232624239892393e18, 1.14011e6, 0, 0.681471999999999967e18, 1e18
563:                                );
564:                            }

573:                                if (price < 1.148586e6) {
574:                                    return PriceData(
575:                                        1.148586e6,
576:                                        0,
577:                                        0.667607971755094454e18,
578:                                        1.140253e6,
579:                                        0,
580:                                        0.681232624239892393e18,
581:                                        1e18
582:                                    );
583:                                } else {	// @audit-issue
584:                                    return PriceData(
585:                                        1.195079e6,
586:                                        0,
587:                                        0.599695360000000011e18,
588:                                        1.148586e6,
589:                                        0,
590:                                        0.667607971755094454e18,
591:                                        1e18
592:                                    );
593:                                }

600:                            if (price < 1.325188e6) {
601:                                return PriceData(
602:                                    1.325188e6, 0, 0.464404086784000025e18, 1.256266e6, 0, 0.527731916799999978e18, 1e18
603:                                );
604:                            } else {	// @audit-issue
605:                                return PriceData(
606:                                    1.403579e6, 0, 0.408675596369920013e18, 1.325188e6, 0, 0.464404086784000025e18, 1e18
607:                                );
608:                            }

613:                                if (price < 1.493424e6) {
614:                                    return PriceData(
615:                                        1.493424e6,
616:                                        0,
617:                                        0.359634524805529598e18,
618:                                        1.403579e6,
619:                                        0,
620:                                        0.408675596369920013e18,
621:                                        1e18
622:                                    );
623:                                } else {	// @audit-issue
624:                                    return PriceData(
625:                                        1.596984e6,
626:                                        0,
627:                                        0.316478381828866062e18,
628:                                        1.493424e6,
629:                                        0,
630:                                        0.359634524805529598e18,
631:                                        1e18
632:                                    );
633:                                }

640:                            if (price < 1.855977e6) {
641:                                return PriceData(
642:                                    1.855977e6, 0, 0.245080858888273884e18, 1.716848e6, 0, 0.278500976009402101e18, 1e18
643:                                );
644:                            } else {	// @audit-issue
645:                                return PriceData(
646:                                    2.01775e6, 0, 0.215671155821681004e18, 1.855977e6, 0, 0.245080858888273884e18, 1e18
647:                                );
648:                            }

655:                                if (price < 2.206036e6) {
656:                                    return PriceData(
657:                                        2.206036e6,
658:                                        0,
659:                                        0.189790617123079292e18,
660:                                        2.01775e6,
661:                                        0,
662:                                        0.215671155821681004e18,
663:                                        1e18
664:                                    );
665:                                } else {	// @audit-issue
666:                                    return PriceData(
667:                                        2.425256e6,
668:                                        0,
669:                                        0.167015743068309769e18,
670:                                        2.206036e6,
671:                                        0,
672:                                        0.189790617123079292e18,
673:                                        1e18
674:                                    );
675:                                }

682:                            if (price < 2.977411e6) {
683:                                return PriceData(
684:                                    2.977411e6, 0, 0.129336991432099091e18, 2.680458e6, 0, 0.146973853900112583e18, 1e18
685:                                );
686:                            } else {	// @audit-issue
687:                                return PriceData(
688:                                    3.322705e6, 0, 0.113816552460247203e18, 2.977411e6, 0, 0.129336991432099091e18, 1e18
689:                                );
690:                            }

695:                                if (price < 3.723858e6) {
696:                                    return PriceData(
697:                                        3.723858e6,
698:                                        0,
699:                                        0.100158566165017532e18,
700:                                        3.322705e6,
701:                                        0,
702:                                        0.113816552460247203e18,
703:                                        1e18
704:                                    );
705:                                } else {	// @audit-issue
706:                                    return PriceData(
707:                                        4.189464e6,
708:                                        0,
709:                                        0.088139538225215433e18,
710:                                        3.723858e6,
711:                                        0,
712:                                        0.100158566165017532e18,
713:                                        1e18
714:                                    );
715:                                }

722:                            if (price < 10.37089e6) {
723:                                return PriceData(
724:                                    10.37089e6, 0, 0.035714285714285712e18, 4.729321e6, 0, 0.077562793638189589e18, 1e18
725:                                );
726:                            } else {	// @audit-issue
727:                                revert("LUT: Invalid price");
728:                            }

761:                                if (price < 0.237671e6) {
762:                                    return PriceData(
763:                                        0.237671e6,
764:                                        0.20510144474217018e18,
765:                                        2.337718072004858261e18,
766:                                        0.213318e6,
767:                                        0.188693329162796575e18,
768:                                        2.410556040105746423e18,
769:                                        1e18
770:                                    );
771:                                } else {	// @audit-issue
772:                                    return PriceData(
773:                                        0.264147e6,
774:                                        0.222936352980619729e18,
775:                                        2.26657220303422724e18,
776:                                        0.237671e6,
777:                                        0.20510144474217018e18,
778:                                        2.337718072004858261e18,
779:                                        1e18
780:                                    );
781:                                }

785:                                if (price < 0.292771e6) {
786:                                    return PriceData(
787:                                        0.292771e6,
788:                                        0.242322122805021467e18,
789:                                        2.196897480682568293e18,
790:                                        0.264147e6,
791:                                        0.222936352980619729e18,
792:                                        2.26657220303422724e18,
793:                                        1e18
794:                                    );
795:                                } else {	// @audit-issue
796:                                    return PriceData(
797:                                        0.323531e6,
798:                                        0.263393611744588529e18,
799:                                        2.128468246736633152e18,
800:                                        0.292771e6,
801:                                        0.242322122805021467e18,
802:                                        2.196897480682568293e18,
803:                                        1e18
804:                                    );
805:                                }

807:                                if (price < 0.356373e6) {
808:                                    return PriceData(
809:                                        0.356373e6,
810:                                        0.286297404070204931e18,
811:                                        2.061053544007124483e18,
812:                                        0.323531e6,
813:                                        0.263393611744588529e18,
814:                                        2.128468246736633152e18,
815:                                        1e18
816:                                    );
817:                                } else {	// @audit-issue
818:                                    return PriceData(
819:                                        0.391201e6,
820:                                        0.311192830511092366e18,
821:                                        1.994416599735895801e18,
822:                                        0.356373e6,
823:                                        0.286297404070204931e18,
824:                                        2.061053544007124483e18,
825:                                        1e18
826:                                    );
827:                                }

833:                                if (price < 0.427871e6) {
834:                                    return PriceData(
835:                                        0.427871e6,
836:                                        0.338253076642491657e18,
837:                                        1.92831441898410505e18,
838:                                        0.391201e6,
839:                                        0.311192830511092366e18,
840:                                        1.994416599735895801e18,
841:                                        1e18
842:                                    );
843:                                } else {	// @audit-issue
844:                                    return PriceData(
845:                                        0.466197e6,
846:                                        0.367666387654882243e18,
847:                                        1.86249753363281334e18,
848:                                        0.427871e6,
849:                                        0.338253076642491657e18,
850:                                        1.92831441898410505e18,
851:                                        1e18
852:                                    );
853:                                }

855:                                if (price < 0.50596e6) {
856:                                    return PriceData(
857:                                        0.50596e6,
858:                                        0.399637377885741607e18,
859:                                        1.796709969924970451e18,
860:                                        0.466197e6,
861:                                        0.367666387654882243e18,
862:                                        1.86249753363281334e18,
863:                                        1e18
864:                                    );
865:                                } else {	// @audit-issue
866:                                    return PriceData(
867:                                        0.546918e6,
868:                                        0.434388454223632148e18,
869:                                        1.73068952191306602e18,
870:                                        0.50596e6,
871:                                        0.399637377885741607e18,
872:                                        1.796709969924970451e18,
873:                                        1e18
874:                                    );
875:                                }

879:                                if (price < 0.588821e6) {
880:                                    return PriceData(
881:                                        0.588821e6,
882:                                        0.472161363286556723e18,
883:                                        1.664168452923131536e18,
884:                                        0.546918e6,
885:                                        0.434388454223632148e18,
886:                                        1.73068952191306602e18,
887:                                        1e18
888:                                    );
889:                                } else {	// @audit-issue
890:                                    return PriceData(
891:                                        0.631434e6,
892:                                        0.513218873137561538e18,
893:                                        1.596874796852916001e18,
894:                                        0.588821e6,
895:                                        0.472161363286556723e18,
896:                                        1.664168452923131536e18,
897:                                        1e18
898:                                    );
899:                                }

901:                                if (price < 0.67456e6) {
902:                                    return PriceData(
903:                                        0.67456e6,
904:                                        0.55784660123648e18,
905:                                        1.52853450260679824e18,
906:                                        0.631434e6,
907:                                        0.513218873137561538e18,
908:                                        1.596874796852916001e18,
909:                                        1e18
910:                                    );
911:                                } else {	// @audit-issue
912:                                    return PriceData(
913:                                        0.718073e6,
914:                                        0.606355001344e18,
915:                                        1.458874768183093584e18,
916:                                        0.67456e6,
917:                                        0.55784660123648e18,
918:                                        1.52853450260679824e18,
919:                                        1e18
920:                                    );
921:                                }

929:                                if (price < 0.76195e6) {
930:                                    return PriceData(
931:                                        0.76195e6,
932:                                        0.659081523200000019e18,
933:                                        1.387629060213009469e18,
934:                                        0.718073e6,
935:                                        0.606355001344e18,
936:                                        1.458874768183093584e18,
937:                                        1e18
938:                                    );
939:                                } else {	// @audit-issue
940:                                    return PriceData(
941:                                        0.769833e6,
942:                                        0.668971758569680497e18,
943:                                        1.37471571145172633e18,
944:                                        0.76195e6,
945:                                        0.659081523200000019e18,
946:                                        1.387629060213009469e18,
947:                                        1e18
948:                                    );
949:                                }

951:                                if (price < 0.775161e6) {
952:                                    return PriceData(
953:                                        0.775161e6,
954:                                        0.675729049060283415e18,
955:                                        1.365968375000512491e18,
956:                                        0.769833e6,
957:                                        0.668971758569680497e18,
958:                                        1.37471571145172633e18,
959:                                        1e18
960:                                    );
961:                                } else {	// @audit-issue
962:                                    return PriceData(
963:                                        0.780497e6,
964:                                        0.682554595010387288e18,
965:                                        1.357193251389227306e18,
966:                                        0.775161e6,
967:                                        0.675729049060283415e18,
968:                                        1.365968375000512491e18,
969:                                        1e18
970:                                    );
971:                                }

975:                                if (price < 0.785842e6) {
976:                                    return PriceData(
977:                                        0.785842e6,
978:                                        0.689449085869078049e18,
979:                                        1.34838993014876074e18,
980:                                        0.780497e6,
981:                                        0.682554595010387288e18,
982:                                        1.357193251389227306e18,
983:                                        1e18
984:                                    );
985:                                } else {	// @audit-issue
986:                                    return PriceData(
987:                                        0.791195e6,
988:                                        0.696413218049573679e18,
989:                                        1.339558007037547016e18,
990:                                        0.785842e6,
991:                                        0.689449085869078049e18,
992:                                        1.34838993014876074e18,
993:                                        1e18
994:                                    );
995:                                }

997:                                if (price < 0.796558e6) {
998:                                    return PriceData(
999:                                        0.796558e6,
1000:                                        0.703447694999569495e18,
1001:                                        1.330697084427678423e18,
1002:                                        0.791195e6,
1003:                                        0.696413218049573679e18,
1004:                                        1.339558007037547016e18,
1005:                                        1e18
1006:                                    );
1007:                                } else {	// @audit-issue
1008:                                    return PriceData(
1009:                                        0.801931e6,
1010:                                        0.710553227272292309e18,
1011:                                        1.321806771708554873e18,
1012:                                        0.796558e6,
1013:                                        0.703447694999569495e18,
1014:                                        1.330697084427678423e18,
1015:                                        1e18
1016:                                    );
1017:                                }

1023:                                if (price < 0.806314e6) {
1024:                                    return PriceData(
1025:                                        0.806314e6,
1026:                                        0.716392959999999968e18,
1027:                                        1.314544530202049311e18,
1028:                                        0.801931e6,
1029:                                        0.710553227272292309e18,
1030:                                        1.321806771708554873e18,
1031:                                        1e18
1032:                                    );
1033:                                } else {	// @audit-issue
1034:                                    return PriceData(
1035:                                        0.807315e6,
1036:                                        0.717730532598275128e18,
1037:                                        1.312886685708826162e18,
1038:                                        0.806314e6,
1039:                                        0.716392959999999968e18,
1040:                                        1.314544530202049311e18,
1041:                                        1e18
1042:                                    );
1043:                                }

1045:                                if (price < 0.812711e6) {
1046:                                    return PriceData(
1047:                                        0.812711e6,
1048:                                        0.724980335957853717e18,
1049:                                        1.303936451137418295e18,
1050:                                        0.807315e6,
1051:                                        0.717730532598275128e18,
1052:                                        1.312886685708826162e18,
1053:                                        1e18
1054:                                    );
1055:                                } else {	// @audit-issue
1056:                                    return PriceData(
1057:                                        0.818119e6,
1058:                                        0.732303369654397684e18,
1059:                                        1.294955701044462559e18,
1060:                                        0.812711e6,
1061:                                        0.724980335957853717e18,
1062:                                        1.303936451137418295e18,
1063:                                        1e18
1064:                                    );
1065:                                }

1069:                                if (price < 0.82354e6) {
1070:                                    return PriceData(
1071:                                        0.82354e6,
1072:                                        0.73970037338828043e18,
1073:                                        1.285944077302980215e18,
1074:                                        0.818119e6,
1075:                                        0.732303369654397684e18,
1076:                                        1.294955701044462559e18,
1077:                                        1e18
1078:                                    );
1079:                                } else {	// @audit-issue
1080:                                    return PriceData(
1081:                                        0.828976e6,
1082:                                        0.74717209433159637e18,
1083:                                        1.276901231112211654e18,
1084:                                        0.82354e6,
1085:                                        0.73970037338828043e18,
1086:                                        1.285944077302980215e18,
1087:                                        1e18
1088:                                    );
1089:                                }

1109:                                if (price < 0.839894e6) {
1110:                                    return PriceData(
1111:                                        0.839894e6,
1112:                                        0.762342714347103767e18,
1113:                                        1.258720525989716954e18,
1114:                                        0.834426e6,
1115:                                        0.754719287203632794e18,
1116:                                        1.267826823523503732e18,
1117:                                        1e18
1118:                                    );
1119:                                } else {	// @audit-issue
1120:                                    return PriceData(
1121:                                        0.845379e6,
1122:                                        0.770043145805155316e18,
1123:                                        1.249582020939133509e18,
1124:                                        0.839894e6,
1125:                                        0.762342714347103767e18,
1126:                                        1.258720525989716954e18,
1127:                                        1e18
1128:                                    );
1129:                                }

1131:                                if (price < 0.850882e6) {
1132:                                    return PriceData(
1133:                                        0.850882e6,
1134:                                        0.777821359399146761e18,
1135:                                        1.240411002374896432e18,
1136:                                        0.845379e6,
1137:                                        0.770043145805155316e18,
1138:                                        1.249582020939133509e18,
1139:                                        1e18
1140:                                    );
1141:                                } else {	// @audit-issue
1142:                                    return PriceData(
1143:                                        0.851493e6,
1144:                                        0.778688000000000047e18,
1145:                                        1.239392846883276889e18,
1146:                                        0.850882e6,
1147:                                        0.777821359399146761e18,
1148:                                        1.240411002374896432e18,
1149:                                        1e18
1150:                                    );
1151:                                }

1155:                                if (price < 0.856405e6) {
1156:                                    return PriceData(
1157:                                        0.856405e6,
1158:                                        0.785678140807218983e18,
1159:                                        1.231207176501035727e18,
1160:                                        0.851493e6,
1161:                                        0.778688000000000047e18,
1162:                                        1.239392846883276889e18,
1163:                                        1e18
1164:                                    );
1165:                                } else {	// @audit-issue
1166:                                    return PriceData(
1167:                                        0.86195e6,
1168:                                        0.793614283643655494e18,
1169:                                        1.221970262376178118e18,
1170:                                        0.856405e6,
1171:                                        0.785678140807218983e18,
1172:                                        1.231207176501035727e18,
1173:                                        1e18
1174:                                    );
1175:                                }

1177:                                if (price < 0.867517e6) {
1178:                                    return PriceData(
1179:                                        0.867517e6,
1180:                                        0.801630589539045979e18,
1181:                                        1.212699992596070864e18,
1182:                                        0.86195e6,
1183:                                        0.793614283643655494e18,
1184:                                        1.221970262376178118e18,
1185:                                        1e18
1186:                                    );
1187:                                } else {	// @audit-issue
1188:                                    return PriceData(
1189:                                        0.873109e6,
1190:                                        0.809727868221258529e18,
1191:                                        1.203396114006087814e18,
1192:                                        0.867517e6,
1193:                                        0.801630589539045979e18,
1194:                                        1.212699992596070864e18,
1195:                                        1e18
1196:                                    );
1197:                                }

1203:                                if (price < 0.878727e6) {
1204:                                    return PriceData(
1205:                                        0.878727e6,
1206:                                        0.817906937597230987e18,
1207:                                        1.194058388444914964e18,
1208:                                        0.873109e6,
1209:                                        0.809727868221258529e18,
1210:                                        1.203396114006087814e18,
1211:                                        1e18
1212:                                    );
1213:                                } else {	// @audit-issue
1214:                                    return PriceData(
1215:                                        0.884372e6,
1216:                                        0.826168623835586646e18,
1217:                                        1.18468659352065786e18,
1218:                                        0.878727e6,
1219:                                        0.817906937597230987e18,
1220:                                        1.194058388444914964e18,
1221:                                        1e18
1222:                                    );
1223:                                }

1225:                                if (price < 0.890047e6) {
1226:                                    return PriceData(
1227:                                        0.890047e6,
1228:                                        0.834513761450087599e18,
1229:                                        1.17528052342063094e18,
1230:                                        0.884372e6,
1231:                                        0.826168623835586646e18,
1232:                                        1.18468659352065786e18,
1233:                                        1e18
1234:                                    );
1235:                                } else {	// @audit-issue
1236:                                    return PriceData(
1237:                                        0.895753e6,
1238:                                        0.84294319338392687e18,
1239:                                        1.16583998975613734e18,
1240:                                        0.890047e6,
1241:                                        0.834513761450087599e18,
1242:                                        1.17528052342063094e18,
1243:                                        1e18
1244:                                    );
1245:                                }

1249:                                if (price < 0.898085e6) {
1250:                                    return PriceData(
1251:                                        0.898085e6,
1252:                                        0.846400000000000041e18,
1253:                                        1.161985895520041945e18,
1254:                                        0.895753e6,
1255:                                        0.84294319338392687e18,
1256:                                        1.16583998975613734e18,
1257:                                        1e18
1258:                                    );
1259:                                } else {	// @audit-issue
1260:                                    return PriceData(
1261:                                        0.901491e6,
1262:                                        0.851457771094875637e18,
1263:                                        1.156364822443562979e18,
1264:                                        0.898085e6,
1265:                                        0.846400000000000041e18,
1266:                                        1.161985895520041945e18,
1267:                                        1e18
1268:                                    );
1269:                                }

1287:                                if (price < 0.913079e6) {
1288:                                    return PriceData(
1289:                                        0.913079e6,
1290:                                        0.868745812768978332e18,
1291:                                        1.137310003616810228e18,
1292:                                        0.907266e6,
1293:                                        0.860058354641288547e18,
1294:                                        1.146854870623147615e18,
1295:                                        1e18
1296:                                    );
1297:                                } else {	// @audit-issue
1298:                                    return PriceData(
1299:                                        0.918932e6,
1300:                                        0.877521022998967948e18,
1301:                                        1.127730111926438461e18,
1302:                                        0.913079e6,
1303:                                        0.868745812768978332e18,
1304:                                        1.137310003616810228e18,
1305:                                        1e18
1306:                                    );
1307:                                }

1309:                                if (price < 0.924827e6) {
1310:                                    return PriceData(
1311:                                        0.924827e6,
1312:                                        0.88638487171612923e18,
1313:                                        1.118115108274055913e18,
1314:                                        0.918932e6,
1315:                                        0.877521022998967948e18,
1316:                                        1.127730111926438461e18,
1317:                                        1e18
1318:                                    );
1319:                                } else {	// @audit-issue
1320:                                    return PriceData(
1321:                                        0.930767e6,
1322:                                        0.895338254258716493e18,
1323:                                        1.10846492868530544e18,
1324:                                        0.924827e6,
1325:                                        0.88638487171612923e18,
1326:                                        1.118115108274055913e18,
1327:                                        1e18
1328:                                    );
1329:                                }

1333:                                if (price < 0.936756e6) {
1334:                                    return PriceData(
1335:                                        0.936756e6,
1336:                                        0.90438207500880452e18,
1337:                                        1.09877953361768621e18,
1338:                                        0.930767e6,
1339:                                        0.895338254258716493e18,
1340:                                        1.10846492868530544e18,
1341:                                        1e18
1342:                                    );
1343:                                } else {	// @audit-issue
1344:                                    return PriceData(
1345:                                        0.942795e6,
1346:                                        0.913517247483640937e18,
1347:                                        1.089058909134983155e18,
1348:                                        0.936756e6,
1349:                                        0.90438207500880452e18,
1350:                                        1.09877953361768621e18,
1351:                                        1e18
1352:                                    );
1353:                                }

1355:                                if (price < 0.947076e6) {
1356:                                    return PriceData(
1357:                                        0.947076e6,
1358:                                        0.92000000000000004e18,
1359:                                        1.082198372170484424e18,
1360:                                        0.942795e6,
1361:                                        0.913517247483640937e18,
1362:                                        1.089058909134983155e18,
1363:                                        1e18
1364:                                    );
1365:                                } else {	// @audit-issue
1366:                                    return PriceData(
1367:                                        0.948888e6,
1368:                                        0.922744694427920065e18,
1369:                                        1.079303068129318754e18,
1370:                                        0.947076e6,
1371:                                        0.92000000000000004e18,
1372:                                        1.082198372170484424e18,
1373:                                        1e18
1374:                                    );
1375:                                }

1381:                                if (price < 0.955039e6) {
1382:                                    return PriceData(
1383:                                        0.955039e6,
1384:                                        0.932065347906990027e18,
1385:                                        1.069512051592246715e18,
1386:                                        0.948888e6,
1387:                                        0.922744694427920065e18,
1388:                                        1.079303068129318754e18,
1389:                                        1e18
1390:                                    );
1391:                                } else {	// @audit-issue
1392:                                    return PriceData(
1393:                                        0.961249e6,
1394:                                        0.941480149400999999e18,
1395:                                        1.059685929936267312e18,
1396:                                        0.955039e6,
1397:                                        0.932065347906990027e18,
1398:                                        1.069512051592246715e18,
1399:                                        1e18
1400:                                    );
1401:                                }

1403:                                if (price < 0.967525e6) {
1404:                                    return PriceData(
1405:                                        0.967525e6,
1406:                                        0.950990049900000023e18,
1407:                                        1.049824804368118425e18,
1408:                                        0.961249e6,
1409:                                        0.941480149400999999e18,
1410:                                        1.059685929936267312e18,
1411:                                        1e18
1412:                                    );
1413:                                } else {	// @audit-issue
1414:                                    return PriceData(
1415:                                        0.973868e6,
1416:                                        0.960596010000000056e18,
1417:                                        1.039928808315135234e18,
1418:                                        0.967525e6,
1419:                                        0.950990049900000023e18,
1420:                                        1.049824804368118425e18,
1421:                                        1e18
1422:                                    );
1423:                                }

1427:                                if (price < 0.980283e6) {
1428:                                    return PriceData(
1429:                                        0.980283e6,
1430:                                        0.970299000000000134e18,
1431:                                        1.029998108905910481e18,
1432:                                        0.973868e6,
1433:                                        0.960596010000000056e18,
1434:                                        1.039928808315135234e18,
1435:                                        1e18
1436:                                    );
1437:                                } else {	// @audit-issue
1438:                                    return PriceData(
1439:                                        0.986773e6,
1440:                                        0.980099999999999971e18,
1441:                                        1.020032908506394831e18,
1442:                                        0.980283e6,
1443:                                        0.970299000000000134e18,
1444:                                        1.029998108905910481e18,
1445:                                        1e18
1446:                                    );
1447:                                }

1469:                                if (price < 1.006679e6) {
1470:                                    return PriceData(
1471:                                        1.006679e6,
1472:                                        1.010000000000000009e18,
1473:                                        0.990033224058159078e18,
1474:                                        0.993344e6,
1475:                                        0.989999999999999991e18,
1476:                                        1.01003344631248293e18,
1477:                                        1e18
1478:                                    );
1479:                                } else {	// @audit-issue
1480:                                    return PriceData(
1481:                                        1.01345e6,
1482:                                        1.020100000000000007e18,
1483:                                        0.980033797419900599e18,
1484:                                        1.006679e6,
1485:                                        1.010000000000000009e18,
1486:                                        0.990033224058159078e18,
1487:                                        1e18
1488:                                    );
1489:                                }

1491:                                if (price < 1.020319e6) {
1492:                                    return PriceData(
1493:                                        1.020319e6,
1494:                                        1.030300999999999911e18,
1495:                                        0.970002111104709575e18,
1496:                                        1.01345e6,
1497:                                        1.020100000000000007e18,
1498:                                        0.980033797419900599e18,
1499:                                        1e18
1500:                                    );
1501:                                } else {	// @audit-issue
1502:                                    return PriceData(
1503:                                        1.027293e6,
1504:                                        1.040604010000000024e18,
1505:                                        0.959938599971011053e18,
1506:                                        1.020319e6,
1507:                                        1.030300999999999911e18,
1508:                                        0.970002111104709575e18,
1509:                                        1e18
1510:                                    );
1511:                                }

1515:                                if (price < 1.033686e6) {
1516:                                    return PriceData(
1517:                                        1.033686e6,
1518:                                        1.050000000000000044e18,
1519:                                        0.950820553711780869e18,
1520:                                        1.027293e6,
1521:                                        1.040604010000000024e18,
1522:                                        0.959938599971011053e18,
1523:                                        1e18
1524:                                    );
1525:                                } else {	// @audit-issue
1526:                                    return PriceData(
1527:                                        1.034375e6,
1528:                                        1.051010050100000148e18,
1529:                                        0.949843744564435544e18,
1530:                                        1.033686e6,
1531:                                        1.050000000000000044e18,
1532:                                        0.950820553711780869e18,
1533:                                        1e18
1534:                                    );
1535:                                }

1537:                                if (price < 1.041574e6) {
1538:                                    return PriceData(
1539:                                        1.041574e6,
1540:                                        1.061520150601000134e18,
1541:                                        0.93971807302139454e18,
1542:                                        1.034375e6,
1543:                                        1.051010050100000148e18,
1544:                                        0.949843744564435544e18,
1545:                                        1e18
1546:                                    );
1547:                                } else {	// @audit-issue
1548:                                    return PriceData(
1549:                                        1.048893e6,
1550:                                        1.072135352107010053e18,
1551:                                        0.929562163027227939e18,
1552:                                        1.041574e6,
1553:                                        1.061520150601000134e18,
1554:                                        0.93971807302139454e18,
1555:                                        1e18
1556:                                    );
1557:                                }

1563:                                if (price < 1.056342e6) {
1564:                                    return PriceData(
1565:                                        1.056342e6,
1566:                                        1.082856705628080007e18,
1567:                                        0.919376643827810258e18,
1568:                                        1.048893e6,
1569:                                        1.072135352107010053e18,
1570:                                        0.929562163027227939e18,
1571:                                        1e18
1572:                                    );
1573:                                } else {	// @audit-issue
1574:                                    return PriceData(
1575:                                        1.063925e6,
1576:                                        1.093685272684360887e18,
1577:                                        0.90916219829307332e18,
1578:                                        1.056342e6,
1579:                                        1.082856705628080007e18,
1580:                                        0.919376643827810258e18,
1581:                                        1e18
1582:                                    );
1583:                                }

1585:                                if (price < 1.070147e6) {
1586:                                    return PriceData(
1587:                                        1.070147e6,
1588:                                        1.102500000000000036e18,
1589:                                        0.900901195775543062e18,
1590:                                        1.063925e6,
1591:                                        1.093685272684360887e18,
1592:                                        0.90916219829307332e18,
1593:                                        1e18
1594:                                    );
1595:                                } else {	// @audit-issue
1596:                                    return PriceData(
1597:                                        1.071652e6,
1598:                                        1.104622125411204525e18,
1599:                                        0.89891956503043724e18,
1600:                                        1.070147e6,
1601:                                        1.102500000000000036e18,
1602:                                        0.900901195775543062e18,
1603:                                        1e18
1604:                                    );
1605:                                }

1609:                                if (price < 1.079529e6) {
1610:                                    return PriceData(
1611:                                        1.079529e6,
1612:                                        1.115668346665316557e18,
1613:                                        0.888649540545595529e18,
1614:                                        1.071652e6,
1615:                                        1.104622125411204525e18,
1616:                                        0.89891956503043724e18,
1617:                                        1e18
1618:                                    );
1619:                                } else {	// @audit-issue
1620:                                    return PriceData(
1621:                                        1.087566e6,
1622:                                        1.126825030131969774e18,
1623:                                        0.878352981447521719e18,
1624:                                        1.079529e6,
1625:                                        1.115668346665316557e18,
1626:                                        0.888649540545595529e18,
1627:                                        1e18
1628:                                    );
1629:                                }

1647:                                if (price < 1.104151e6) {
1648:                                    return PriceData(
1649:                                        1.104151e6,
1650:                                        1.149474213237622333e18,
1651:                                        0.857683999872391523e18,
1652:                                        1.09577e6,
1653:                                        1.1380932804332895e18,
1654:                                        0.868030806693890433e18,
1655:                                        1e18
1656:                                    );
1657:                                } else {	// @audit-issue
1658:                                    return PriceData(
1659:                                        1.110215e6,
1660:                                        1.157625000000000126e18,
1661:                                        0.850322213751246947e18,
1662:                                        1.104151e6,
1663:                                        1.149474213237622333e18,
1664:                                        0.857683999872391523e18,
1665:                                        1e18
1666:                                    );
1667:                                }

1669:                                if (price < 1.112718e6) {
1670:                                    return PriceData(
1671:                                        1.112718e6,
1672:                                        1.160968955369998667e18,
1673:                                        0.847313611512600207e18,
1674:                                        1.110215e6,
1675:                                        1.157625000000000126e18,
1676:                                        0.850322213751246947e18,
1677:                                        1e18
1678:                                    );
1679:                                } else {	// @audit-issue
1680:                                    return PriceData(
1681:                                        1.121482e6,
1682:                                        1.172578644923698565e18,
1683:                                        0.836920761422192294e18,
1684:                                        1.112718e6,
1685:                                        1.160968955369998667e18,
1686:                                        0.847313611512600207e18,
1687:                                        1e18
1688:                                    );
1689:                                }

1693:                                if (price < 1.130452e6) {
1694:                                    return PriceData(
1695:                                        1.130452e6,
1696:                                        1.184304431372935618e18,
1697:                                        0.826506641040327228e18,
1698:                                        1.121482e6,
1699:                                        1.172578644923698565e18,
1700:                                        0.836920761422192294e18,
1701:                                        1e18
1702:                                    );
1703:                                } else {	// @audit-issue
1704:                                    return PriceData(
1705:                                        1.139642e6,
1706:                                        1.196147475686665018e18,
1707:                                        0.8160725157999702e18,
1708:                                        1.130452e6,
1709:                                        1.184304431372935618e18,
1710:                                        0.826506641040327228e18,
1711:                                        1e18
1712:                                    );
1713:                                }

1715:                                if (price < 1.149062e6) {
1716:                                    return PriceData(
1717:                                        1.149062e6,
1718:                                        1.208108950443531393e18,
1719:                                        0.805619727489791271e18,
1720:                                        1.139642e6,
1721:                                        1.196147475686665018e18,
1722:                                        0.8160725157999702e18,
1723:                                        1e18
1724:                                    );
1725:                                } else {	// @audit-issue
1726:                                    return PriceData(
1727:                                        1.15496e6,
1728:                                        1.21550625000000001e18,
1729:                                        0.799198479643147719e18,
1730:                                        1.149062e6,
1731:                                        1.208108950443531393e18,
1732:                                        0.805619727489791271e18,
1733:                                        1e18
1734:                                    );
1735:                                }

1741:                                if (price < 1.158725e6) {
1742:                                    return PriceData(
1743:                                        1.158725e6,
1744:                                        1.22019003994796682e18,
1745:                                        0.795149696605042422e18,
1746:                                        1.15496e6,
1747:                                        1.21550625000000001e18,
1748:                                        0.799198479643147719e18,
1749:                                        1e18
1750:                                    );
1751:                                } else {	// @audit-issue
1752:                                    return PriceData(
1753:                                        1.168643e6,
1754:                                        1.232391940347446369e18,
1755:                                        0.784663924675502389e18,
1756:                                        1.158725e6,
1757:                                        1.22019003994796682e18,
1758:                                        0.795149696605042422e18,
1759:                                        1e18
1760:                                    );
1761:                                }

1763:                                if (price < 1.178832e6) {
1764:                                    return PriceData(
1765:                                        1.178832e6,
1766:                                        1.244715859750920917e18,
1767:                                        0.774163996557160172e18,
1768:                                        1.168643e6,
1769:                                        1.232391940347446369e18,
1770:                                        0.784663924675502389e18,
1771:                                        1e18
1772:                                    );
1773:                                } else {	// @audit-issue
1774:                                    return PriceData(
1775:                                        1.189304e6,
1776:                                        1.257163018348430139e18,
1777:                                        0.763651582672810969e18,
1778:                                        1.178832e6,
1779:                                        1.244715859750920917e18,
1780:                                        0.774163996557160172e18,
1781:                                        1e18
1782:                                    );
1783:                                }

1787:                                if (price < 1.200076e6) {
1788:                                    return PriceData(
1789:                                        1.200076e6,
1790:                                        1.269734648531914534e18,
1791:                                        0.753128441185147435e18,
1792:                                        1.189304e6,
1793:                                        1.257163018348430139e18,
1794:                                        0.763651582672810969e18,
1795:                                        1e18
1796:                                    );
1797:                                } else {	// @audit-issue
1798:                                    return PriceData(
1799:                                        1.205768e6,
1800:                                        1.276281562499999911e18,
1801:                                        0.747685899578659385e18,
1802:                                        1.200076e6,
1803:                                        1.269734648531914534e18,
1804:                                        0.753128441185147435e18,
1805:                                        1e18
1806:                                    );
1807:                                }

1827:                                if (price < 1.222589e6) {
1828:                                    return PriceData(
1829:                                        1.222589e6,
1830:                                        1.295256314967406119e18,
1831:                                        0.732057459169776381e18,
1832:                                        1.211166e6,
1833:                                        1.282431995017233595e18,
1834:                                        0.74259642008426785e18,
1835:                                        1e18
1836:                                    );
1837:                                } else {	// @audit-issue
1838:                                    return PriceData(
1839:                                        1.234362e6,
1840:                                        1.308208878117080198e18,
1841:                                        0.721513591905860174e18,
1842:                                        1.222589e6,
1843:                                        1.295256314967406119e18,
1844:                                        0.732057459169776381e18,
1845:                                        1e18
1846:                                    );
1847:                                }

1849:                                if (price < 1.246507e6) {
1850:                                    return PriceData(
1851:                                        1.246507e6,
1852:                                        1.321290966898250874e18,
1853:                                        0.710966947125877935e18,
1854:                                        1.234362e6,
1855:                                        1.308208878117080198e18,
1856:                                        0.721513591905860174e18,
1857:                                        1e18
1858:                                    );
1859:                                } else {	// @audit-issue
1860:                                    return PriceData(
1861:                                        1.259043e6,
1862:                                        1.33450387656723346e18,
1863:                                        0.700419750561125598e18,
1864:                                        1.246507e6,
1865:                                        1.321290966898250874e18,
1866:                                        0.710966947125877935e18,
1867:                                        1e18
1868:                                    );
1869:                                }

1873:                                if (price < 1.264433e6) {
1874:                                    return PriceData(
1875:                                        1.264433e6,
1876:                                        1.340095640624999973e18,
1877:                                        0.695987932996588454e18,
1878:                                        1.259043e6,
1879:                                        1.33450387656723346e18,
1880:                                        0.700419750561125598e18,
1881:                                        1e18
1882:                                    );
1883:                                } else {	// @audit-issue
1884:                                    return PriceData(
1885:                                        1.271991e6,
1886:                                        1.347848915332905628e18,
1887:                                        0.689874326166576179e18,
1888:                                        1.264433e6,
1889:                                        1.340095640624999973e18,
1890:                                        0.695987932996588454e18,
1891:                                        1e18
1892:                                    );
1893:                                }

1895:                                if (price < 1.285375e6) {
1896:                                    return PriceData(
1897:                                        1.285375e6,
1898:                                        1.361327404486234682e18,
1899:                                        0.67933309721453039e18,
1900:                                        1.271991e6,
1901:                                        1.347848915332905628e18,
1902:                                        0.689874326166576179e18,
1903:                                        1e18
1904:                                    );
1905:                                } else {	// @audit-issue
1906:                                    return PriceData(
1907:                                        1.299217e6,
1908:                                        1.374940678531097138e18,
1909:                                        0.668798587125333244e18,
1910:                                        1.285375e6,
1911:                                        1.361327404486234682e18,
1912:                                        0.67933309721453039e18,
1913:                                        1e18
1914:                                    );
1915:                                }

1921:                                if (price < 1.313542e6) {
1922:                                    return PriceData(
1923:                                        1.313542e6,
1924:                                        1.38869008531640814e18,
1925:                                        0.658273420002602916e18,
1926:                                        1.299217e6,
1927:                                        1.374940678531097138e18,
1928:                                        0.668798587125333244e18,
1929:                                        1e18
1930:                                    );
1931:                                } else {	// @audit-issue
1932:                                    return PriceData(
1933:                                        1.328377e6,
1934:                                        1.402576986169572049e18,
1935:                                        0.647760320838866033e18,
1936:                                        1.313542e6,
1937:                                        1.38869008531640814e18,
1938:                                        0.658273420002602916e18,
1939:                                        1e18
1940:                                    );
1941:                                }

1943:                                if (price < 1.333292e6) {
1944:                                    return PriceData(
1945:                                        1.333292e6,
1946:                                        1.407100422656250016e18,
1947:                                        0.644361360672887962e18,
1948:                                        1.328377e6,
1949:                                        1.402576986169572049e18,
1950:                                        0.647760320838866033e18,
1951:                                        1e18
1952:                                    );
1953:                                } else {	// @audit-issue
1954:                                    return PriceData(
1955:                                        1.343751e6,
1956:                                        1.416602756031267951e18,
1957:                                        0.637262115356114656e18,
1958:                                        1.333292e6,
1959:                                        1.407100422656250016e18,
1960:                                        0.644361360672887962e18,
1961:                                        1e18
1962:                                    );
1963:                                }

1967:                                if (price < 1.359692e6) {
1968:                                    return PriceData(
1969:                                        1.359692e6,
1970:                                        1.430768783591580551e18,
1971:                                        0.626781729444674585e18,
1972:                                        1.343751e6,
1973:                                        1.416602756031267951e18,
1974:                                        0.637262115356114656e18,
1975:                                        1e18
1976:                                    );
1977:                                } else {	// @audit-issue
1978:                                    return PriceData(
1979:                                        1.376232e6,
1980:                                        1.445076471427496179e18,
1981:                                        0.616322188162944262e18,
1982:                                        1.359692e6,
1983:                                        1.430768783591580551e18,
1984:                                        0.626781729444674585e18,
1985:                                        1e18
1986:                                    );
1987:                                }

2005:                                if (price < 1.41124e6) {
2006:                                    return PriceData(
2007:                                        1.41124e6,
2008:                                        1.474122508503188822e18,
2009:                                        0.595478226183906334e18,
2010:                                        1.393403e6,
2011:                                        1.459527236141771489e18,
2012:                                        0.605886614260108591e18,
2013:                                        1e18
2014:                                    );
2015:                                } else {	// @audit-issue
2016:                                    return PriceData(
2017:                                        1.415386e6,
2018:                                        1.47745544378906235e18,
2019:                                        0.593119977480511928e18,
2020:                                        1.41124e6,
2021:                                        1.474122508503188822e18,
2022:                                        0.595478226183906334e18,
2023:                                        1e18
2024:                                    );
2025:                                }

2027:                                if (price < 1.42978e6) {
2028:                                    return PriceData(
2029:                                        1.42978e6,
2030:                                        1.488863733588220883e18,
2031:                                        0.585100335536025584e18,
2032:                                        1.415386e6,
2033:                                        1.47745544378906235e18,
2034:                                        0.593119977480511928e18,
2035:                                        1e18
2036:                                    );
2037:                                } else {	// @audit-issue
2038:                                    return PriceData(
2039:                                        1.514667e6,
2040:                                        1.551328215978515557e18,
2041:                                        0.54263432113736132e18,
2042:                                        1.42978e6,
2043:                                        1.488863733588220883e18,
2044:                                        0.585100335536025584e18,
2045:                                        1e18
2046:                                    );
2047:                                }

2051:                                if (price < 1.636249e6) {
2052:                                    return PriceData(
2053:                                        1.636249e6,
2054:                                        1.628894626777441568e18,
2055:                                        0.493325115988533236e18,
2056:                                        1.514667e6,
2057:                                        1.551328215978515557e18,
2058:                                        0.54263432113736132e18,
2059:                                        1e18
2060:                                    );
2061:                                } else {	// @audit-issue
2062:                                    return PriceData(
2063:                                        1.786708e6,
2064:                                        1.710339358116313546e18,
2065:                                        0.445648172809785581e18,
2066:                                        1.636249e6,
2067:                                        1.628894626777441568e18,
2068:                                        0.493325115988533236e18,
2069:                                        1e18
2070:                                    );
2071:                                }

2073:                                if (price < 1.974398e6) {
2074:                                    return PriceData(
2075:                                        1.974398e6,
2076:                                        1.79585632602212919e18,
2077:                                        0.400069510798421513e18,
2078:                                        1.786708e6,
2079:                                        1.710339358116313546e18,
2080:                                        0.445648172809785581e18,
2081:                                        1e18
2082:                                    );
2083:                                } else {	// @audit-issue
2084:                                    return PriceData(
2085:                                        2.209802e6,
2086:                                        1.885649142323235772e18,
2087:                                        0.357031765135700119e18,
2088:                                        1.974398e6,
2089:                                        1.79585632602212919e18,
2090:                                        0.400069510798421513e18,
2091:                                        1e18
2092:                                    );
2093:                                }

2099:                                if (price < 2.505865e6) {
2100:                                    return PriceData(
2101:                                        2.505865e6,
2102:                                        1.97993159943939756e18,
2103:                                        0.316916199929126341e18,
2104:                                        2.209802e6,
2105:                                        1.885649142323235772e18,
2106:                                        0.357031765135700119e18,
2107:                                        1e18
2108:                                    );
2109:                                } else {	// @audit-issue
2110:                                    return PriceData(
2111:                                        2.878327e6,
2112:                                        2.078928179411367427e18,
2113:                                        0.28000760254479623e18,
2114:                                        2.505865e6,
2115:                                        1.97993159943939756e18,
2116:                                        0.316916199929126341e18,
2117:                                        1e18
2118:                                    );
2119:                                }

2121:                                if (price < 3.346057e6) {
2122:                                    return PriceData(
2123:                                        3.346057e6,
2124:                                        2.182874588381935599e18,
2125:                                        0.246470170347584949e18,
2126:                                        2.878327e6,
2127:                                        2.078928179411367427e18,
2128:                                        0.28000760254479623e18,
2129:                                        1e18
2130:                                    );
2131:                                } else {	// @audit-issue
2132:                                    return PriceData(
2133:                                        3.931396e6,
2134:                                        2.292018317801032268e18,
2135:                                        0.216340086006769544e18,
2136:                                        3.346057e6,
2137:                                        2.182874588381935599e18,
2138:                                        0.246470170347584949e18,
2139:                                        1e18
2140:                                    );
2141:                                }

2145:                                if (price < 4.660591e6) {
2146:                                    return PriceData(
2147:                                        4.660591e6,
2148:                                        2.406619233691083881e18,
2149:                                        0.189535571483960663e18,
2150:                                        3.931396e6,
2151:                                        2.292018317801032268e18,
2152:                                        0.216340086006769544e18,
2153:                                        1e18
2154:                                    );
2155:                                } else {	// @audit-issue
2156:                                    return PriceData(
2157:                                        10.709509e6,
2158:                                        3e18,
2159:                                        0.103912563829966526e18,
2160:                                        4.660591e6,
2161:                                        2.406619233691083881e18,
2162:                                        0.189535571483960663e18,
2163:                                        1e18
2164:                                    );
2165:                                }
```
[57](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L47-L67), [87](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L77-L97), [108](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L104-L112), [129](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L119-L139), [150](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L146-L154), [169](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L159-L179), [190](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L186-L194), [213](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L203-L223), [241](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L231-L251), [271](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L261-L281), [292](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L288-L296), [313](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L303-L323), [334](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L330-L338), [353](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L343-L363), [374](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L370-L378), [399](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L389-L409), [427](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L417-L437), [457](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L447-L467), [478](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L474-L482), [499](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L489-L509), [520](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L516-L524), [539](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L529-L549), [560](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L556-L564), [583](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L573-L593), [604](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L600-L608), [623](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L613-L633), [644](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L640-L648), [665](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L655-L675), [686](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L682-L690), [705](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L695-L715), [726](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L722-L728), [771](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L761-L781), [795](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L785-L805), [817](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L807-L827), [843](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L833-L853), [865](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L855-L875), [889](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L879-L899), [911](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L901-L921), [939](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L929-L949), [961](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L951-L971), [985](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L975-L995), [1007](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L997-L1017), [1033](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1023-L1043), [1055](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1045-L1065), [1079](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1069-L1089), [1119](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1109-L1129), [1141](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1131-L1151), [1165](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1155-L1175), [1187](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1177-L1197), [1213](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1203-L1223), [1235](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1225-L1245), [1259](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1249-L1269), [1297](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1287-L1307), [1319](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1309-L1329), [1343](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1333-L1353), [1365](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1355-L1375), [1391](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1381-L1401), [1413](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1403-L1423), [1437](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1427-L1447), [1479](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1469-L1489), [1501](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1491-L1511), [1525](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1515-L1535), [1547](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1537-L1557), [1573](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1563-L1583), [1595](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1585-L1605), [1619](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1609-L1629), [1657](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1647-L1667), [1679](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1669-L1689), [1703](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1693-L1713), [1725](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1715-L1735), [1751](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1741-L1761), [1773](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1763-L1783), [1797](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1787-L1807), [1837](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1827-L1847), [1859](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1849-L1869), [1883](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1873-L1893), [1905](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1895-L1915), [1931](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1921-L1941), [1953](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1943-L1963), [1977](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1967-L1987), [2015](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L2005-L2025), [2037](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L2027-L2047), [2061](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L2051-L2071), [2083](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L2073-L2093), [2109](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L2099-L2119), [2131](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L2121-L2141), [2155](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L2145-L2165), 


#### Recommendation

Consider simplifying code by removing unnecessary `else` blocks when the `if` block returns. You can achieve cleaner and more concise code by directly returning a value after the `if` block instead of using an `else` block for a subsequent return statement.

### Unused import
The identifier is imported but never used within the file.

```solidity
Path: ./src/WellUpgradeable.sol

8:import {IERC20, SafeERC20} from "oz/token/ERC20/utils/SafeERC20.sol";	// @audit-issue: SafeERC20 not used in the contract.
```
[8](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L8-L8), 


```solidity
Path: ./src/functions/Stable2.sol

5:import {IBeanstalkWellFunction, IMultiFlowPumpWellFunction} from "src/interfaces/IBeanstalkWellFunction.sol";	// @audit-issue: IMultiFlowPumpWellFunction not used in the contract.

8:import {IERC20} from "forge-std/interfaces/IERC20.sol";	// @audit-issue: IERC20 not used in the contract.
```
[5](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L5-L5), [8](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L8-L8), 


#### Recommendation

Regularly review your Solidity code to remove unused imports. This practice declutters the codebase, making it easier to understand and maintain. In addition, consider using tools or IDE features that can automatically detect and highlight unused imports for cleanup. Keeping imports limited to only what is necessary helps maintain a clear understanding of the contract's dependencies and reduces potential confusion for developers working on or reviewing the code.

### `if`-statement can be converted to a ternary
The code can be made more compact while also increasing readability by converting the following `if`-statements to ternaries (e.g. `foo += (x > y) ? a : b`)

```solidity
Path: ./src/functions/Stable2.sol

214:        if (pd.lutData.lowPriceJ > pd.lutData.highPriceJ) {	// @audit-issue
215:            pd.maxStepSize = scaledReserves[j] * (pd.lutData.lowPriceJ - pd.lutData.highPriceJ) / pd.lutData.lowPriceJ;
216:        } else {
217:            pd.maxStepSize = scaledReserves[j] * (pd.lutData.highPriceJ - pd.lutData.lowPriceJ) / pd.lutData.highPriceJ;
218:        }

282:        if (pd.lutData.lowPriceJ > pd.lutData.highPriceJ) {	// @audit-issue
283:            pd.maxStepSize = scaledReserves[j] * (pd.lutData.lowPriceJ - pd.lutData.highPriceJ) / pd.lutData.lowPriceJ;
284:        } else {
285:            pd.maxStepSize = scaledReserves[j] * (pd.lutData.highPriceJ - pd.lutData.lowPriceJ) / pd.lutData.highPriceJ;
286:        }

314:        if (decimal0 == 0) {	// @audit-issue
315:            decimal0 = 18;
316:        }

317:        if (decimal0 == 0) {	// @audit-issue
318:            decimal1 = 18;
319:        }

388:        if (pd.targetPrice > pd.currentPrice) {	// @audit-issue
389:            // if the targetPrice is greater than the currentPrice,
390:            // the reserve needs to be decremented to increase currentPrice.
391:            return reserve
392:                - pd.maxStepSize * (pd.targetPrice - pd.currentPrice) / (pd.lutData.highPrice - pd.lutData.lowPrice);
393:        } else {
394:            // if the targetPrice is less than the currentPrice,
395:            // the reserve needs to be incremented to decrease currentPrice.
396:            return reserve
397:                + pd.maxStepSize * (pd.currentPrice - pd.targetPrice) / (pd.lutData.highPrice - pd.lutData.lowPrice);
398:        }
```
[214](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L214-L218), [282](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L282-L286), [314](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L314-L316), [317](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L317-L319), [388](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L388-L398), 


#### Recommendation

Consider using single line if statements as ternary.

### Contract uses both `require()`/`revert()` as well as custom errors
Consider using just one method in a single file

```solidity
Path: ./src/WellUpgradeable.sol

16:contract WellUpgradeable is Well, UUPSUpgradeable, OwnableUpgradeable {	// @audit-issue
```
[16](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L16-L16), 


```solidity
Path: ./src/functions/Stable2.sol

25:contract Stable2 is ProportionalLPToken2, IBeanstalkWellFunction {	// @audit-issue
```
[25](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L25-L25), 


#### Recommendation

Consistently use either `require()`/`revert()` or custom errors for error handling in your contract to maintain code consistency and readability. Using both methods in the same contract can lead to confusion and make the code harder to understand.

### Custom error has no error details
Consider adding parameters to the error to indicate which user or values caused the failure

```solidity
Path: ./src/functions/Stable2.sol

50:    error InvalidTokenDecimals();	// @audit-issue

51:    error InvalidLUT();	// @audit-issue
```
[50](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L50-L50), [51](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L51-L51), 


#### Recommendation

When defining custom errors, consider adding parameters or error details that provide information about the specific conditions or inputs that caused the error. Including error details can make debugging and troubleshooting easier by providing context on the cause of the failure.

### Constants in comparisons should appear on the left side
Doing so will prevent [typo bugs](https://www.moserware.com/2008/01/constants-on-left-are-better-but-this.html)

```solidity
Path: ./src/functions/Stable2.sol

78:        if (reserves[0] == 0 && reserves[1] == 0) return 0;	// @audit-issue

86:        if (sumReserves == 0) return 0;	// @audit-issue

124:        (uint256 c, uint256 b) = getBandC(a * N * N, lpTokenSupply, j == 0 ? scaledReserves[1] : scaledReserves[0]);	// @audit-issue

179:        uint256 i = j == 1 ? 0 : 1;	// @audit-issue

252:        uint256 i = j == 1 ? 0 : 1;	// @audit-issue

314:        if (decimal0 == 0) {	// @audit-issue

317:        if (decimal0 == 0) {	// @audit-issue
```
[78](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L78-L78), [86](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L86-L86), [124](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L124-L124), [179](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L179-L179), [252](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L252-L252), [314](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L314-L314), [317](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L317-L317), 


#### Recommendation

To prevent typo bugs and improve code readability, it's advisable to place constants on the left side of comparisons. This coding practice helps catch accidental assignment (=) instead of comparison (==) and enhances code quality.

### Convert duplicated codes to modifier/functions
Duplicated code in Solidity contracts, especially when repeated across multiple functions, can lead to inefficiencies and increased maintenance challenges. It not only bloats the contract but also makes it harder to implement changes consistently across the codebase. By identifying common patterns or checks that are repeated and abstracting them into modifiers or separate functions, the code can be made more concise, readable, and maintainable. This practice not only reduces the overall bytecode size, potentially lowering deployment and execution costs, but also enhances the contract's reliability by ensuring consistency in how these common checks or operations are executed.

```solidity
Path: ./src/functions/Stable2.sol

252:        uint256 i = j == 1 ? 0 : 1;	// @audit-issue: Exactly same copy pasted functionality between lines: `{'179->187'}`
253:        // scale reserves and ratios:
254:        uint256[] memory decimals = decodeWellData(data);
255:        uint256[] memory scaledReserves = getScaledReserves(reserves, decimals);
256:
257:        PriceData memory pd;
258:        uint256[] memory scaledRatios = getScaledReserves(ratios, decimals);
259:        // calc target price with 6 decimal precision:
260:        pd.targetPrice = scaledRatios[i] * PRICE_PRECISION / scaledRatios[j];

214:        if (pd.lutData.lowPriceJ > pd.lutData.highPriceJ) {	// @audit-issue: Same if statement in line(s): ['282']

229:            if (pd.currentPrice > pd.targetPrice) {	// @audit-issue: Same if statement in line(s): ['294']

230:                if (pd.currentPrice - pd.targetPrice <= PRICE_THRESHOLD) {	// @audit-issue: Same if statement in line(s): ['295']

234:                if (pd.targetPrice - pd.currentPrice <= PRICE_THRESHOLD) {	// @audit-issue: Same if statement in line(s): ['299']
```
[252](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L252-L260), [214](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L214-L214), [229](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L229-L229), [230](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L230-L230), [234](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L234-L234), 


#### Recommendation

Identify and consolidate duplicated code blocks in Solidity contracts into reusable modifiers or functions. This approach streamlines the contract by eliminating redundancy, thereby improving clarity, maintainability, and potentially reducing gas costs. For common checks, consider using modifiers for concise and consistent enforcement of conditions. For reusable logic, encapsulate it in functions to avoid code duplication and simplify future updates or bug fixes.

### Style guide: Function ordering does not follow the Solidity style guide
According to the Solidity [style guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions), functions should be laid out in the following order :`constructor()`, `receive()`, `fallback()`, `external`, `public`, `internal`, `private`, but the cases below do not follow this pattern

```solidity
Path: ./src/WellUpgradeable.sol

93:    function upgradeTo(address newImplementation) public override {	// @audit-issue
```
[93](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L93-L93), 


```solidity
Path: ./src/functions/Stable2.sol

173:    function calcReserveAtRatioSwap(	// @audit-issue
```
[173](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L173-L173), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/latest/style-guide.html).

### Duplicate import statements
Contracts often depend on libraries and other contracts to modularize code and reuse functionalities. However, redundant imports occur when a contract imports a library or another contract that is already imported by one of its dependencies. This redundancy does not impact the compiled bytecode but can clutter the codebase, making it harder to understand the direct dependencies of each contract. Simplifying imports by removing these redundancies enhances code readability and maintainability.

```solidity
Path: ./src/WellUpgradeable.sol

8:import {IERC20, SafeERC20} from "oz/token/ERC20/utils/SafeERC20.sol";	// @audit-issue: Same library is also imported on: `['Well', 'IAquifer']`, at lines: `[7, 5]`
```
[8](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L8-L8), 


#### Recommendation

Review your Solidity contracts and eliminate duplicate import statements. Ensure each file is imported only once where it's needed. If you find the same import statement across multiple files, consider whether you can restructure your code to centralize common logic or dependencies, potentially through base contracts or libraries. Regularly conduct code audits or utilize linters and other static analysis tools to identify and resolve duplicate imports, thereby enhancing the clarity, structure, and security of your Solidity codebase.

### Adding a return statement when the function defines a named return variable, is redundant
Once the return variable has been assigned (or has its default value), there is no need to explicitly return it at the end of the function, since it's returned automatically.

```solidity
Path: ./src/functions/Stable2.sol

74:    function calcLpTokenSupply(
75:        uint256[] memory reserves,
76:        bytes memory data
77:    ) public view returns (uint256 lpTokenSupply) {
78:        if (reserves[0] == 0 && reserves[1] == 0) return 0;	// @audit-issue
79:        uint256[] memory decimals = decodeWellData(data);
80:        // scale reserves to 18 decimals.
81:        uint256[] memory scaledReserves = getScaledReserves(reserves, decimals);
82:
83:        uint256 Ann = a * N * N;
84:
85:        uint256 sumReserves = scaledReserves[0] + scaledReserves[1];
86:        if (sumReserves == 0) return 0;
87:        lpTokenSupply = sumReserves;
88:        for (uint256 i = 0; i < 255; i++) {
89:            uint256 dP = lpTokenSupply;
90:            // If division by 0, this will be borked: only withdrawal will work. And that is good
91:            dP = dP * lpTokenSupply / (scaledReserves[0] * N);
92:            dP = dP * lpTokenSupply / (scaledReserves[1] * N);
93:            uint256 prevReserves = lpTokenSupply;
94:            lpTokenSupply = (Ann * sumReserves / A_PRECISION + (dP * N)) * lpTokenSupply
95:                / (((Ann - A_PRECISION) * lpTokenSupply / A_PRECISION) + ((N + 1) * dP));
96:            // Equality with the precision of 1
97:            if (lpTokenSupply > prevReserves) {
98:                if (lpTokenSupply - prevReserves <= 1) return lpTokenSupply;
99:            } else {
100:                if (prevReserves - lpTokenSupply <= 1) return lpTokenSupply;
101:            }
102:        }
103:    }

74:    function calcLpTokenSupply(
75:        uint256[] memory reserves,
76:        bytes memory data
77:    ) public view returns (uint256 lpTokenSupply) {
78:        if (reserves[0] == 0 && reserves[1] == 0) return 0;
79:        uint256[] memory decimals = decodeWellData(data);
80:        // scale reserves to 18 decimals.
81:        uint256[] memory scaledReserves = getScaledReserves(reserves, decimals);
82:
83:        uint256 Ann = a * N * N;
84:
85:        uint256 sumReserves = scaledReserves[0] + scaledReserves[1];
86:        if (sumReserves == 0) return 0;	// @audit-issue
87:        lpTokenSupply = sumReserves;
88:        for (uint256 i = 0; i < 255; i++) {
89:            uint256 dP = lpTokenSupply;
90:            // If division by 0, this will be borked: only withdrawal will work. And that is good
91:            dP = dP * lpTokenSupply / (scaledReserves[0] * N);
92:            dP = dP * lpTokenSupply / (scaledReserves[1] * N);
93:            uint256 prevReserves = lpTokenSupply;
94:            lpTokenSupply = (Ann * sumReserves / A_PRECISION + (dP * N)) * lpTokenSupply
95:                / (((Ann - A_PRECISION) * lpTokenSupply / A_PRECISION) + ((N + 1) * dP));
96:            // Equality with the precision of 1
97:            if (lpTokenSupply > prevReserves) {
98:                if (lpTokenSupply - prevReserves <= 1) return lpTokenSupply;
99:            } else {
100:                if (prevReserves - lpTokenSupply <= 1) return lpTokenSupply;
101:            }
102:        }
103:    }

74:    function calcLpTokenSupply(
75:        uint256[] memory reserves,
76:        bytes memory data
77:    ) public view returns (uint256 lpTokenSupply) {
78:        if (reserves[0] == 0 && reserves[1] == 0) return 0;
79:        uint256[] memory decimals = decodeWellData(data);
80:        // scale reserves to 18 decimals.
81:        uint256[] memory scaledReserves = getScaledReserves(reserves, decimals);
82:
83:        uint256 Ann = a * N * N;
84:
85:        uint256 sumReserves = scaledReserves[0] + scaledReserves[1];
86:        if (sumReserves == 0) return 0;
87:        lpTokenSupply = sumReserves;
88:        for (uint256 i = 0; i < 255; i++) {
89:            uint256 dP = lpTokenSupply;
90:            // If division by 0, this will be borked: only withdrawal will work. And that is good
91:            dP = dP * lpTokenSupply / (scaledReserves[0] * N);
92:            dP = dP * lpTokenSupply / (scaledReserves[1] * N);
93:            uint256 prevReserves = lpTokenSupply;
94:            lpTokenSupply = (Ann * sumReserves / A_PRECISION + (dP * N)) * lpTokenSupply
95:                / (((Ann - A_PRECISION) * lpTokenSupply / A_PRECISION) + ((N + 1) * dP));
96:            // Equality with the precision of 1
97:            if (lpTokenSupply > prevReserves) {
98:                if (lpTokenSupply - prevReserves <= 1) return lpTokenSupply;	// @audit-issue
99:            } else {
100:                if (prevReserves - lpTokenSupply <= 1) return lpTokenSupply;
101:            }
102:        }
103:    }

114:    function calcReserve(
115:        uint256[] memory reserves,
116:        uint256 j,
117:        uint256 lpTokenSupply,
118:        bytes memory data
119:    ) public view returns (uint256 reserve) {
120:        uint256[] memory decimals = decodeWellData(data);
121:        uint256[] memory scaledReserves = getScaledReserves(reserves, decimals);
122:
123:        // avoid stack too deep errors.
124:        (uint256 c, uint256 b) = getBandC(a * N * N, lpTokenSupply, j == 0 ? scaledReserves[1] : scaledReserves[0]);
125:        reserve = lpTokenSupply;
126:        uint256 prevReserve;
127:
128:        for (uint256 i; i < 255; ++i) {
129:            prevReserve = reserve;
130:            reserve = _calcReserve(reserve, b, c, lpTokenSupply);
131:            // Equality with the precision of 1
132:            // scale reserve down to original precision
133:            if (reserve > prevReserve) {
134:                if (reserve - prevReserve <= 1) {
135:                    return reserve / (10 ** (18 - decimals[j]));	// @audit-issue
136:                }
137:            } else {
138:                if (prevReserve - reserve <= 1) {
139:                    return reserve / (10 ** (18 - decimals[j]));
140:                }
141:            }
142:        }
143:        revert("did not find convergence");
144:    }

173:    function calcReserveAtRatioSwap(
174:        uint256[] memory reserves,
175:        uint256 j,
176:        uint256[] memory ratios,
177:        bytes calldata data
178:    ) external view returns (uint256 reserve) {
179:        uint256 i = j == 1 ? 0 : 1;
180:        // scale reserves and ratios:
181:        uint256[] memory decimals = decodeWellData(data);
182:        uint256[] memory scaledReserves = getScaledReserves(reserves, decimals);
183:
184:        PriceData memory pd;
185:        uint256[] memory scaledRatios = getScaledReserves(ratios, decimals);
186:        // calc target price with 6 decimal precision:
187:        pd.targetPrice = scaledRatios[i] * PRICE_PRECISION / scaledRatios[j];
188:
189:        // get ratios and price from the closest highest and lowest price from targetPrice:
190:        pd.lutData = ILookupTable(lookupTable).getRatiosFromPriceSwap(pd.targetPrice);
191:
192:        // calculate lp token supply:
193:        uint256 lpTokenSupply = calcLpTokenSupply(scaledReserves, abi.encode(18, 18));
194:
195:        // lpTokenSupply / 2 gives the reserves at parity:
196:        uint256 parityReserve = lpTokenSupply / 2;
197:
198:        // update `scaledReserves` based on whether targetPrice is closer to low or high price:
199:        if (pd.lutData.highPrice - pd.targetPrice > pd.targetPrice - pd.lutData.lowPrice) {
200:            // targetPrice is closer to lowPrice.
201:            scaledReserves[i] = parityReserve * pd.lutData.lowPriceI / pd.lutData.precision;
202:            scaledReserves[j] = parityReserve * pd.lutData.lowPriceJ / pd.lutData.precision;
203:            // initialize currentPrice:
204:            pd.currentPrice = pd.lutData.lowPrice;
205:        } else {
206:            // targetPrice is closer to highPrice.
207:            scaledReserves[i] = parityReserve * pd.lutData.highPriceI / pd.lutData.precision;
208:            scaledReserves[j] = parityReserve * pd.lutData.highPriceJ / pd.lutData.precision;
209:            // initialize currentPrice:
210:            pd.currentPrice = pd.lutData.highPrice;
211:        }
212:
213:        // calculate max step size:
214:        if (pd.lutData.lowPriceJ > pd.lutData.highPriceJ) {
215:            pd.maxStepSize = scaledReserves[j] * (pd.lutData.lowPriceJ - pd.lutData.highPriceJ) / pd.lutData.lowPriceJ;
216:        } else {
217:            pd.maxStepSize = scaledReserves[j] * (pd.lutData.highPriceJ - pd.lutData.lowPriceJ) / pd.lutData.highPriceJ;
218:        }
219:
220:        for (uint256 k; k < 255; k++) {
221:            scaledReserves[j] = updateReserve(pd, scaledReserves[j]);
222:
223:            // calculate scaledReserve[i]:
224:            scaledReserves[i] = calcReserve(scaledReserves, i, lpTokenSupply, abi.encode(18, 18));
225:            // calc currentPrice:
226:            pd.currentPrice = _calcRate(scaledReserves, i, j, lpTokenSupply);
227:
228:            // check if new price is within 1 of target price:
229:            if (pd.currentPrice > pd.targetPrice) {
230:                if (pd.currentPrice - pd.targetPrice <= PRICE_THRESHOLD) {
231:                    return scaledReserves[j] / (10 ** (18 - decimals[j]));	// @audit-issue
232:                }
233:            } else {
234:                if (pd.targetPrice - pd.currentPrice <= PRICE_THRESHOLD) {
235:                    return scaledReserves[j] / (10 ** (18 - decimals[j]));
236:                }
237:            }
238:        }
239:    }

246:    function calcReserveAtRatioLiquidity(
247:        uint256[] calldata reserves,
248:        uint256 j,
249:        uint256[] calldata ratios,
250:        bytes calldata data
251:    ) external view returns (uint256 reserve) {
252:        uint256 i = j == 1 ? 0 : 1;
253:        // scale reserves and ratios:
254:        uint256[] memory decimals = decodeWellData(data);
255:        uint256[] memory scaledReserves = getScaledReserves(reserves, decimals);
256:
257:        PriceData memory pd;
258:        uint256[] memory scaledRatios = getScaledReserves(ratios, decimals);
259:        // calc target price with 6 decimal precision:
260:        pd.targetPrice = scaledRatios[i] * PRICE_PRECISION / scaledRatios[j];
261:
262:        // get ratios and price from the closest highest and lowest price from targetPrice:
263:        pd.lutData = ILookupTable(lookupTable).getRatiosFromPriceLiquidity(pd.targetPrice);
264:
265:        // update scaledReserve[j] such that calcRate(scaledReserves, i, j) = low/high Price,
266:        // depending on which is closer to targetPrice.
267:        if (pd.lutData.highPrice - pd.targetPrice > pd.targetPrice - pd.lutData.lowPrice) {
268:            // targetPrice is closer to lowPrice.
269:            scaledReserves[j] = scaledReserves[i] * pd.lutData.lowPriceJ / pd.lutData.precision;
270:
271:            // set current price to lowPrice.
272:            pd.currentPrice = pd.lutData.lowPrice;
273:        } else {
274:            // targetPrice is closer to highPrice.
275:            scaledReserves[j] = scaledReserves[i] * pd.lutData.highPriceJ / pd.lutData.precision;
276:
277:            // set current price to highPrice.
278:            pd.currentPrice = pd.lutData.highPrice;
279:        }
280:
281:        // calculate max step size:
282:        if (pd.lutData.lowPriceJ > pd.lutData.highPriceJ) {
283:            pd.maxStepSize = scaledReserves[j] * (pd.lutData.lowPriceJ - pd.lutData.highPriceJ) / pd.lutData.lowPriceJ;
284:        } else {
285:            pd.maxStepSize = scaledReserves[j] * (pd.lutData.highPriceJ - pd.lutData.lowPriceJ) / pd.lutData.highPriceJ;
286:        }
287:
288:        for (uint256 k; k < 255; k++) {
289:            scaledReserves[j] = updateReserve(pd, scaledReserves[j]);
290:            // calculate new price from reserves:
291:            pd.currentPrice = calcRate(scaledReserves, i, j, abi.encode(18, 18));
292:
293:            // check if new price is within PRICE_THRESHOLD:
294:            if (pd.currentPrice > pd.targetPrice) {
295:                if (pd.currentPrice - pd.targetPrice <= PRICE_THRESHOLD) {
296:                    return scaledReserves[j] / (10 ** (18 - decimals[j]));	// @audit-issue
297:                }
298:            } else {
299:                if (pd.targetPrice - pd.currentPrice <= PRICE_THRESHOLD) {
300:                    return scaledReserves[j] / (10 ** (18 - decimals[j]));
301:                }
302:            }
303:        }
304:    }
```
[78](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L74-L103), [86](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L74-L103), [98](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L74-L103), [135](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L114-L144), [231](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L173-L239), [296](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L246-L304), 


#### Recommendation

When a function defines a named return variable and assigns a value to it, there is no need to add an explicit return statement at the end of the function. The named return variable will be automatically returned with its assigned value. Removing the redundant return statement can lead to cleaner and more concise code, improving readability and reducing the risk of introducing unnecessary errors.

### Remove `forge-std` import
`forge-std` is used for logging and debugging purposes and should be removed when not used for development.

When developing smart contracts in Solidity, it is common to import various external libraries to enhance functionality or streamline the development process. One such library that is often used during development is `forge-std`. However, it is important to remember to remove this import before deploying your contracts to a production environment.

The `forge-std` library provides logging and debugging functionality that can be useful during development but serves no purpose in a production setting. Keeping the `forge-std` import in your code adds unnecessary overhead, complexity, and potential security vulnerabilities.

```solidity
Path: ./src/functions/Stable2.sol

8:import {IERC20} from "forge-std/interfaces/IERC20.sol";	// @audit-issue
```
[8](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L8-L8), 


#### Recommendation

To address this issue, simply remove the `forge-std` import from your Solidity smart contract code. This library is used for logging and debugging purposes and is not needed in a production environment. Make sure to test your contract thoroughly after the removal to ensure it operates correctly.

### Defined named returns not used within function
Such instances can be replaced with unnamed returns

```solidity
Path: ./src/functions/Stable2.sol

178:    ) external view returns (uint256 reserve) {	// @audit-issue: Return param `reserve` is defined but not used in function.

251:    ) external view returns (uint256 reserve) {	// @audit-issue: Return param `reserve` is defined but not used in function.
```
[178](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L178-L178), [251](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L251-L251), 


#### Recommendation

Inspect your Solidity functions for unused named return variables. If a named return is not actively used in the function, switch to using unnamed return types. This change simplifies the function signature and avoids potential confusion over the function's return behavior. Ensure that the function's documentation and comments clearly describe the return type and purpose, maintaining clarity even with unnamed return variables. This approach contributes to cleaner, more concise function declarations in your Solidity codebase.

### Too long functions should be refactored
Functions with too many lines are difficult to understand. It is recommended to refactor complex functions into multiple shorter and easier to understand functions.


```solidity
Path: ./src/functions/Stable2.sol

173:    function calcReserveAtRatioSwap(	// @audit-issue 66 lines

246:    function calcReserveAtRatioLiquidity(	// @audit-issue 58 lines
```
[173](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L173-L173), [246](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L246-L246), 


```solidity
Path: ./src/functions/StableLUT/Stable2LUT1.sol

27:    function getRatiosFromPriceLiquidity(uint256 price) external pure returns (PriceData memory) {	// @audit-issue 707 lines

740:    function getRatiosFromPriceSwap(uint256 price) external pure returns (PriceData memory) {	// @audit-issue 1434 lines
```
[27](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L27-L27), [740](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L740-L740), 


#### Recommendation

To address this issue, refactor long and complex functions into multiple shorter and more manageable functions. This will improve code readability and maintainability, making it easier to understand and maintain your smart contract.

### Lines are too long
Usually lines in source code are limited to [80](https://softwareengineering.stackexchange.com/questions/148677/why-is-80-characters-the-standard-limit-for-code-width) characters. Today's screens are much larger so it's reasonable to stretch this in some cases. The solidity style guide recommends a maximumum line length of [120 characters](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-length), so the lines below should be split when they reach that length.        self.impact_details = 


```solidity
Path: ./src/WellUpgradeable.sol

59:    // verification is done by verifying the ERC1967 implmentation (the well address) maps to the aquifers well -> implmentation mapping.	// @audit-issue: Length of the line is: 138

62:     * @notice Check that the execution is being performed through a delegatecall call and that the execution context is	// @audit-issue: Length of the line is: 121

113:     * IMPORTANT: A proxy pointing at a proxiable contract should not be considered proxiable itself, because this risks	// @audit-issue: Length of the line is: 121

115:     * are ERC-1167 minimal immutable clones and cannot delgate to another proxy. Thus, `proxiableUUID` was updated to support	// @audit-issue: Length of the line is: 127
```
[59](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L59-L59), [62](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L62-L62), [113](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L113-L113), [115](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L115-L115), 


```solidity
Path: ./src/functions/Stable2.sol

217:            pd.maxStepSize = scaledReserves[j] * (pd.lutData.highPriceJ - pd.lutData.lowPriceJ) / pd.lutData.highPriceJ;	// @audit-issue: Length of the line is: 121

285:            pd.maxStepSize = scaledReserves[j] * (pd.lutData.highPriceJ - pd.lutData.lowPriceJ) / pd.lutData.highPriceJ;	// @audit-issue: Length of the line is: 121
```
[217](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L217-L217), [285](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L285-L285), 


```solidity
Path: ./src/functions/StableLUT/Stable2LUT1.sol

70:                                    0.404944e6, 0, 6.130393650367882863e18, 0.370355e6, 0, 6.866040888412029197e18, 1e18	// @audit-issue: Length of the line is: 121

100:                                    0.516039e6, 0, 4.363493111652613443e18, 0.478063e6, 0, 4.887112285050926097e18, 1e18	// @audit-issue: Length of the line is: 121

142:                                    0.708539e6, 0, 2.475963176294809553e18, 0.670518e6, 0, 2.773078757450186949e18, 1e18	// @audit-issue: Length of the line is: 121

148:                                    0.746003e6, 0, 2.210681407406080101e18, 0.708539e6, 0, 2.475963176294809553e18, 1e18	// @audit-issue: Length of the line is: 121

152:                                    0.782874e6, 0, 1.973822685183999948e18, 0.746003e6, 0, 2.210681407406080101e18, 1e18	// @audit-issue: Length of the line is: 121

182:                                    0.873157e6, 0, 1.485947395978354457e18, 0.855108e6, 0, 1.573519359999999923e18, 1e18	// @audit-issue: Length of the line is: 121

188:                                    0.879393e6, 0, 1.456811172527798348e18, 0.873157e6, 0, 1.485947395978354457e18, 1e18	// @audit-issue: Length of the line is: 121

192:                                    0.885627e6, 0, 1.428246247576273165e18, 0.879393e6, 0, 1.456811172527798348e18, 1e18	// @audit-issue: Length of the line is: 121

226:                                    0.898101e6, 0, 1.372785705090612263e18, 0.891863e6, 0, 1.400241419192424397e18, 1e18	// @audit-issue: Length of the line is: 121

254:                                    0.916852e6, 0, 1.293606630453796313e18, 0.910594e6, 0, 1.319478763062872151e18, 1e18	// @audit-issue: Length of the line is: 121

290:                                    0.935697e6, 0, 1.218994419994757328e18, 0.929402e6, 0, 1.243374308394652239e18, 1e18	// @audit-issue: Length of the line is: 121

372:                                    0.993421e6, 0, 1.020000000000000018e18, 0.986882e6, 0, 1.040399999999999991e18, 1e18	// @audit-issue: Length of the line is: 121

376:                                    1.006758e6, 0, 0.980000000000000093e18, 0.993421e6, 0, 1.020000000000000018e18, 1e18	// @audit-issue: Length of the line is: 121

412:                                    1.027335e6, 0, 0.922368159999999992e18, 1.020422e6, 0, 0.941192000000000029e18, 1e18	// @audit-issue: Length of the line is: 121

522:                                    1.108094e6, 0, 0.738569102645403985e18, 1.100323e6, 0, 0.753641941474902044e18, 1e18	// @audit-issue: Length of the line is: 121

552:                                    1.132044e6, 0, 0.695135330857033051e18, 1.123949e6, 0, 0.709321766180645907e18, 1e18	// @audit-issue: Length of the line is: 121

596:                                    1.256266e6, 0, 0.527731916799999978e18, 1.195079e6, 0, 0.599695360000000011e18, 1e18	// @audit-issue: Length of the line is: 121

602:                                    1.325188e6, 0, 0.464404086784000025e18, 1.256266e6, 0, 0.527731916799999978e18, 1e18	// @audit-issue: Length of the line is: 121

606:                                    1.403579e6, 0, 0.408675596369920013e18, 1.325188e6, 0, 0.464404086784000025e18, 1e18	// @audit-issue: Length of the line is: 121

636:                                    1.716848e6, 0, 0.278500976009402101e18, 1.596984e6, 0, 0.316478381828866062e18, 1e18	// @audit-issue: Length of the line is: 121

642:                                    1.855977e6, 0, 0.245080858888273884e18, 1.716848e6, 0, 0.278500976009402101e18, 1e18	// @audit-issue: Length of the line is: 121

678:                                    2.680458e6, 0, 0.146973853900112583e18, 2.425256e6, 0, 0.167015743068309769e18, 1e18	// @audit-issue: Length of the line is: 121

684:                                    2.977411e6, 0, 0.129336991432099091e18, 2.680458e6, 0, 0.146973853900112583e18, 1e18	// @audit-issue: Length of the line is: 121

688:                                    3.322705e6, 0, 0.113816552460247203e18, 2.977411e6, 0, 0.129336991432099091e18, 1e18	// @audit-issue: Length of the line is: 121

718:                                    4.729321e6, 0, 0.077562793638189589e18, 4.189464e6, 0, 0.088139538225215433e18, 1e18	// @audit-issue: Length of the line is: 121

724:                                    10.37089e6, 0, 0.035714285714285712e18, 4.729321e6, 0, 0.077562793638189589e18, 1e18	// @audit-issue: Length of the line is: 121
```
[70](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L70-L70), [100](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L100-L100), [142](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L142-L142), [148](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L148-L148), [152](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L152-L152), [182](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L182-L182), [188](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L188-L188), [192](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L192-L192), [226](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L226-L226), [254](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L254-L254), [290](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L290-L290), [372](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L372-L372), [376](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L376-L376), [412](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L412-L412), [522](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L522-L522), [552](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L552-L552), [596](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L596-L596), [602](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L602-L602), [606](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L606-L606), [636](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L636-L636), [642](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L642-L642), [678](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L678-L678), [684](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L684-L684), [688](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L688-L688), [718](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L718-L718), [724](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L724-L724), 


#### Recommendation

To adhere to coding standards and enhance code readability, consider splitting long lines of code when they approach the recommended maximum line length. In Solidity, a common guideline is to limit lines to a maximum of 120 characters. Splitting long lines can improve code maintainability and make it easier to understand.

### Subtraction may underflow if result of multiplication is too large
The following expressions may underflow, as the subtrahend could be greater than the minuend in case the former is multiplied by a large number.

```solidity
Path: ./src/functions/Stable2.sol

372:        return (reserve * reserve + c) / (reserve * 2 + b - lpTokenSupply);	// @audit-issue

391:            return reserve
392:                - pd.maxStepSize * (pd.targetPrice - pd.currentPrice) / (pd.lutData.highPrice - pd.lutData.lowPrice);	// @audit-issue
```
[372](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L372-L372), [392](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L391-L392), 


#### Recommendation

Implement checks to guard against underflow in subtraction operations that follow multiplications in your Solidity contracts. Use SafeMath library or Solidity 0.8.x or higher, which has built-in overflow/underflow checks, to safely perform these arithmetic operations. Before performing the subtraction, ensure that the minuend is greater than or equal to the subtrahend. If using a version of Solidity that does not include automatic checks, consider adding explicit require statements to validate this condition, or use a library like OpenZeppelin's SafeMath to provide these safeguards. Careful handling of arithmetic operations, especially in critical functions involving financial calculations, is crucial for the security and correctness of your contract.

### Proper Assignment of `immutable` Variables in Solidity Constructors
In Solidity, `immutable` variables are designed to be assigned once and only once, and their value is set during contract creation. However, a common mistake is to assign an `immutable` variable directly at the point of declaration.

```solidity
Path: ./src/WellUpgradeable.sol

17:    address private immutable ___self = address(this);	// @audit-issue
```
[17](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L17-L17), 


#### Recommendation

Refrain from assigning values to `immutable` variables at the point of their declaration. Instead, assign values to these variables within the constructor of your Solidity contract. This ensures that the `immutable` variables are correctly initialized during contract deployment. For example:
```solidity
uint256 immutable someValue;

constructor() {
    someValue = block.timestamp + 45 days;
}
```


### Bug in Deduplication of Verbatim Blocks
The current Solidity version has the [Bug in Deduplication of Verbatim Blocks](https://soliditylang.org/blog/2023/11/08/verbatim-invalid-deduplication-bug/) issue.

```solidity
Path: ./src/WellUpgradeable.sol

3:pragma solidity ^0.8.20;	// @audit-issue
```
[3](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L3-L3), 


```solidity
Path: ./src/functions/Stable2.sol

3:pragma solidity ^0.8.20;	// @audit-issue
```
[3](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L3-L3), 


```solidity
Path: ./src/functions/StableLUT/Stable2LUT1.sol

3:pragma solidity ^0.8.20;	// @audit-issue
```
[3](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L3-L3), 


#### Recommendation

It is recommended to follow the fix provided in the [link](https://soliditylang.org/blog/2023/11/08/verbatim-invalid-deduplication-bug/).

### Hardcoded string that is repeatedly used can be replaced with a constant
For better maintainability, please consider creating and using a constant for those strings instead of hardcoding them.

```solidity
Path: ./src/functions/StableLUT/Stable2LUT1.sol

35:                                    revert("LUT: Invalid price");	// @audit-issue: This string `LUT: Invalid price` is used also on line(s): `[727, 748, 2167]`
```
[35](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L35-L35), 


#### Recommendation

Identify hardcoded strings that are used repeatedly in your Solidity contract and replace them with constants. Define these constants at the beginning of your contract or in a separate dedicated contract or library if they are shared across multiple contracts. This practice centralizes the management of these values, making your code more maintainable and reducing the risk of inconsistencies or errors when changes are required. Refactoring to use constants not only simplifies updates but also enhances the readability and clarity of your code.

### Style Guide: Surround top level declarations in Solidity source with two blank lines.
1- Surround top level declarations in Solidity source with two blank lines.
2- Within a contract surround function declarations with a single blank line.


```solidity
Path: ./src/WellUpgradeable.sol

17:    address private immutable ___self = address(this);
18:
19:    /**
20:     * @notice verifies that the execution is called through an minimal proxy or is not a delegate call.
21:     */	// @audit-issue: There should be single blank line between function declarations.
22:    modifier notDelegatedOrIsMinimalProxy() {

49:    }
50:
51:    /**
52:     * @notice `initNoWellToken` allows for the Well to be initialized without deploying a Well token.
53:     */	// @audit-issue: There should be single blank line between function declarations.
54:    function initNoWellToken() external initializer {}

54:    function initNoWellToken() external initializer {}
55:
56:    // Wells deployed by aquifers use the EIP-1167 minimal proxy pattern for gas-efficent deployments.
57:    // This pattern breaks the UUPS upgrade pattern, as the `__self` variable is set to the initial well implmentation.
58:    // `_authorizeUpgrade` and `upgradeTo` are modified to allow for upgrades to the Well implementation.
59:    // verification is done by verifying the ERC1967 implmentation (the well address) maps to the aquifers well -> implmentation mapping.
60:
61:    /**
62:     * @notice Check that the execution is being performed through a delegatecall call and that the execution context is
63:     * a proxy contract with an ERC1167 minimal proxy from an aquifier, pointing to a well implmentation.
64:     */	// @audit-issue: There should be single blank line between function declarations.
65:    function _authorizeUpgrade(address newImplmentation) internal view override {

85:    }
86:
87:    /**
88:     * @notice Upgrades the implementation of the proxy to `newImplementation`.
89:     * Calls {_authorizeUpgrade}.
90:     * @dev `upgradeTo` was modified to support ERC-1167 minimal proxies
91:     * cloned (Bored) by an Aquifer.
92:     */	// @audit-issue: There should be single blank line between function declarations.
93:    function upgradeTo(address newImplementation) public override {

96:    }
97:
98:    /**
99:     * @notice Upgrades the implementation of the proxy to `newImplementation`.
100:     * Calls {_authorizeUpgrade}.
101:     * @dev `upgradeTo` was modified to support ERC-1167 minimal proxies
102:     * cloned (Bored) by an Aquifer.
103:     */	// @audit-issue: There should be single blank line between function declarations.
104:    function upgradeToAndCall(address newImplementation, bytes memory data) public payable override {

107:    }
108:
109:    /**
110:     * @dev Implementation of the ERC1822 {proxiableUUID} function. This returns the storage slot used by the
111:     * implementation. It is used to validate the implementation's compatibility when performing an upgrade.
112:     *
113:     * IMPORTANT: A proxy pointing at a proxiable contract should not be considered proxiable itself, because this risks
114:     * bricking a proxy that upgrades to it, by delegating to itself until out of gas. However, Wells bored by Aquifers
115:     * are ERC-1167 minimal immutable clones and cannot delgate to another proxy. Thus, `proxiableUUID` was updated to support
116:     * this specific usecase.
117:     */	// @audit-issue: There should be single blank line between function declarations.
118:    function proxiableUUID() external view override notDelegatedOrIsMinimalProxy returns (bytes32) {
```
[21](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L17-L22), [53](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L49-L54), [64](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L54-L65), [92](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L85-L93), [103](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L96-L104), [117](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L107-L118), 


```solidity
Path: ./src/functions/Stable2.sol

51:    error InvalidLUT();
52:
53:    // Due to the complexity of `calcReserveAtRatioLiquidity` and `calcReserveAtRatioSwap`,
54:    // a LUT is used to reduce the complexity of the calculations on chain.
55:    // the lookup table contract implements 3 functions:
56:    // 1. getRatiosFromPriceLiquidity(uint256) -> PriceData memory
57:    // 2. getRatiosFromPriceSwap(uint256) -> PriceData memory
58:    // 3. getAParameter() -> uint256
59:    // Lookup tables are a function of the A parameter.	// @audit-issue: There should be single blank line between function declarations.
60:    constructor(address lut) {

64:    }
65:
66:    /**
67:     * @notice Calculate the amount of lp tokens given reserves.
68:     * D invariant calculation in non-overflowing integer operations iteratively
69:     * A * sum(x_i) * n**n + D = A * D * n**n + D**(n+1) / (n**n * prod(x_i))
70:     *
71:     * Converging solution:
72:     * D[j+1] = (4 * A * sum(b_i) - (D[j] ** 3) / (4 * prod(b_i))) / (4 * A - 1)
73:     */	// @audit-issue: There should be single blank line between function declarations.
74:    function calcLpTokenSupply(

103:    }
104:
105:    /**
106:     * @notice Calculate x[i] if one reduces D from being calculated for reserves to D
107:     * Done by solving quadratic equation iteratively.
108:     * x_1**2 + x_1 * (sum' - (A*n**n - 1) * D / (A * n**n)) = D ** (n + 1) / (n ** (2 * n) * prod' * A)
109:     * x_1**2 + b*x_1 = c
110:     * x_1 = (x_1**2 + c) / (2*x_1 + b)
111:     * @dev This function has a precision of +/- 1,
112:     * which may round in favor of the well or the user.
113:     */	// @audit-issue: There should be single blank line between function declarations.
114:    function calcReserve(

144:    }
145:
146:    /**
147:     * @inheritdoc IMultiFlowPumpWellFunction
148:     * @dev Returns a rate with  decimal precision.
149:     * Requires a minimum scaled reserves of 1e12.
150:     * 6 decimals was chosen as higher decimals would require a higher minimum scaled reserve,
151:     * which is prohibtive for large value tokens.
152:     */	// @audit-issue: There should be single blank line between function declarations.
153:    function calcRate(

166:    }
167:
168:    /**
169:     * @inheritdoc IMultiFlowPumpWellFunction
170:     * @dev `calcReserveAtRatioSwap` fetches the closes approximate ratios from the target price,
171:     * and performs newtons method in order to converge into a reserve.
172:     */	// @audit-issue: There should be single blank line between function declarations.
173:    function calcReserveAtRatioSwap(

239:    }
240:
241:    /**
242:     * @inheritdoc IBeanstalkWellFunction
243:     * @dev `calcReserveAtRatioLiquidity` fetches the closes approximate ratios from the target price,
244:     * and performs newtons method in order to converge into a reserve.
245:     */	// @audit-issue: There should be single blank line between function declarations.
246:    function calcReserveAtRatioLiquidity(

304:    }
305:
306:    /**
307:     * @notice decodes the data encoded in the well.
308:     * @return decimals an array of the decimals of the tokens in the well.
309:     */	// @audit-issue: There should be single blank line between function declarations.
310:    function decodeWellData(bytes memory data) public view virtual returns (uint256[] memory decimals) {

333:    }
334:
335:    /**
336:     * @notice internal calcRate function.
337:     */	// @audit-issue: There should be single blank line between function declarations.
338:    function _calcRate(

351:    }
352:
353:    /**
354:     * @notice scale `reserves` by `precision`.
355:     * @dev this sets both reserves to 18 decimals.
356:     */	// @audit-issue: There should be single blank line between function declarations.
357:    function getScaledReserves(

382:    }
383:
384:    /**
385:     * @notice calculates the step size, and returns the updated reserve.
386:     */	// @audit-issue: There should be single blank line between function declarations.
387:    function updateReserve(PriceData memory pd, uint256 reserve) internal pure returns (uint256) {
```
[59](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L51-L60), [73](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L64-L74), [113](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L103-L114), [152](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L144-L153), [172](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L166-L173), [245](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L239-L246), [309](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L304-L310), [337](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L333-L338), [356](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L351-L357), [386](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L382-L387), 


```solidity
Path: ./src/functions/StableLUT/Stable2LUT1.sol

21:    }
22:
23:    /**
24:     * @notice Returns the estimated range of reserve ratios for a given price,
25:     * assuming one token reserve remains constant.
26:     */	// @audit-issue: There should be single blank line between function declarations.
27:    function getRatiosFromPriceLiquidity(uint256 price) external pure returns (PriceData memory) {

734:    }
735:
736:    /**
737:     * @notice Returns the estimated range of reserve ratios for a given price,
738:     * assuming the pool liquidity remains constant.
739:     */	// @audit-issue: There should be single blank line between function declarations.
740:    function getRatiosFromPriceSwap(uint256 price) external pure returns (PriceData memory) {
```
[26](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L21-L27), [739](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L734-L740), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/latest/style-guide.html#blank-lines).

### `constants` should be defined rather than using magic numbers
Even [assembly](https://github.com/code-423n4/2022-05-opensea-seaport/blob/9d7ce4d08bf3c3010304a0476a785c70c0e90ae7/contracts/lib/TokenTransferrer.sol#L35-L39) can benefit from using readable constants instead of hex/numeric literals

```solidity
Path: ./src/functions/Stable2.sol

88:        for (uint256 i = 0; i < 255; i++) {	// @audit-issue

135:                    return reserve / (10 ** (18 - decimals[j]));	// @audit-issue

139:                    return reserve / (10 ** (18 - decimals[j]));	// @audit-issue

128:        for (uint256 i; i < 255; ++i) {	// @audit-issue

231:                    return scaledReserves[j] / (10 ** (18 - decimals[j]));	// @audit-issue

235:                    return scaledReserves[j] / (10 ** (18 - decimals[j]));	// @audit-issue

220:        for (uint256 k; k < 255; k++) {	// @audit-issue

296:                    return scaledReserves[j] / (10 ** (18 - decimals[j]));	// @audit-issue

300:                    return scaledReserves[j] / (10 ** (18 - decimals[j]));	// @audit-issue

288:        for (uint256 k; k < 255; k++) {	// @audit-issue

320:        if (decimal0 > 18 || decimal1 > 18) revert InvalidTokenDecimals();	// @audit-issue

350:        rate = _reserves[i] - calcReserve(_reserves, i, lpTokenSupply, abi.encode(18, 18));	// @audit-issue

362:        scaledReserves[0] = reserves[0] * 10 ** (18 - decimals[0]);	// @audit-issue

363:        scaledReserves[1] = reserves[1] * 10 ** (18 - decimals[1]);	// @audit-issue

372:        return (reserve * reserve + c) / (reserve * 2 + b - lpTokenSupply);	// @audit-issue
```
[88](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L88-L88), [135](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L135-L135), [139](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L139-L139), [128](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L128-L128), [231](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L231-L231), [235](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L235-L235), [220](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L220-L220), [296](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L296-L296), [300](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L300-L300), [288](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L288-L288), [320](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L320-L320), [350](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L350-L350), [362](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L362-L362), [363](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L363-L363), [372](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L372-L372), 


```solidity
Path: ./src/functions/StableLUT/Stable2LUT1.sol

28:        if (price < 1.006758e6) {	// @audit-issue

29:            if (price < 0.885627e6) {	// @audit-issue

30:                if (price < 0.59332e6) {	// @audit-issue

31:                    if (price < 0.404944e6) {	// @audit-issue

32:                        if (price < 0.30624e6) {	// @audit-issue

33:                            if (price < 0.27702e6) {	// @audit-issue

34:                                if (price < 0.001083e6) {	// @audit-issue

46:                            if (price < 0.370355e6) {	// @audit-issue

47:                                if (price < 0.337394e6) {	// @audit-issue

75:                        if (price < 0.516039e6) {	// @audit-issue

76:                            if (price < 0.478063e6) {	// @audit-issue

77:                                if (price < 0.440934e6) {	// @audit-issue

104:                            if (price < 0.554558e6) {	// @audit-issue

116:                    if (price < 0.782874e6) {	// @audit-issue

117:                        if (price < 0.708539e6) {	// @audit-issue

118:                            if (price < 0.670518e6) {	// @audit-issue

119:                                if (price < 0.632052e6) {	// @audit-issue

146:                            if (price < 0.746003e6) {	// @audit-issue

157:                        if (price < 0.873157e6) {	// @audit-issue

158:                            if (price < 0.855108e6) {	// @audit-issue

159:                                if (price < 0.819199e6) {	// @audit-issue

186:                            if (price < 0.879393e6) {	// @audit-issue

199:                if (price < 0.94201e6) {	// @audit-issue

200:                    if (price < 0.916852e6) {	// @audit-issue

201:                        if (price < 0.898101e6) {	// @audit-issue

202:                            if (price < 0.891863e6) {	// @audit-issue

203:                                if (price < 0.89081e6) {	// @audit-issue

230:                            if (price < 0.910594e6) {	// @audit-issue

231:                                if (price < 0.904344e6) {	// @audit-issue

259:                        if (price < 0.929402e6) {	// @audit-issue

260:                            if (price < 0.9266e6) {	// @audit-issue

261:                                if (price < 0.92312e6) {	// @audit-issue

288:                            if (price < 0.935697e6) {	// @audit-issue

300:                    if (price < 0.96748e6) {	// @audit-issue

301:                        if (price < 0.961075e6) {	// @audit-issue

302:                            if (price < 0.954697e6) {	// @audit-issue

303:                                if (price < 0.948343e6) {	// @audit-issue

330:                            if (price < 0.962847e6) {	// @audit-issue

341:                        if (price < 0.986882e6) {	// @audit-issue

342:                            if (price < 0.98038e6) {	// @audit-issue

343:                                if (price < 0.973914e6) {	// @audit-issue

370:                            if (price < 0.993421e6) {	// @audit-issue

384:            if (price < 1.140253e6) {	// @audit-issue

385:                if (price < 1.077582e6) {	// @audit-issue

386:                    if (price < 1.04366e6) {	// @audit-issue

387:                        if (price < 1.027335e6) {	// @audit-issue

388:                            if (price < 1.020422e6) {	// @audit-issue

389:                                if (price < 1.013564e6) {	// @audit-issue

416:                            if (price < 1.041342e6) {	// @audit-issue

417:                                if (price < 1.034307e6) {	// @audit-issue

445:                        if (price < 1.062857e6) {	// @audit-issue

446:                            if (price < 1.055613e6) {	// @audit-issue

447:                                if (price < 1.048443e6) {	// @audit-issue

474:                            if (price < 1.070179e6) {	// @audit-issue

486:                    if (price < 1.108094e6) {	// @audit-issue

487:                        if (price < 1.09265e6) {	// @audit-issue

488:                            if (price < 1.090025e6) {	// @audit-issue

489:                                if (price < 1.085071e6) {	// @audit-issue

516:                            if (price < 1.100323e6) {	// @audit-issue

527:                        if (price < 1.132044e6) {	// @audit-issue

528:                            if (price < 1.123949e6) {	// @audit-issue

529:                                if (price < 1.115967e6) {	// @audit-issue

556:                            if (price < 1.14011e6) {	// @audit-issue

569:                if (price < 2.01775e6) {	// @audit-issue

570:                    if (price < 1.403579e6) {	// @audit-issue

571:                        if (price < 1.256266e6) {	// @audit-issue

572:                            if (price < 1.195079e6) {	// @audit-issue

573:                                if (price < 1.148586e6) {	// @audit-issue

600:                            if (price < 1.325188e6) {	// @audit-issue

611:                        if (price < 1.716848e6) {	// @audit-issue

612:                            if (price < 1.596984e6) {	// @audit-issue

613:                                if (price < 1.493424e6) {	// @audit-issue

640:                            if (price < 1.855977e6) {	// @audit-issue

652:                    if (price < 3.322705e6) {	// @audit-issue

653:                        if (price < 2.680458e6) {	// @audit-issue

654:                            if (price < 2.425256e6) {	// @audit-issue

655:                                if (price < 2.206036e6) {	// @audit-issue

682:                            if (price < 2.977411e6) {	// @audit-issue

693:                        if (price < 4.729321e6) {	// @audit-issue

694:                            if (price < 4.189464e6) {	// @audit-issue

695:                                if (price < 3.723858e6) {	// @audit-issue

722:                            if (price < 10.37089e6) {	// @audit-issue

741:        if (price < 0.993344e6) {	// @audit-issue

742:            if (price < 0.834426e6) {	// @audit-issue

743:                if (price < 0.718073e6) {	// @audit-issue

744:                    if (price < 0.391201e6) {	// @audit-issue

745:                        if (price < 0.264147e6) {	// @audit-issue

746:                            if (price < 0.213318e6) {	// @audit-issue

747:                                if (price < 0.001083e6) {	// @audit-issue

761:                                if (price < 0.237671e6) {	// @audit-issue

784:                            if (price < 0.323531e6) {	// @audit-issue

785:                                if (price < 0.292771e6) {	// @audit-issue

807:                                if (price < 0.356373e6) {	// @audit-issue

831:                        if (price < 0.546918e6) {	// @audit-issue

832:                            if (price < 0.466197e6) {	// @audit-issue

833:                                if (price < 0.427871e6) {	// @audit-issue

855:                                if (price < 0.50596e6) {	// @audit-issue

878:                            if (price < 0.631434e6) {	// @audit-issue

879:                                if (price < 0.588821e6) {	// @audit-issue

901:                                if (price < 0.67456e6) {	// @audit-issue

926:                    if (price < 0.801931e6) {	// @audit-issue

927:                        if (price < 0.780497e6) {	// @audit-issue

928:                            if (price < 0.769833e6) {	// @audit-issue

929:                                if (price < 0.76195e6) {	// @audit-issue

951:                                if (price < 0.775161e6) {	// @audit-issue

974:                            if (price < 0.791195e6) {	// @audit-issue

975:                                if (price < 0.785842e6) {	// @audit-issue

997:                                if (price < 0.796558e6) {	// @audit-issue

1021:                        if (price < 0.818119e6) {	// @audit-issue

1022:                            if (price < 0.807315e6) {	// @audit-issue

1023:                                if (price < 0.806314e6) {	// @audit-issue

1045:                                if (price < 0.812711e6) {	// @audit-issue

1068:                            if (price < 0.828976e6) {	// @audit-issue

1069:                                if (price < 0.82354e6) {	// @audit-issue

1105:                if (price < 0.907266e6) {	// @audit-issue

1106:                    if (price < 0.873109e6) {	// @audit-issue

1107:                        if (price < 0.851493e6) {	// @audit-issue

1108:                            if (price < 0.845379e6) {	// @audit-issue

1109:                                if (price < 0.839894e6) {	// @audit-issue

1131:                                if (price < 0.850882e6) {	// @audit-issue

1154:                            if (price < 0.86195e6) {	// @audit-issue

1155:                                if (price < 0.856405e6) {	// @audit-issue

1177:                                if (price < 0.867517e6) {	// @audit-issue

1201:                        if (price < 0.895753e6) {	// @audit-issue

1202:                            if (price < 0.884372e6) {	// @audit-issue

1203:                                if (price < 0.878727e6) {	// @audit-issue

1225:                                if (price < 0.890047e6) {	// @audit-issue

1248:                            if (price < 0.901491e6) {	// @audit-issue

1249:                                if (price < 0.898085e6) {	// @audit-issue

1284:                    if (price < 0.948888e6) {	// @audit-issue

1285:                        if (price < 0.930767e6) {	// @audit-issue

1286:                            if (price < 0.918932e6) {	// @audit-issue

1287:                                if (price < 0.913079e6) {	// @audit-issue

1309:                                if (price < 0.924827e6) {	// @audit-issue

1332:                            if (price < 0.942795e6) {	// @audit-issue

1333:                                if (price < 0.936756e6) {	// @audit-issue

1355:                                if (price < 0.947076e6) {	// @audit-issue

1379:                        if (price < 0.973868e6) {	// @audit-issue

1380:                            if (price < 0.961249e6) {	// @audit-issue

1381:                                if (price < 0.955039e6) {	// @audit-issue

1403:                                if (price < 0.967525e6) {	// @audit-issue

1426:                            if (price < 0.986773e6) {	// @audit-issue

1427:                                if (price < 0.980283e6) {	// @audit-issue

1464:            if (price < 1.211166e6) {	// @audit-issue

1465:                if (price < 1.09577e6) {	// @audit-issue

1466:                    if (price < 1.048893e6) {	// @audit-issue

1467:                        if (price < 1.027293e6) {	// @audit-issue

1468:                            if (price < 1.01345e6) {	// @audit-issue

1469:                                if (price < 1.006679e6) {	// @audit-issue

1491:                                if (price < 1.020319e6) {	// @audit-issue

1514:                            if (price < 1.034375e6) {	// @audit-issue

1515:                                if (price < 1.033686e6) {	// @audit-issue

1537:                                if (price < 1.041574e6) {	// @audit-issue

1561:                        if (price < 1.071652e6) {	// @audit-issue

1562:                            if (price < 1.063925e6) {	// @audit-issue

1563:                                if (price < 1.056342e6) {	// @audit-issue

1585:                                if (price < 1.070147e6) {	// @audit-issue

1608:                            if (price < 1.087566e6) {	// @audit-issue

1609:                                if (price < 1.079529e6) {	// @audit-issue

1644:                    if (price < 1.15496e6) {	// @audit-issue

1645:                        if (price < 1.121482e6) {	// @audit-issue

1646:                            if (price < 1.110215e6) {	// @audit-issue

1647:                                if (price < 1.104151e6) {	// @audit-issue

1669:                                if (price < 1.112718e6) {	// @audit-issue

1692:                            if (price < 1.139642e6) {	// @audit-issue

1693:                                if (price < 1.130452e6) {	// @audit-issue

1715:                                if (price < 1.149062e6) {	// @audit-issue

1739:                        if (price < 1.189304e6) {	// @audit-issue

1740:                            if (price < 1.168643e6) {	// @audit-issue

1741:                                if (price < 1.158725e6) {	// @audit-issue

1763:                                if (price < 1.178832e6) {	// @audit-issue

1786:                            if (price < 1.205768e6) {	// @audit-issue

1787:                                if (price < 1.200076e6) {	// @audit-issue

1823:                if (price < 1.393403e6) {	// @audit-issue

1824:                    if (price < 1.299217e6) {	// @audit-issue

1825:                        if (price < 1.259043e6) {	// @audit-issue

1826:                            if (price < 1.234362e6) {	// @audit-issue

1827:                                if (price < 1.222589e6) {	// @audit-issue

1849:                                if (price < 1.246507e6) {	// @audit-issue

1872:                            if (price < 1.271991e6) {	// @audit-issue

1873:                                if (price < 1.264433e6) {	// @audit-issue

1895:                                if (price < 1.285375e6) {	// @audit-issue

1919:                        if (price < 1.343751e6) {	// @audit-issue

1920:                            if (price < 1.328377e6) {	// @audit-issue

1921:                                if (price < 1.313542e6) {	// @audit-issue

1943:                                if (price < 1.333292e6) {	// @audit-issue

1966:                            if (price < 1.376232e6) {	// @audit-issue

1967:                                if (price < 1.359692e6) {	// @audit-issue

2002:                    if (price < 2.209802e6) {	// @audit-issue

2003:                        if (price < 1.514667e6) {	// @audit-issue

2004:                            if (price < 1.415386e6) {	// @audit-issue

2005:                                if (price < 1.41124e6) {	// @audit-issue

2027:                                if (price < 1.42978e6) {	// @audit-issue

2050:                            if (price < 1.786708e6) {	// @audit-issue

2051:                                if (price < 1.636249e6) {	// @audit-issue

2073:                                if (price < 1.974398e6) {	// @audit-issue

2097:                        if (price < 3.931396e6) {	// @audit-issue

2098:                            if (price < 2.878327e6) {	// @audit-issue

2099:                                if (price < 2.505865e6) {	// @audit-issue

2121:                                if (price < 3.346057e6) {	// @audit-issue

2144:                            if (price < 10.709509e6) {	// @audit-issue

2145:                                if (price < 4.660591e6) {	// @audit-issue
```
[28](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L28-L28), [29](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L29-L29), [30](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L30-L30), [31](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L31-L31), [32](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L32-L32), [33](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L33-L33), [34](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L34-L34), [46](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L46-L46), [47](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L47-L47), [75](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L75-L75), [76](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L76-L76), [77](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L77-L77), [104](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L104-L104), [116](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L116-L116), [117](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L117-L117), [118](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L118-L118), [119](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L119-L119), [146](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L146-L146), [157](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L157-L157), [158](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L158-L158), [159](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L159-L159), [186](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L186-L186), [199](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L199-L199), [200](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L200-L200), [201](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L201-L201), [202](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L202-L202), [203](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L203-L203), [230](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L230-L230), [231](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L231-L231), [259](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L259-L259), [260](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L260-L260), [261](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L261-L261), [288](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L288-L288), [300](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L300-L300), [301](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L301-L301), [302](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L302-L302), [303](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L303-L303), [330](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L330-L330), [341](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L341-L341), [342](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L342-L342), [343](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L343-L343), [370](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L370-L370), [384](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L384-L384), [385](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L385-L385), [386](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L386-L386), [387](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L387-L387), [388](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L388-L388), [389](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L389-L389), [416](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L416-L416), [417](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L417-L417), [445](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L445-L445), [446](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L446-L446), [447](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L447-L447), [474](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L474-L474), [486](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L486-L486), [487](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L487-L487), [488](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L488-L488), [489](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L489-L489), [516](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L516-L516), [527](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L527-L527), [528](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L528-L528), [529](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L529-L529), [556](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L556-L556), [569](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L569-L569), [570](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L570-L570), [571](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L571-L571), [572](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L572-L572), [573](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L573-L573), [600](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L600-L600), [611](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L611-L611), [612](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L612-L612), [613](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L613-L613), [640](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L640-L640), [652](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L652-L652), [653](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L653-L653), [654](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L654-L654), [655](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L655-L655), [682](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L682-L682), [693](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L693-L693), [694](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L694-L694), [695](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L695-L695), [722](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L722-L722), [741](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L741-L741), [742](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L742-L742), [743](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L743-L743), [744](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L744-L744), [745](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L745-L745), [746](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L746-L746), [747](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L747-L747), [761](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L761-L761), [784](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L784-L784), [785](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L785-L785), [807](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L807-L807), [831](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L831-L831), [832](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L832-L832), [833](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L833-L833), [855](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L855-L855), [878](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L878-L878), [879](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L879-L879), [901](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L901-L901), [926](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L926-L926), [927](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L927-L927), [928](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L928-L928), [929](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L929-L929), [951](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L951-L951), [974](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L974-L974), [975](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L975-L975), [997](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L997-L997), [1021](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1021-L1021), [1022](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1022-L1022), [1023](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1023-L1023), [1045](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1045-L1045), [1068](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1068-L1068), [1069](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1069-L1069), [1105](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1105-L1105), [1106](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1106-L1106), [1107](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1107-L1107), [1108](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1108-L1108), [1109](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1109-L1109), [1131](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1131-L1131), [1154](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1154-L1154), [1155](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1155-L1155), [1177](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1177-L1177), [1201](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1201-L1201), [1202](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1202-L1202), [1203](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1203-L1203), [1225](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1225-L1225), [1248](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1248-L1248), [1249](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1249-L1249), [1284](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1284-L1284), [1285](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1285-L1285), [1286](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1286-L1286), [1287](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1287-L1287), [1309](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1309-L1309), [1332](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1332-L1332), [1333](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1333-L1333), [1355](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1355-L1355), [1379](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1379-L1379), [1380](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1380-L1380), [1381](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1381-L1381), [1403](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1403-L1403), [1426](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1426-L1426), [1427](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1427-L1427), [1464](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1464-L1464), [1465](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1465-L1465), [1466](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1466-L1466), [1467](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1467-L1467), [1468](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1468-L1468), [1469](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1469-L1469), [1491](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1491-L1491), [1514](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1514-L1514), [1515](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1515-L1515), [1537](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1537-L1537), [1561](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1561-L1561), [1562](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1562-L1562), [1563](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1563-L1563), [1585](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1585-L1585), [1608](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1608-L1608), [1609](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1609-L1609), [1644](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1644-L1644), [1645](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1645-L1645), [1646](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1646-L1646), [1647](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1647-L1647), [1669](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1669-L1669), [1692](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1692-L1692), [1693](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1693-L1693), [1715](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1715-L1715), [1739](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1739-L1739), [1740](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1740-L1740), [1741](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1741-L1741), [1763](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1763-L1763), [1786](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1786-L1786), [1787](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1787-L1787), [1823](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1823-L1823), [1824](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1824-L1824), [1825](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1825-L1825), [1826](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1826-L1826), [1827](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1827-L1827), [1849](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1849-L1849), [1872](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1872-L1872), [1873](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1873-L1873), [1895](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1895-L1895), [1919](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1919-L1919), [1920](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1920-L1920), [1921](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1921-L1921), [1943](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1943-L1943), [1966](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1966-L1966), [1967](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L1967-L1967), [2002](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L2002-L2002), [2003](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L2003-L2003), [2004](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L2004-L2004), [2005](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L2005-L2005), [2027](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L2027-L2027), [2050](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L2050-L2050), [2051](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L2051-L2051), [2073](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L2073-L2073), [2097](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L2097-L2097), [2098](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L2098-L2098), [2099](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L2099-L2099), [2121](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L2121-L2121), [2144](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L2144-L2144), [2145](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L2145-L2145), 


#### Recommendation

Consider defining constants with meaningful names for magic numbers and hexadecimal literals to improve code readability and maintainability.

### Lack of index element validation in function
There's no validation to check whether the index element provided as an argument actually exists in the call. This omission could lead to unintended behavior if an element that does not exist in the call is passed to the function. The function should validate that the provided index element exists in the call before proceeding.

```solidity
Path: ./src/functions/Stable2.sol

78:        if (reserves[0] == 0 && reserves[1] == 0) return 0;	// @audit-issue

346:        _reserves[i] = reserves[i];	// @audit-issue

347:        _reserves[j] = reserves[j] + PRICE_PRECISION;	// @audit-issue

362:        scaledReserves[0] = reserves[0] * 10 ** (18 - decimals[0]);	// @audit-issue

363:        scaledReserves[1] = reserves[1] * 10 ** (18 - decimals[1]);	// @audit-issue
```
[78](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L78-L78), [346](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L346-L346), [347](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L347-L347), [362](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L362-L362), [363](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L363-L363), 


#### Recommendation

Integrate explicit index validation checks at the beginning of functions that operate based on index elements. Use conditional statements to verify that the provided index falls within the valid range of existing elements. For array operations, ensure the index is less than the array's length. For mappings, consider additional logic to confirm the presence of a key. For example, in an array-based function:
```solidity
function getElementByIndex(uint256 index) public view returns (ElementType) {
    require(index < array.length, "Index out of bounds");
    return array[index];
}
```


### Contract should expose an `interface`
All `external`/`public` functions should extend an `interface`. This is useful to make sure that the whole API is extracted.


```solidity
Path: ./src/WellUpgradeable.sol

16:contract WellUpgradeable is Well, UUPSUpgradeable, OwnableUpgradeable {	// @audit-issue
```
[16](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L16-L16), 


#### Recommendation

Consider defining an `interface` that includes all `external`/`public` functions of the contract. Exposing a well-defined interface helps ensure that the entire API is extracted and provides a clear and standardized way for other contracts or users to interact with your contract.

### Consider using named returns
Using named returns makes the code more self-documenting, makes it easier to fill out NatSpec, and in some cases can save gas. The cases below are where there currently is at most one return statement, which is ideal for named returns.

```solidity
Path: ./src/WellUpgradeable.sol

118:    function proxiableUUID() external view override notDelegatedOrIsMinimalProxy returns (bytes32) {	// @audit-issue

122:    function getImplementation() external view returns (address) {	// @audit-issue

126:    function getVersion() external pure virtual returns (uint256) {	// @audit-issue

130:    function getInitializerVersion() external view returns (uint256) {	// @audit-issue
```
[118](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L118-L118), [122](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L122-L122), [126](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L126-L126), [130](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L130-L130), 


```solidity
Path: ./src/functions/Stable2.sol

327:    function name() external pure returns (string memory) {	// @audit-issue

331:    function symbol() external pure returns (string memory) {	// @audit-issue

366:    function _calcReserve(
367:        uint256 reserve,
368:        uint256 b,
369:        uint256 c,
370:        uint256 lpTokenSupply
371:    ) private pure returns (uint256) {	// @audit-issue

387:    function updateReserve(PriceData memory pd, uint256 reserve) internal pure returns (uint256) {	// @audit-issue
```
[327](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L327-L327), [331](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L331-L331), [371](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L366-L371), [387](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L387-L387), 


```solidity
Path: ./src/functions/StableLUT/Stable2LUT1.sol

19:    function getAParameter() external pure returns (uint256) {	// @audit-issue

27:    function getRatiosFromPriceLiquidity(uint256 price) external pure returns (PriceData memory) {	// @audit-issue

740:    function getRatiosFromPriceSwap(uint256 price) external pure returns (PriceData memory) {	// @audit-issue
```
[19](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L19-L19), [27](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L27-L27), [740](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L740-L740), 


#### Recommendation

Adopt named returns in your Solidity functions, especially in cases where functions contain a single return statement. This approach enhances code readability and documentation by making the return values clear and explicit. When defining your function, specify the return types with names, and manipulate these named variables directly within your function logic. Additionally, leverage named returns to streamline your NatSpec documentation, providing clear descriptions for each return variable. Evaluate your current contracts for opportunities to refactor functions to use named returns, prioritizing those with simple return patterns for immediate benefits in gas efficiency and code clarity.

### Missing events in initializers/deploys
As a best practice, consider emitting an event when the contract is initialized. In this way, it's easy for the user to track the exact point in time when the contract was initialized, by filtering the emitted events.

```solidity
Path: ./src/WellUpgradeable.sol

33:    function init(string memory _name, string memory _symbol) external override reinitializer(2) {	// @audit-issue
```
[33](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L33-L33), 


#### Recommendation

To provide transparency and enable users to track the initialization of the contract, consider emitting an event within the contract's initializer function. Emitting an event during initialization can help users pinpoint the exact moment the contract was initialized by filtering and monitoring the emitted events.

### Inefficient Array Usage
Use mappings instead of arrays for managing lists of data in order to conserve gas. Mappings are less expensive and more efficient for accessing any value without having to iterate through an array.

```solidity
Path: ./src/WellUpgradeable.sol

40:        IERC20[] memory _tokens = tokens();	// @audit-issue
```
[40](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L40-L40), 


```solidity
Path: ./src/functions/Stable2.sol

75:        uint256[] memory reserves,	// @audit-issue

79:        uint256[] memory decimals = decodeWellData(data);	// @audit-issue

81:        uint256[] memory scaledReserves = getScaledReserves(reserves, decimals);	// @audit-issue

115:        uint256[] memory reserves,	// @audit-issue

120:        uint256[] memory decimals = decodeWellData(data);	// @audit-issue

121:        uint256[] memory scaledReserves = getScaledReserves(reserves, decimals);	// @audit-issue

154:        uint256[] memory reserves,	// @audit-issue

159:        uint256[] memory decimals = decodeWellData(data);	// @audit-issue

160:        uint256[] memory scaledReserves = getScaledReserves(reserves, decimals);	// @audit-issue

174:        uint256[] memory reserves,	// @audit-issue

176:        uint256[] memory ratios,	// @audit-issue

181:        uint256[] memory decimals = decodeWellData(data);	// @audit-issue

182:        uint256[] memory scaledReserves = getScaledReserves(reserves, decimals);	// @audit-issue

185:        uint256[] memory scaledRatios = getScaledReserves(ratios, decimals);	// @audit-issue

247:        uint256[] calldata reserves,	// @audit-issue

249:        uint256[] calldata ratios,	// @audit-issue

254:        uint256[] memory decimals = decodeWellData(data);	// @audit-issue

255:        uint256[] memory scaledReserves = getScaledReserves(reserves, decimals);	// @audit-issue

258:        uint256[] memory scaledRatios = getScaledReserves(ratios, decimals);	// @audit-issue

339:        uint256[] memory reserves,	// @audit-issue

345:        uint256[] memory _reserves = new uint256[](2);	// @audit-issue

358:        uint256[] memory reserves,	// @audit-issue

359:        uint256[] memory decimals	// @audit-issue
```
[75](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L75-L75), [79](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L79-L79), [81](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L81-L81), [115](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L115-L115), [120](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L120-L120), [121](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L121-L121), [154](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L154-L154), [159](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L159-L159), [160](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L160-L160), [174](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L174-L174), [176](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L176-L176), [181](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L181-L181), [182](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L182-L182), [185](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L185-L185), [247](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L247-L247), [249](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L249-L249), [254](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L254-L254), [255](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L255-L255), [258](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L258-L258), [339](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L339-L339), [345](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L345-L345), [358](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L358-L358), [359](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L359-L359), 


#### Recommendation

In scenarios where data access efficiency is critical, prefer using mappings over arrays in Solidity contracts. Mappings offer more efficient and gas-effective data retrieval and updates, especially when dealing with large or frequently accessed datasets. Ensure to structure your data and choose keys thoughtfully to maximize the efficiency gains offered by mappings. While arrays might be suitable for ordered data or when the entire dataset needs to be iterated, for most other use cases, mappings are likely to be the more gas-efficient choice.

### Enum values should be used in place of constant array indexes
Consider using an `enum` instead of hardcoding an index access to make the code easier to understand.

```solidity
Path: ./src/functions/Stable2.sol

78:        if (reserves[0] == 0 && reserves[1] == 0) return 0;	// @audit-issue

85:        uint256 sumReserves = scaledReserves[0] + scaledReserves[1];	// @audit-issue

91:            dP = dP * lpTokenSupply / (scaledReserves[0] * N);	// @audit-issue

92:            dP = dP * lpTokenSupply / (scaledReserves[1] * N);	// @audit-issue

124:        (uint256 c, uint256 b) = getBandC(a * N * N, lpTokenSupply, j == 0 ? scaledReserves[1] : scaledReserves[0]);	// @audit-issue

323:        decimals[0] = decimal0;	// @audit-issue

324:        decimals[1] = decimal1;	// @audit-issue

362:        scaledReserves[0] = reserves[0] * 10 ** (18 - decimals[0]);	// @audit-issue

363:        scaledReserves[1] = reserves[1] * 10 ** (18 - decimals[1]);	// @audit-issue
```
[78](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L78-L78), [85](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L85-L85), [91](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L91-L91), [92](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L92-L92), [124](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L124-L124), [323](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L323-L323), [324](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L324-L324), [362](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L362-L362), [363](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L363-L363), 


#### Recommendation

To improve code readability and maintainability, replace hardcoded array indexes with corresponding enum values. Enum values provide descriptive names for array elements, making your code more self-explanatory and reducing the risk of errors when working with arrays. This enhances the overall clarity and robustness of your smart contract code.

### Complex math should be split into multiple steps
Consider splitting long arithmetic calculations (more than 4 operations) into multiple steps to improve the code readability.

```solidity
Path: ./src/functions/Stable2.sol

94:            lpTokenSupply = (Ann * sumReserves / A_PRECISION + (dP * N)) * lpTokenSupply	// @audit-issue
95:                / (((Ann - A_PRECISION) * lpTokenSupply / A_PRECISION) + ((N + 1) * dP));

94:            lpTokenSupply = (Ann * sumReserves / A_PRECISION + (dP * N)) * lpTokenSupply	// @audit-issue

95:                / (((Ann - A_PRECISION) * lpTokenSupply / A_PRECISION) + ((N + 1) * dP));	// @audit-issue

372:        return (reserve * reserve + c) / (reserve * 2 + b - lpTokenSupply);	// @audit-issue

380:        c = lpTokenSupply * lpTokenSupply / (reserves * N) * lpTokenSupply * A_PRECISION / (Ann * N);	// @audit-issue

391:            return reserve	// @audit-issue
392:                - pd.maxStepSize * (pd.targetPrice - pd.currentPrice) / (pd.lutData.highPrice - pd.lutData.lowPrice);

396:            return reserve	// @audit-issue
397:                + pd.maxStepSize * (pd.currentPrice - pd.targetPrice) / (pd.lutData.highPrice - pd.lutData.lowPrice);
```
[94](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L94-L95), [94](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L94-L94), [95](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L95-L95), [372](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L372-L372), [380](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L380-L380), [391](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L391-L392), [396](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L396-L397), 


#### Recommendation

To enhance code readability and maintainability, it's advisable to break down complex arithmetic calculations into multiple steps. This not only makes the code more understandable but also helps in debugging and verifying the correctness of calculations.

### Imports should be organized more systematically
The contract's interface should be imported first, followed by each of the interfaces it uses, followed by all other files. The examples below do not follow this layout.

```solidity
Path: ./src/functions/Stable2.sol

25:contract Stable2 is ProportionalLPToken2, IBeanstalkWellFunction {	// @audit-issue: `ProportionalLPToken2` came before `IBeanstalkWellFunction`.
```
[25](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L25-L25), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).

### Some variables have a implicit default visibility
Consider always adding an explicit visibility modifier for variables, as the default is `internal`.

```solidity
Path: ./src/functions/Stable2.sol

34:    uint256 constant N = 2;	// @audit-issue

37:    uint256 constant A_PRECISION = 100;	// @audit-issue

40:    uint256 constant PRICE_PRECISION = 1e6;	// @audit-issue

44:    uint256 constant PRICE_THRESHOLD = 100; // 0.01%	// @audit-issue

46:    address immutable lookupTable;	// @audit-issue

47:    uint256 immutable a;	// @audit-issue
```
[34](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L34-L34), [37](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L37-L37), [40](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L40-L40), [44](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L44-L44), [46](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L46-L46), [47](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L47-L47), 


#### Recommendation

Always add an explicit visibility modifier for variables to enhance code clarity and avoid potential issues. The default visibility for variables is `internal`, but specifying it explicitly makes your intentions clear.

### Unnecessary Use of override Keyword
In Solidity version 0.8.8 and later, the use of the override keyword becomes superfluous when a function is overriding solely from an interface and the function isn't present in multiple base contracts. Previously, the override keyword was required as an explicit indication to the compiler. However, this is no longer the case, and the extraneous use of the keyword can make the code less clean and more verbose.
Solidity documentation on [Function Overriding](https://docs.soliditylang.org/en/v0.8.20/contracts.html#function-overriding).


```solidity
Path: ./src/WellUpgradeable.sol

33:    function init(string memory _name, string memory _symbol) external override reinitializer(2) {	// @audit-issue

65:    function _authorizeUpgrade(address newImplmentation) internal view override {	// @audit-issue

93:    function upgradeTo(address newImplementation) public override {	// @audit-issue

104:    function upgradeToAndCall(address newImplementation, bytes memory data) public payable override {	// @audit-issue

118:    function proxiableUUID() external view override notDelegatedOrIsMinimalProxy returns (bytes32) {	// @audit-issue
```
[33](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L33-L33), [65](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L65-L65), [93](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L93-L93), [104](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L104-L104), [118](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L118-L118), 


#### Recommendation

In Solidity versions 0.8.8 and later, the `override` keyword is no longer required for functions that are solely overriding from an interface and not present in multiple base contracts. Removing the unnecessary `override` keyword can make the code cleaner and less verbose.

### Avoid external calls in modifiers
It is unusual to have external calls in modifiers, and doing so will make reviewers more likely to miss important external interactions. Consider moving the external call to an internal function, and calling that function from the modifier.

```solidity
Path: ./src/WellUpgradeable.sol

25:            address wellImplmentation = IAquifer(aquifer).wellImplementation(address(this));	// @audit-issue
```
[25](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L25-L25), 


#### Recommendation

Refrain from incorporating external calls directly within modifiers. Instead, encapsulate the external call within an internal function and invoke this function from within the body of the functions that use the modifier. This approach enhances code readability and security, making it easier for reviewers and auditors to track external interactions. Additionally, it centralizes external calls, simplifying the management and review of these potentially risky operations. Always ensure external calls are handled with care, implementing checks, balances, and reentrancy guards as necessary to protect your contract from malicious actors and unintended consequences.

### Consider making contracts `Upgradeable`
This allows for bugs to be fixed in production, at the expense of significantly increasing centralization.

```solidity
Path: ./src/functions/Stable2.sol

25:contract Stable2 is ProportionalLPToken2, IBeanstalkWellFunction {	// @audit-issue
```
[25](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L25-L25), 


```solidity
Path: ./src/functions/StableLUT/Stable2LUT1.sol

13:contract Stable2LUT1 is ILookupTable {	// @audit-issue
```
[13](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L13-L13), 


#### Recommendation

Assess the need for upgradeability in your Solidity contracts based on the project's requirements and lifecycle. If chosen, implement a well-known proxy pattern ensuring rigorous security and governance mechanisms are in place. Be aware of the increased centralization and plan accordingly to mitigate potential risks, such as through decentralized governance models or multi-sig control for upgrade decisions.

### Consider using descriptive `constants` when passing zero as a function argument
Passing zero as a function argument can sometimes result in a security issue (e.g. passing zero as the slippage parameter). Consider using a `constant` variable with a descriptive name, so it's clear that the argument is intentionally being used, and for the right reasons.

```solidity
Path: ./src/WellUpgradeable.sol

95:        _upgradeToAndCallUUPS(newImplementation, new bytes(0), false);	// @audit-issue
```
[95](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L95-L95), 


```solidity
Path: ./src/functions/StableLUT/Stable2LUT1.sol

38:                                        PriceData(0.27702e6, 0, 9.646293093274934449e18, 0.001083e6, 0, 2000e18, 1e18);	// @audit-issue

41:                                return PriceData(
42:                                    0.30624e6, 0, 8.612761690424049377e18, 0.27702e6, 0, 9.646293093274934449e18, 1e18	// @audit-issue
43:                                );

48:                                    return PriceData(
49:                                        0.337394e6,
50:                                        0,	// @audit-issue
51:                                        7.689965795021471706e18,
52:                                        0.30624e6,
53:                                        0,
54:                                        8.612761690424049377e18,
55:                                        1e18
56:                                    );

48:                                    return PriceData(
49:                                        0.337394e6,
50:                                        0,
51:                                        7.689965795021471706e18,
52:                                        0.30624e6,
53:                                        0,	// @audit-issue
54:                                        8.612761690424049377e18,
55:                                        1e18
56:                                    );

58:                                    return PriceData(
59:                                        0.370355e6,
60:                                        0,	// @audit-issue
61:                                        6.866040888412029197e18,
62:                                        0.337394e6,
63:                                        0,
64:                                        7.689965795021471706e18,
65:                                        1e18
66:                                    );

58:                                    return PriceData(
59:                                        0.370355e6,
60:                                        0,
61:                                        6.866040888412029197e18,
62:                                        0.337394e6,
63:                                        0,	// @audit-issue
64:                                        7.689965795021471706e18,
65:                                        1e18
66:                                    );

69:                                return PriceData(
70:                                    0.404944e6, 0, 6.130393650367882863e18, 0.370355e6, 0, 6.866040888412029197e18, 1e18	// @audit-issue
71:                                );

78:                                    return PriceData(
79:                                        0.440934e6,
80:                                        0,	// @audit-issue
81:                                        5.473565759257038366e18,
82:                                        0.404944e6,
83:                                        0,
84:                                        6.130393650367882863e18,
85:                                        1e18
86:                                    );

78:                                    return PriceData(
79:                                        0.440934e6,
80:                                        0,
81:                                        5.473565759257038366e18,
82:                                        0.404944e6,
83:                                        0,	// @audit-issue
84:                                        6.130393650367882863e18,
85:                                        1e18
86:                                    );

88:                                    return PriceData(
89:                                        0.478063e6,
90:                                        0,	// @audit-issue
91:                                        4.887112285050926097e18,
92:                                        0.440934e6,
93:                                        0,
94:                                        5.473565759257038366e18,
95:                                        1e18
96:                                    );

88:                                    return PriceData(
89:                                        0.478063e6,
90:                                        0,
91:                                        4.887112285050926097e18,
92:                                        0.440934e6,
93:                                        0,	// @audit-issue
94:                                        5.473565759257038366e18,
95:                                        1e18
96:                                    );

99:                                return PriceData(
100:                                    0.516039e6, 0, 4.363493111652613443e18, 0.478063e6, 0, 4.887112285050926097e18, 1e18	// @audit-issue
101:                                );

105:                                return PriceData(
106:                                    0.554558e6, 0, 3.89597599254697613e18, 0.516039e6, 0, 4.363493111652613443e18, 1e18	// @audit-issue
107:                                );

109:                                return PriceData(
110:                                    0.59332e6, 0, 3.478549993345514402e18, 0.554558e6, 0, 3.89597599254697613e18, 1e18	// @audit-issue
111:                                );

120:                                    return PriceData(
121:                                        0.632052e6,
122:                                        0,	// @audit-issue
123:                                        3.105848208344209382e18,
124:                                        0.59332e6,
125:                                        0,
126:                                        3.478549993345514402e18,
127:                                        1e18
128:                                    );

120:                                    return PriceData(
121:                                        0.632052e6,
122:                                        0,
123:                                        3.105848208344209382e18,
124:                                        0.59332e6,
125:                                        0,	// @audit-issue
126:                                        3.478549993345514402e18,
127:                                        1e18
128:                                    );

130:                                    return PriceData(
131:                                        0.670518e6,
132:                                        0,	// @audit-issue
133:                                        2.773078757450186949e18,
134:                                        0.632052e6,
135:                                        0,
136:                                        3.105848208344209382e18,
137:                                        1e18
138:                                    );

130:                                    return PriceData(
131:                                        0.670518e6,
132:                                        0,
133:                                        2.773078757450186949e18,
134:                                        0.632052e6,
135:                                        0,	// @audit-issue
136:                                        3.105848208344209382e18,
137:                                        1e18
138:                                    );

141:                                return PriceData(
142:                                    0.708539e6, 0, 2.475963176294809553e18, 0.670518e6, 0, 2.773078757450186949e18, 1e18	// @audit-issue
143:                                );

147:                                return PriceData(
148:                                    0.746003e6, 0, 2.210681407406080101e18, 0.708539e6, 0, 2.475963176294809553e18, 1e18	// @audit-issue
149:                                );

151:                                return PriceData(
152:                                    0.782874e6, 0, 1.973822685183999948e18, 0.746003e6, 0, 2.210681407406080101e18, 1e18	// @audit-issue
153:                                );

160:                                    return PriceData(
161:                                        0.819199e6,
162:                                        0,	// @audit-issue
163:                                        1.762341683200000064e18,
164:                                        0.782874e6,
165:                                        0,
166:                                        1.973822685183999948e18,
167:                                        1e18
168:                                    );

160:                                    return PriceData(
161:                                        0.819199e6,
162:                                        0,
163:                                        1.762341683200000064e18,
164:                                        0.782874e6,
165:                                        0,	// @audit-issue
166:                                        1.973822685183999948e18,
167:                                        1e18
168:                                    );

170:                                    return PriceData(
171:                                        0.855108e6,
172:                                        0,	// @audit-issue
173:                                        1.573519359999999923e18,
174:                                        0.819199e6,
175:                                        0,
176:                                        1.762341683200000064e18,
177:                                        1e18
178:                                    );

170:                                    return PriceData(
171:                                        0.855108e6,
172:                                        0,
173:                                        1.573519359999999923e18,
174:                                        0.819199e6,
175:                                        0,	// @audit-issue
176:                                        1.762341683200000064e18,
177:                                        1e18
178:                                    );

181:                                return PriceData(
182:                                    0.873157e6, 0, 1.485947395978354457e18, 0.855108e6, 0, 1.573519359999999923e18, 1e18	// @audit-issue
183:                                );

187:                                return PriceData(
188:                                    0.879393e6, 0, 1.456811172527798348e18, 0.873157e6, 0, 1.485947395978354457e18, 1e18	// @audit-issue
189:                                );

191:                                return PriceData(
192:                                    0.885627e6, 0, 1.428246247576273165e18, 0.879393e6, 0, 1.456811172527798348e18, 1e18	// @audit-issue
193:                                );

204:                                    return PriceData(
205:                                        0.89081e6,
206:                                        0,	// @audit-issue
207:                                        1.404927999999999955e18,
208:                                        0.885627e6,
209:                                        0,
210:                                        1.428246247576273165e18,
211:                                        1e18
212:                                    );

204:                                    return PriceData(
205:                                        0.89081e6,
206:                                        0,
207:                                        1.404927999999999955e18,
208:                                        0.885627e6,
209:                                        0,	// @audit-issue
210:                                        1.428246247576273165e18,
211:                                        1e18
212:                                    );

214:                                    return PriceData(
215:                                        0.891863e6,
216:                                        0,	// @audit-issue
217:                                        1.400241419192424397e18,
218:                                        0.89081e6,
219:                                        0,
220:                                        1.404927999999999955e18,
221:                                        1e18
222:                                    );

214:                                    return PriceData(
215:                                        0.891863e6,
216:                                        0,
217:                                        1.400241419192424397e18,
218:                                        0.89081e6,
219:                                        0,	// @audit-issue
220:                                        1.404927999999999955e18,
221:                                        1e18
222:                                    );

225:                                return PriceData(
226:                                    0.898101e6, 0, 1.372785705090612263e18, 0.891863e6, 0, 1.400241419192424397e18, 1e18	// @audit-issue
227:                                );

232:                                    return PriceData(
233:                                        0.904344e6,
234:                                        0,	// @audit-issue
235:                                        1.345868338324129665e18,
236:                                        0.898101e6,
237:                                        0,
238:                                        1.372785705090612263e18,
239:                                        1e18
240:                                    );

232:                                    return PriceData(
233:                                        0.904344e6,
234:                                        0,
235:                                        1.345868338324129665e18,
236:                                        0.898101e6,
237:                                        0,	// @audit-issue
238:                                        1.372785705090612263e18,
239:                                        1e18
240:                                    );

242:                                    return PriceData(
243:                                        0.910594e6,
244:                                        0,	// @audit-issue
245:                                        1.319478763062872151e18,
246:                                        0.904344e6,
247:                                        0,
248:                                        1.345868338324129665e18,
249:                                        1e18
250:                                    );

242:                                    return PriceData(
243:                                        0.910594e6,
244:                                        0,
245:                                        1.319478763062872151e18,
246:                                        0.904344e6,
247:                                        0,	// @audit-issue
248:                                        1.345868338324129665e18,
249:                                        1e18
250:                                    );

253:                                return PriceData(
254:                                    0.916852e6, 0, 1.293606630453796313e18, 0.910594e6, 0, 1.319478763062872151e18, 1e18	// @audit-issue
255:                                );

262:                                    return PriceData(
263:                                        0.92312e6,
264:                                        0,	// @audit-issue
265:                                        1.268241794562545266e18,
266:                                        0.916852e6,
267:                                        0,
268:                                        1.293606630453796313e18,
269:                                        1e18
270:                                    );

262:                                    return PriceData(
263:                                        0.92312e6,
264:                                        0,
265:                                        1.268241794562545266e18,
266:                                        0.916852e6,
267:                                        0,	// @audit-issue
268:                                        1.293606630453796313e18,
269:                                        1e18
270:                                    );

272:                                    return PriceData(
273:                                        0.9266e6,
274:                                        0,	// @audit-issue
275:                                        1.254399999999999959e18,
276:                                        0.92312e6,
277:                                        0,
278:                                        1.268241794562545266e18,
279:                                        1e18
280:                                    );

272:                                    return PriceData(
273:                                        0.9266e6,
274:                                        0,
275:                                        1.254399999999999959e18,
276:                                        0.92312e6,
277:                                        0,	// @audit-issue
278:                                        1.268241794562545266e18,
279:                                        1e18
280:                                    );

283:                                return PriceData(
284:                                    0.929402e6, 0, 1.243374308394652239e18, 0.9266e6, 0, 1.254399999999999959e18, 1e18	// @audit-issue
285:                                );

289:                                return PriceData(
290:                                    0.935697e6, 0, 1.218994419994757328e18, 0.929402e6, 0, 1.243374308394652239e18, 1e18	// @audit-issue
291:                                );

293:                                return PriceData(
294:                                    0.94201e6, 0, 1.195092568622310836e18, 0.935697e6, 0, 1.218994419994757328e18, 1e18	// @audit-issue
295:                                );

304:                                    return PriceData(
305:                                        0.948343e6,
306:                                        0,	// @audit-issue
307:                                        1.171659381002265521e18,
308:                                        0.94201e6,
309:                                        0,
310:                                        1.195092568622310836e18,
311:                                        1e18
312:                                    );

304:                                    return PriceData(
305:                                        0.948343e6,
306:                                        0,
307:                                        1.171659381002265521e18,
308:                                        0.94201e6,
309:                                        0,	// @audit-issue
310:                                        1.195092568622310836e18,
311:                                        1e18
312:                                    );

314:                                    return PriceData(
315:                                        0.954697e6,
316:                                        0,	// @audit-issue
317:                                        1.14868566764928004e18,
318:                                        0.948343e6,
319:                                        0,
320:                                        1.171659381002265521e18,
321:                                        1e18
322:                                    );

314:                                    return PriceData(
315:                                        0.954697e6,
316:                                        0,
317:                                        1.14868566764928004e18,
318:                                        0.948343e6,
319:                                        0,	// @audit-issue
320:                                        1.171659381002265521e18,
321:                                        1e18
322:                                    );

325:                                return PriceData(
326:                                    0.961075e6, 0, 1.12616241926400007e18, 0.954697e6, 0, 1.14868566764928004e18, 1e18	// @audit-issue
327:                                );

331:                                return PriceData(
332:                                    0.962847e6, 0, 1.120000000000000107e18, 0.961075e6, 0, 1.12616241926400007e18, 1e18	// @audit-issue
333:                                );

335:                                return PriceData(
336:                                    0.96748e6, 0, 1.104080803200000016e18, 0.962847e6, 0, 1.120000000000000107e18, 1e18	// @audit-issue
337:                                );

344:                                    return PriceData(
345:                                        0.973914e6,
346:                                        0,	// @audit-issue
347:                                        1.082432159999999977e18,
348:                                        0.96748e6,
349:                                        0,
350:                                        1.104080803200000016e18,
351:                                        1e18
352:                                    );

344:                                    return PriceData(
345:                                        0.973914e6,
346:                                        0,
347:                                        1.082432159999999977e18,
348:                                        0.96748e6,
349:                                        0,	// @audit-issue
350:                                        1.104080803200000016e18,
351:                                        1e18
352:                                    );

354:                                    return PriceData(
355:                                        0.98038e6,
356:                                        0,	// @audit-issue
357:                                        1.061208000000000151e18,
358:                                        0.973914e6,
359:                                        0,
360:                                        1.082432159999999977e18,
361:                                        1e18
362:                                    );

354:                                    return PriceData(
355:                                        0.98038e6,
356:                                        0,
357:                                        1.061208000000000151e18,
358:                                        0.973914e6,
359:                                        0,	// @audit-issue
360:                                        1.082432159999999977e18,
361:                                        1e18
362:                                    );

365:                                return PriceData(
366:                                    0.986882e6, 0, 1.040399999999999991e18, 0.98038e6, 0, 1.061208000000000151e18, 1e18	// @audit-issue
367:                                );

371:                                return PriceData(
372:                                    0.993421e6, 0, 1.020000000000000018e18, 0.986882e6, 0, 1.040399999999999991e18, 1e18	// @audit-issue
373:                                );

375:                                return PriceData(
376:                                    1.006758e6, 0, 0.980000000000000093e18, 0.993421e6, 0, 1.020000000000000018e18, 1e18	// @audit-issue
377:                                );

390:                                    return PriceData(
391:                                        1.013564e6,
392:                                        0,	// @audit-issue
393:                                        0.960400000000000031e18,
394:                                        1.006758e6,
395:                                        0,
396:                                        0.980000000000000093e18,
397:                                        1e18
398:                                    );

390:                                    return PriceData(
391:                                        1.013564e6,
392:                                        0,
393:                                        0.960400000000000031e18,
394:                                        1.006758e6,
395:                                        0,	// @audit-issue
396:                                        0.980000000000000093e18,
397:                                        1e18
398:                                    );

400:                                    return PriceData(
401:                                        1.020422e6,
402:                                        0,	// @audit-issue
403:                                        0.941192000000000029e18,
404:                                        1.013564e6,
405:                                        0,
406:                                        0.960400000000000031e18,
407:                                        1e18
408:                                    );

400:                                    return PriceData(
401:                                        1.020422e6,
402:                                        0,
403:                                        0.941192000000000029e18,
404:                                        1.013564e6,
405:                                        0,	// @audit-issue
406:                                        0.960400000000000031e18,
407:                                        1e18
408:                                    );

411:                                return PriceData(
412:                                    1.027335e6, 0, 0.922368159999999992e18, 1.020422e6, 0, 0.941192000000000029e18, 1e18	// @audit-issue
413:                                );

418:                                    return PriceData(
419:                                        1.034307e6,
420:                                        0,	// @audit-issue
421:                                        0.903920796799999926e18,
422:                                        1.027335e6,
423:                                        0,
424:                                        0.922368159999999992e18,
425:                                        1e18
426:                                    );

418:                                    return PriceData(
419:                                        1.034307e6,
420:                                        0,
421:                                        0.903920796799999926e18,
422:                                        1.027335e6,
423:                                        0,	// @audit-issue
424:                                        0.922368159999999992e18,
425:                                        1e18
426:                                    );

428:                                    return PriceData(
429:                                        1.041342e6,
430:                                        0,	// @audit-issue
431:                                        0.885842380864000023e18,
432:                                        1.034307e6,
433:                                        0,
434:                                        0.903920796799999926e18,
435:                                        1e18
436:                                    );

428:                                    return PriceData(
429:                                        1.041342e6,
430:                                        0,
431:                                        0.885842380864000023e18,
432:                                        1.034307e6,
433:                                        0,	// @audit-issue
434:                                        0.903920796799999926e18,
435:                                        1e18
436:                                    );

439:                                return PriceData(
440:                                    1.04366e6, 0, 0.880000000000000004e18, 1.041342e6, 0, 0.885842380864000023e18, 1e18	// @audit-issue
441:                                );

448:                                    return PriceData(
449:                                        1.048443e6,
450:                                        0,	// @audit-issue
451:                                        0.868125533246720038e18,
452:                                        1.04366e6,
453:                                        0,
454:                                        0.880000000000000004e18,
455:                                        1e18
456:                                    );

448:                                    return PriceData(
449:                                        1.048443e6,
450:                                        0,
451:                                        0.868125533246720038e18,
452:                                        1.04366e6,
453:                                        0,	// @audit-issue
454:                                        0.880000000000000004e18,
455:                                        1e18
456:                                    );

458:                                    return PriceData(
459:                                        1.055613e6,
460:                                        0,	// @audit-issue
461:                                        0.8507630225817856e18,
462:                                        1.048443e6,
463:                                        0,
464:                                        0.868125533246720038e18,
465:                                        1e18
466:                                    );

458:                                    return PriceData(
459:                                        1.055613e6,
460:                                        0,
461:                                        0.8507630225817856e18,
462:                                        1.048443e6,
463:                                        0,	// @audit-issue
464:                                        0.868125533246720038e18,
465:                                        1e18
466:                                    );

469:                                return PriceData(
470:                                    1.062857e6, 0, 0.833747762130149894e18, 1.055613e6, 0, 0.8507630225817856e18, 1e18	// @audit-issue
471:                                );

475:                                return PriceData(
476:                                    1.070179e6, 0, 0.81707280688754691e18, 1.062857e6, 0, 0.833747762130149894e18, 1e18	// @audit-issue
477:                                );

479:                                return PriceData(
480:                                    1.077582e6, 0, 0.800731350749795956e18, 1.070179e6, 0, 0.81707280688754691e18, 1e18	// @audit-issue
481:                                );

490:                                    return PriceData(
491:                                        1.085071e6,
492:                                        0,	// @audit-issue
493:                                        0.784716723734800059e18,
494:                                        1.077582e6,
495:                                        0,
496:                                        0.800731350749795956e18,
497:                                        1e18
498:                                    );

490:                                    return PriceData(
491:                                        1.085071e6,
492:                                        0,
493:                                        0.784716723734800059e18,
494:                                        1.077582e6,
495:                                        0,	// @audit-issue
496:                                        0.800731350749795956e18,
497:                                        1e18
498:                                    );

500:                                    return PriceData(
501:                                        1.090025e6,
502:                                        0,	// @audit-issue
503:                                        0.774399999999999977e18,
504:                                        1.085071e6,
505:                                        0,
506:                                        0.784716723734800059e18,
507:                                        1e18
508:                                    );

500:                                    return PriceData(
501:                                        1.090025e6,
502:                                        0,
503:                                        0.774399999999999977e18,
504:                                        1.085071e6,
505:                                        0,	// @audit-issue
506:                                        0.784716723734800059e18,
507:                                        1e18
508:                                    );

511:                                return PriceData(
512:                                    1.09265e6, 0, 0.769022389260104022e18, 1.090025e6, 0, 0.774399999999999977e18, 1e18	// @audit-issue
513:                                );

517:                                return PriceData(
518:                                    1.100323e6, 0, 0.753641941474902044e18, 1.09265e6, 0, 0.769022389260104022e18, 1e18	// @audit-issue
519:                                );

521:                                return PriceData(
522:                                    1.108094e6, 0, 0.738569102645403985e18, 1.100323e6, 0, 0.753641941474902044e18, 1e18	// @audit-issue
523:                                );

530:                                    return PriceData(
531:                                        1.115967e6,
532:                                        0,	// @audit-issue
533:                                        0.723797720592495919e18,
534:                                        1.108094e6,
535:                                        0,
536:                                        0.738569102645403985e18,
537:                                        1e18
538:                                    );

530:                                    return PriceData(
531:                                        1.115967e6,
532:                                        0,
533:                                        0.723797720592495919e18,
534:                                        1.108094e6,
535:                                        0,	// @audit-issue
536:                                        0.738569102645403985e18,
537:                                        1e18
538:                                    );

540:                                    return PriceData(
541:                                        1.123949e6,
542:                                        0,	// @audit-issue
543:                                        0.709321766180645907e18,
544:                                        1.115967e6,
545:                                        0,
546:                                        0.723797720592495919e18,
547:                                        1e18
548:                                    );

540:                                    return PriceData(
541:                                        1.123949e6,
542:                                        0,
543:                                        0.709321766180645907e18,
544:                                        1.115967e6,
545:                                        0,	// @audit-issue
546:                                        0.723797720592495919e18,
547:                                        1e18
548:                                    );

551:                                return PriceData(
552:                                    1.132044e6, 0, 0.695135330857033051e18, 1.123949e6, 0, 0.709321766180645907e18, 1e18	// @audit-issue
553:                                );

557:                                return PriceData(
558:                                    1.14011e6, 0, 0.681471999999999967e18, 1.132044e6, 0, 0.695135330857033051e18, 1e18	// @audit-issue
559:                                );

561:                                return PriceData(
562:                                    1.140253e6, 0, 0.681232624239892393e18, 1.14011e6, 0, 0.681471999999999967e18, 1e18	// @audit-issue
563:                                );

574:                                    return PriceData(
575:                                        1.148586e6,
576:                                        0,	// @audit-issue
577:                                        0.667607971755094454e18,
578:                                        1.140253e6,
579:                                        0,
580:                                        0.681232624239892393e18,
581:                                        1e18
582:                                    );

574:                                    return PriceData(
575:                                        1.148586e6,
576:                                        0,
577:                                        0.667607971755094454e18,
578:                                        1.140253e6,
579:                                        0,	// @audit-issue
580:                                        0.681232624239892393e18,
581:                                        1e18
582:                                    );

584:                                    return PriceData(
585:                                        1.195079e6,
586:                                        0,	// @audit-issue
587:                                        0.599695360000000011e18,
588:                                        1.148586e6,
589:                                        0,
590:                                        0.667607971755094454e18,
591:                                        1e18
592:                                    );

584:                                    return PriceData(
585:                                        1.195079e6,
586:                                        0,
587:                                        0.599695360000000011e18,
588:                                        1.148586e6,
589:                                        0,	// @audit-issue
590:                                        0.667607971755094454e18,
591:                                        1e18
592:                                    );

595:                                return PriceData(
596:                                    1.256266e6, 0, 0.527731916799999978e18, 1.195079e6, 0, 0.599695360000000011e18, 1e18	// @audit-issue
597:                                );

601:                                return PriceData(
602:                                    1.325188e6, 0, 0.464404086784000025e18, 1.256266e6, 0, 0.527731916799999978e18, 1e18	// @audit-issue
603:                                );

605:                                return PriceData(
606:                                    1.403579e6, 0, 0.408675596369920013e18, 1.325188e6, 0, 0.464404086784000025e18, 1e18	// @audit-issue
607:                                );

614:                                    return PriceData(
615:                                        1.493424e6,
616:                                        0,	// @audit-issue
617:                                        0.359634524805529598e18,
618:                                        1.403579e6,
619:                                        0,
620:                                        0.408675596369920013e18,
621:                                        1e18
622:                                    );

614:                                    return PriceData(
615:                                        1.493424e6,
616:                                        0,
617:                                        0.359634524805529598e18,
618:                                        1.403579e6,
619:                                        0,	// @audit-issue
620:                                        0.408675596369920013e18,
621:                                        1e18
622:                                    );

624:                                    return PriceData(
625:                                        1.596984e6,
626:                                        0,	// @audit-issue
627:                                        0.316478381828866062e18,
628:                                        1.493424e6,
629:                                        0,
630:                                        0.359634524805529598e18,
631:                                        1e18
632:                                    );

624:                                    return PriceData(
625:                                        1.596984e6,
626:                                        0,
627:                                        0.316478381828866062e18,
628:                                        1.493424e6,
629:                                        0,	// @audit-issue
630:                                        0.359634524805529598e18,
631:                                        1e18
632:                                    );

635:                                return PriceData(
636:                                    1.716848e6, 0, 0.278500976009402101e18, 1.596984e6, 0, 0.316478381828866062e18, 1e18	// @audit-issue
637:                                );

641:                                return PriceData(
642:                                    1.855977e6, 0, 0.245080858888273884e18, 1.716848e6, 0, 0.278500976009402101e18, 1e18	// @audit-issue
643:                                );

645:                                return PriceData(
646:                                    2.01775e6, 0, 0.215671155821681004e18, 1.855977e6, 0, 0.245080858888273884e18, 1e18	// @audit-issue
647:                                );

656:                                    return PriceData(
657:                                        2.206036e6,
658:                                        0,	// @audit-issue
659:                                        0.189790617123079292e18,
660:                                        2.01775e6,
661:                                        0,
662:                                        0.215671155821681004e18,
663:                                        1e18
664:                                    );

656:                                    return PriceData(
657:                                        2.206036e6,
658:                                        0,
659:                                        0.189790617123079292e18,
660:                                        2.01775e6,
661:                                        0,	// @audit-issue
662:                                        0.215671155821681004e18,
663:                                        1e18
664:                                    );

666:                                    return PriceData(
667:                                        2.425256e6,
668:                                        0,	// @audit-issue
669:                                        0.167015743068309769e18,
670:                                        2.206036e6,
671:                                        0,
672:                                        0.189790617123079292e18,
673:                                        1e18
674:                                    );

666:                                    return PriceData(
667:                                        2.425256e6,
668:                                        0,
669:                                        0.167015743068309769e18,
670:                                        2.206036e6,
671:                                        0,	// @audit-issue
672:                                        0.189790617123079292e18,
673:                                        1e18
674:                                    );

677:                                return PriceData(
678:                                    2.680458e6, 0, 0.146973853900112583e18, 2.425256e6, 0, 0.167015743068309769e18, 1e18	// @audit-issue
679:                                );

683:                                return PriceData(
684:                                    2.977411e6, 0, 0.129336991432099091e18, 2.680458e6, 0, 0.146973853900112583e18, 1e18	// @audit-issue
685:                                );

687:                                return PriceData(
688:                                    3.322705e6, 0, 0.113816552460247203e18, 2.977411e6, 0, 0.129336991432099091e18, 1e18	// @audit-issue
689:                                );

696:                                    return PriceData(
697:                                        3.723858e6,
698:                                        0,	// @audit-issue
699:                                        0.100158566165017532e18,
700:                                        3.322705e6,
701:                                        0,
702:                                        0.113816552460247203e18,
703:                                        1e18
704:                                    );

696:                                    return PriceData(
697:                                        3.723858e6,
698:                                        0,
699:                                        0.100158566165017532e18,
700:                                        3.322705e6,
701:                                        0,	// @audit-issue
702:                                        0.113816552460247203e18,
703:                                        1e18
704:                                    );

706:                                    return PriceData(
707:                                        4.189464e6,
708:                                        0,	// @audit-issue
709:                                        0.088139538225215433e18,
710:                                        3.723858e6,
711:                                        0,
712:                                        0.100158566165017532e18,
713:                                        1e18
714:                                    );

706:                                    return PriceData(
707:                                        4.189464e6,
708:                                        0,
709:                                        0.088139538225215433e18,
710:                                        3.723858e6,
711:                                        0,	// @audit-issue
712:                                        0.100158566165017532e18,
713:                                        1e18
714:                                    );

717:                                return PriceData(
718:                                    4.729321e6, 0, 0.077562793638189589e18, 4.189464e6, 0, 0.088139538225215433e18, 1e18	// @audit-issue
719:                                );

723:                                return PriceData(
724:                                    10.37089e6, 0, 0.035714285714285712e18, 4.729321e6, 0, 0.077562793638189589e18, 1e18	// @audit-issue
725:                                );
```
[38](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L38-L38), [42](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L41-L43), [50](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L48-L56), [53](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L48-L56), [60](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L58-L66), [63](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L58-L66), [70](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L69-L71), [80](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L78-L86), [83](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L78-L86), [90](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L88-L96), [93](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L88-L96), [100](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L99-L101), [106](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L105-L107), [110](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L109-L111), [122](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L120-L128), [125](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L120-L128), [132](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L130-L138), [135](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L130-L138), [142](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L141-L143), [148](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L147-L149), [152](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L151-L153), [162](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L160-L168), [165](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L160-L168), [172](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L170-L178), [175](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L170-L178), [182](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L181-L183), [188](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L187-L189), [192](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L191-L193), [206](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L204-L212), [209](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L204-L212), [216](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L214-L222), [219](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L214-L222), [226](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L225-L227), [234](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L232-L240), [237](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L232-L240), [244](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L242-L250), [247](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L242-L250), [254](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L253-L255), [264](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L262-L270), [267](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L262-L270), [274](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L272-L280), [277](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L272-L280), [284](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L283-L285), [290](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L289-L291), [294](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L293-L295), [306](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L304-L312), [309](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L304-L312), [316](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L314-L322), [319](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L314-L322), [326](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L325-L327), [332](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L331-L333), [336](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L335-L337), [346](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L344-L352), [349](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L344-L352), [356](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L354-L362), [359](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L354-L362), [366](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L365-L367), [372](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L371-L373), [376](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L375-L377), [392](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L390-L398), [395](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L390-L398), [402](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L400-L408), [405](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L400-L408), [412](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L411-L413), [420](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L418-L426), [423](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L418-L426), [430](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L428-L436), [433](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L428-L436), [440](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L439-L441), [450](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L448-L456), [453](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L448-L456), [460](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L458-L466), [463](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L458-L466), [470](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L469-L471), [476](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L475-L477), [480](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L479-L481), [492](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L490-L498), [495](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L490-L498), [502](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L500-L508), [505](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L500-L508), [512](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L511-L513), [518](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L517-L519), [522](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L521-L523), [532](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L530-L538), [535](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L530-L538), [542](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L540-L548), [545](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L540-L548), [552](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L551-L553), [558](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L557-L559), [562](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L561-L563), [576](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L574-L582), [579](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L574-L582), [586](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L584-L592), [589](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L584-L592), [596](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L595-L597), [602](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L601-L603), [606](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L605-L607), [616](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L614-L622), [619](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L614-L622), [626](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L624-L632), [629](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L624-L632), [636](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L635-L637), [642](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L641-L643), [646](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L645-L647), [658](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L656-L664), [661](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L656-L664), [668](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L666-L674), [671](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L666-L674), [678](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L677-L679), [684](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L683-L685), [688](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L687-L689), [698](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L696-L704), [701](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L696-L704), [708](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L706-L714), [711](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L706-L714), [718](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L717-L719), [724](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L723-L725), 


#### Recommendation

Replace direct usage of zero as a function argument with a well-named `constant`. For example, use something like `uint256 constant NO_SLIPPAGE = 0;` when zero is intended to signify 'no slippage'. This approach enhances code readability, reduces ambiguity, and helps ensure that the function is used correctly and for its intended purpose.

### Non-`external`/`public` function names should begin with an underscore
According to the Solidity Style Guide, Non-external/public function names should begin with an [underscore](https://docs.soliditylang.org/en/latest/style-guide.html#underscore-prefix-for-non-external-functions-and-variables)


```solidity
Path: ./src/functions/Stable2.sol

357:    function getScaledReserves(	// @audit-issue
358:        uint256[] memory reserves,
359:        uint256[] memory decimals
360:    ) internal pure returns (uint256[] memory scaledReserves) {

375:    function getBandC(	// @audit-issue
376:        uint256 Ann,
377:        uint256 lpTokenSupply,
378:        uint256 reserves
379:    ) private pure returns (uint256 c, uint256 b) {

387:    function updateReserve(PriceData memory pd, uint256 reserve) internal pure returns (uint256) {	// @audit-issue
```
[357](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L357-L360), [375](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L375-L379), [387](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L387-L387), 


#### Recommendation

To adhere to the Solidity Style Guide, consider prefixing the names of non-`external`/`public` functions with an underscore (_). This naming convention enhances code readability and helps distinguish the visibility of functions.

### Variable names for `immutable`s should use CONSTANT_CASE
For `immutable` variable names, each word should use all capital letters, with underscores separating each word (CONSTANT_CASE)

```solidity
Path: ./src/WellUpgradeable.sol

17:    address private immutable ___self = address(this);	// @audit-issue name should be: _SELF
```
[17](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L17-L17), 


```solidity
Path: ./src/functions/Stable2.sol

46:    address immutable lookupTable;	// @audit-issue name should be: LOOKUP_TABLE

47:    uint256 immutable a;	// @audit-issue name should be: A
```
[46](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L46-L46), [47](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L47-L47), 


#### Recommendation

When naming `immutable` variables, follow the CONSTANT_CASE convention, which means using all capital letters with underscores to separate words. This naming convention improves code readability and aligns with best practices for naming constants.

### Revert statements within external and public functions can be used to perform DOS attacks
In Solidity, `revert` statements are used to undo changes and throw an exception when certain conditions are not met. However, in public and external functions, improper use of `revert` can be exploited for Denial of Service (DoS) attacks. An attacker can intentionally trigger these `revert' conditions, causing legitimate transactions to consistently fail. For example, if a function relies on specific conditions from user input or contract state, an attacker could manipulate these to continually force `revert`s, blocking the function's execution. Therefore, it's crucial to design contract logic to handle exceptions properly and avoid scenarios where `revert` can be predictably triggered by malicious actors. This includes careful input validation and considering alternative design patterns that are less susceptible to such abuses.

```solidity
Path: ./src/WellUpgradeable.sol

45:                    revert DuplicateTokens(_tokens[i]);	// @audit-issue
```
[45](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L45-L45), 


```solidity
Path: ./src/functions/Stable2.sol

320:        if (decimal0 > 18 || decimal1 > 18) revert InvalidTokenDecimals();	// @audit-issue
```
[320](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L320-L320), 


```solidity
Path: ./src/functions/StableLUT/Stable2LUT1.sol

35:                                    revert("LUT: Invalid price");	// @audit-issue

727:                                revert("LUT: Invalid price");	// @audit-issue

748:                                    revert("LUT: Invalid price");	// @audit-issue

2167:                                revert("LUT: Invalid price");	// @audit-issue
```
[35](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L35-L35), [727](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L727-L727), [748](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L748-L748), [2167](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L2167-L2167), 


#### Recommendation

Design your Solidity contract's public and external functions with care to mitigate the risk of DoS attacks via `revert` statements. Implement robust input validation to ensure inputs are within expected bounds and conditions. Consider alternative logic or design patterns that reduce the reliance on `revert` for critical operations, particularly those that can be influenced externally. Evaluate the use of modifiers, try-catch blocks, or state checks that allow for safer handling of conditions and exceptions. Ensure that your contract's critical functionality remains accessible and resilient against potential abuse of `revert` behavior by malicious actors.

### Consider adding emergency-stop functionality
In the event of a security breach or any unforeseen emergency, swiftly suspending all protocol operations becomes crucial. Having a mechanism in place to halt all functions collectively, instead of pausing individual contracts separately, substantially enhances the efficiency of mitigating ongoing attacks or vulnerabilities. This not only quickens the response time to potential threats but also reduces operational stress during these critical periods. Therefore, consider integrating a 'circuit breaker' or 'emergency stop' function into the smart contract system architecture. Such a feature would provide the capability to suspend the entire protocol instantly, which could prove invaluable during a time-sensitive crisis management situation.

```solidity
Path: ./src/WellUpgradeable.sol

16:contract WellUpgradeable is Well, UUPSUpgradeable, OwnableUpgradeable {	// @audit-issue
```
[16](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L16-L16), 


```solidity
Path: ./src/functions/Stable2.sol

25:contract Stable2 is ProportionalLPToken2, IBeanstalkWellFunction {	// @audit-issue
```
[25](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L25-L25), 


```solidity
Path: ./src/functions/StableLUT/Stable2LUT1.sol

13:contract Stable2LUT1 is ILookupTable {	// @audit-issue
```
[13](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L13-L13), 


#### Recommendation

Implement an emergency-stop feature in your Solidity contract system to enhance security and crisis response capabilities. This can be achieved through a 'circuit breaker' pattern, where a central switch or set of conditions can instantly suspend critical operations across the contract ecosystem. Ensure that this mechanism is accessible to authorized parties only, such as contract administrators or a decentralized governance system. Design the emergency-stop functionality to be transparent and auditable, with clear conditions and processes for activation and deactivation. Regularly test and audit this feature to ensure its reliability and effectiveness in potential emergency situations.

### Cyclomatic complexity in functions
Cyclomatic complexity is a software metric used to measure the complexity of a program. It quantifies the number of linearly independent paths through a program's source code, giving an idea of how complex the control flow is. High cyclomatic complexity may indicate a higher risk of defects and can make the code harder to understand, test, and maintain. It often suggests that a function or method is trying to do too much, and a refactor might be needed. By breaking down complex functions into smaller, more focused pieces, you can improve readability, ease of testing, and overall maintainability.

```solidity
Path: ./src/functions/StableLUT/Stable2LUT1.sol

27:    function getRatiosFromPriceLiquidity(uint256 price) external pure returns (PriceData memory) {	// @audit-issue
28:        if (price < 1.006758e6) {
29:            if (price < 0.885627e6) {
30:                if (price < 0.59332e6) {
31:                    if (price < 0.404944e6) {
32:                        if (price < 0.30624e6) {
33:                            if (price < 0.27702e6) {
34:                                if (price < 0.001083e6) {
35:                                    revert("LUT: Invalid price");
36:                                } else {
37:                                    return
38:                                        PriceData(0.27702e6, 0, 9.646293093274934449e18, 0.001083e6, 0, 2000e18, 1e18);
39:                                }
40:                            } else {
41:                                return PriceData(
42:                                    0.30624e6, 0, 8.612761690424049377e18, 0.27702e6, 0, 9.646293093274934449e18, 1e18
43:                                );
44:                            }
45:                        } else {
46:                            if (price < 0.370355e6) {
47:                                if (price < 0.337394e6) {
48:                                    return PriceData(
49:                                        0.337394e6,
50:                                        0,
51:                                        7.689965795021471706e18,
52:                                        0.30624e6,
53:                                        0,
54:                                        8.612761690424049377e18,
55:                                        1e18
56:                                    );
57:                                } else {
58:                                    return PriceData(
59:                                        0.370355e6,
60:                                        0,
61:                                        6.866040888412029197e18,
62:                                        0.337394e6,
63:                                        0,
64:                                        7.689965795021471706e18,
65:                                        1e18
66:                                    );
67:                                }
68:                            } else {
69:                                return PriceData(
70:                                    0.404944e6, 0, 6.130393650367882863e18, 0.370355e6, 0, 6.866040888412029197e18, 1e18
71:                                );
72:                            }
73:                        }
74:                    } else {
75:                        if (price < 0.516039e6) {
76:                            if (price < 0.478063e6) {
77:                                if (price < 0.440934e6) {
78:                                    return PriceData(
79:                                        0.440934e6,
80:                                        0,
81:                                        5.473565759257038366e18,
82:                                        0.404944e6,
83:                                        0,
84:                                        6.130393650367882863e18,
85:                                        1e18
86:                                    );
87:                                } else {
88:                                    return PriceData(
89:                                        0.478063e6,
90:                                        0,
91:                                        4.887112285050926097e18,
92:                                        0.440934e6,
93:                                        0,
94:                                        5.473565759257038366e18,
95:                                        1e18
96:                                    );
97:                                }
98:                            } else {
99:                                return PriceData(
100:                                    0.516039e6, 0, 4.363493111652613443e18, 0.478063e6, 0, 4.887112285050926097e18, 1e18
101:                                );
102:                            }
103:                        } else {
104:                            if (price < 0.554558e6) {
105:                                return PriceData(
106:                                    0.554558e6, 0, 3.89597599254697613e18, 0.516039e6, 0, 4.363493111652613443e18, 1e18
107:                                );
108:                            } else {
109:                                return PriceData(
110:                                    0.59332e6, 0, 3.478549993345514402e18, 0.554558e6, 0, 3.89597599254697613e18, 1e18
111:                                );
112:                            }
113:                        }
114:                    }
115:                } else {
116:                    if (price < 0.782874e6) {
117:                        if (price < 0.708539e6) {
118:                            if (price < 0.670518e6) {
119:                                if (price < 0.632052e6) {
120:                                    return PriceData(
121:                                        0.632052e6,
122:                                        0,
123:                                        3.105848208344209382e18,
124:                                        0.59332e6,
125:                                        0,
126:                                        3.478549993345514402e18,
127:                                        1e18
128:                                    );
129:                                } else {
130:                                    return PriceData(
131:                                        0.670518e6,
132:                                        0,
133:                                        2.773078757450186949e18,
134:                                        0.632052e6,
135:                                        0,
136:                                        3.105848208344209382e18,
137:                                        1e18
138:                                    );
139:                                }
140:                            } else {
141:                                return PriceData(
142:                                    0.708539e6, 0, 2.475963176294809553e18, 0.670518e6, 0, 2.773078757450186949e18, 1e18
143:                                );
144:                            }
145:                        } else {
146:                            if (price < 0.746003e6) {
147:                                return PriceData(
148:                                    0.746003e6, 0, 2.210681407406080101e18, 0.708539e6, 0, 2.475963176294809553e18, 1e18
149:                                );
150:                            } else {
151:                                return PriceData(
152:                                    0.782874e6, 0, 1.973822685183999948e18, 0.746003e6, 0, 2.210681407406080101e18, 1e18
153:                                );
154:                            }
155:                        }
156:                    } else {
157:                        if (price < 0.873157e6) {
158:                            if (price < 0.855108e6) {
159:                                if (price < 0.819199e6) {
160:                                    return PriceData(
161:                                        0.819199e6,
162:                                        0,
163:                                        1.762341683200000064e18,
164:                                        0.782874e6,
165:                                        0,
166:                                        1.973822685183999948e18,
167:                                        1e18
168:                                    );
169:                                } else {
170:                                    return PriceData(
171:                                        0.855108e6,
172:                                        0,
173:                                        1.573519359999999923e18,
174:                                        0.819199e6,
175:                                        0,
176:                                        1.762341683200000064e18,
177:                                        1e18
178:                                    );
179:                                }
180:                            } else {
181:                                return PriceData(
182:                                    0.873157e6, 0, 1.485947395978354457e18, 0.855108e6, 0, 1.573519359999999923e18, 1e18
183:                                );
184:                            }
185:                        } else {
186:                            if (price < 0.879393e6) {
187:                                return PriceData(
188:                                    0.879393e6, 0, 1.456811172527798348e18, 0.873157e6, 0, 1.485947395978354457e18, 1e18
189:                                );
190:                            } else {
191:                                return PriceData(
192:                                    0.885627e6, 0, 1.428246247576273165e18, 0.879393e6, 0, 1.456811172527798348e18, 1e18
193:                                );
194:                            }
195:                        }
196:                    }
197:                }
198:            } else {
199:                if (price < 0.94201e6) {
200:                    if (price < 0.916852e6) {
201:                        if (price < 0.898101e6) {
202:                            if (price < 0.891863e6) {
203:                                if (price < 0.89081e6) {
204:                                    return PriceData(
205:                                        0.89081e6,
206:                                        0,
207:                                        1.404927999999999955e18,
208:                                        0.885627e6,
209:                                        0,
210:                                        1.428246247576273165e18,
211:                                        1e18
212:                                    );
213:                                } else {
214:                                    return PriceData(
215:                                        0.891863e6,
216:                                        0,
217:                                        1.400241419192424397e18,
218:                                        0.89081e6,
219:                                        0,
220:                                        1.404927999999999955e18,
221:                                        1e18
222:                                    );
223:                                }
224:                            } else {
225:                                return PriceData(
226:                                    0.898101e6, 0, 1.372785705090612263e18, 0.891863e6, 0, 1.400241419192424397e18, 1e18
227:                                );
228:                            }
229:                        } else {
230:                            if (price < 0.910594e6) {
231:                                if (price < 0.904344e6) {
232:                                    return PriceData(
233:                                        0.904344e6,
234:                                        0,
235:                                        1.345868338324129665e18,
236:                                        0.898101e6,
237:                                        0,
238:                                        1.372785705090612263e18,
239:                                        1e18
240:                                    );
241:                                } else {
242:                                    return PriceData(
243:                                        0.910594e6,
244:                                        0,
245:                                        1.319478763062872151e18,
246:                                        0.904344e6,
247:                                        0,
248:                                        1.345868338324129665e18,
249:                                        1e18
250:                                    );
251:                                }
252:                            } else {
253:                                return PriceData(
254:                                    0.916852e6, 0, 1.293606630453796313e18, 0.910594e6, 0, 1.319478763062872151e18, 1e18
255:                                );
256:                            }
257:                        }
258:                    } else {
259:                        if (price < 0.929402e6) {
260:                            if (price < 0.9266e6) {
261:                                if (price < 0.92312e6) {
262:                                    return PriceData(
263:                                        0.92312e6,
264:                                        0,
265:                                        1.268241794562545266e18,
266:                                        0.916852e6,
267:                                        0,
268:                                        1.293606630453796313e18,
269:                                        1e18
270:                                    );
271:                                } else {
272:                                    return PriceData(
273:                                        0.9266e6,
274:                                        0,
275:                                        1.254399999999999959e18,
276:                                        0.92312e6,
277:                                        0,
278:                                        1.268241794562545266e18,
279:                                        1e18
280:                                    );
281:                                }
282:                            } else {
283:                                return PriceData(
284:                                    0.929402e6, 0, 1.243374308394652239e18, 0.9266e6, 0, 1.254399999999999959e18, 1e18
285:                                );
286:                            }
287:                        } else {
288:                            if (price < 0.935697e6) {
289:                                return PriceData(
290:                                    0.935697e6, 0, 1.218994419994757328e18, 0.929402e6, 0, 1.243374308394652239e18, 1e18
291:                                );
292:                            } else {
293:                                return PriceData(
294:                                    0.94201e6, 0, 1.195092568622310836e18, 0.935697e6, 0, 1.218994419994757328e18, 1e18
295:                                );
296:                            }
297:                        }
298:                    }
299:                } else {
300:                    if (price < 0.96748e6) {
301:                        if (price < 0.961075e6) {
302:                            if (price < 0.954697e6) {
303:                                if (price < 0.948343e6) {
304:                                    return PriceData(
305:                                        0.948343e6,
306:                                        0,
307:                                        1.171659381002265521e18,
308:                                        0.94201e6,
309:                                        0,
310:                                        1.195092568622310836e18,
311:                                        1e18
312:                                    );
313:                                } else {
314:                                    return PriceData(
315:                                        0.954697e6,
316:                                        0,
317:                                        1.14868566764928004e18,
318:                                        0.948343e6,
319:                                        0,
320:                                        1.171659381002265521e18,
321:                                        1e18
322:                                    );
323:                                }
324:                            } else {
325:                                return PriceData(
326:                                    0.961075e6, 0, 1.12616241926400007e18, 0.954697e6, 0, 1.14868566764928004e18, 1e18
327:                                );
328:                            }
329:                        } else {
330:                            if (price < 0.962847e6) {
331:                                return PriceData(
332:                                    0.962847e6, 0, 1.120000000000000107e18, 0.961075e6, 0, 1.12616241926400007e18, 1e18
333:                                );
334:                            } else {
335:                                return PriceData(
336:                                    0.96748e6, 0, 1.104080803200000016e18, 0.962847e6, 0, 1.120000000000000107e18, 1e18
337:                                );
338:                            }
339:                        }
340:                    } else {
341:                        if (price < 0.986882e6) {
342:                            if (price < 0.98038e6) {
343:                                if (price < 0.973914e6) {
344:                                    return PriceData(
345:                                        0.973914e6,
346:                                        0,
347:                                        1.082432159999999977e18,
348:                                        0.96748e6,
349:                                        0,
350:                                        1.104080803200000016e18,
351:                                        1e18
352:                                    );
353:                                } else {
354:                                    return PriceData(
355:                                        0.98038e6,
356:                                        0,
357:                                        1.061208000000000151e18,
358:                                        0.973914e6,
359:                                        0,
360:                                        1.082432159999999977e18,
361:                                        1e18
362:                                    );
363:                                }
364:                            } else {
365:                                return PriceData(
366:                                    0.986882e6, 0, 1.040399999999999991e18, 0.98038e6, 0, 1.061208000000000151e18, 1e18
367:                                );
368:                            }
369:                        } else {
370:                            if (price < 0.993421e6) {
371:                                return PriceData(
372:                                    0.993421e6, 0, 1.020000000000000018e18, 0.986882e6, 0, 1.040399999999999991e18, 1e18
373:                                );
374:                            } else {
375:                                return PriceData(
376:                                    1.006758e6, 0, 0.980000000000000093e18, 0.993421e6, 0, 1.020000000000000018e18, 1e18
377:                                );
378:                            }
379:                        }
380:                    }
381:                }
382:            }
383:        } else {
384:            if (price < 1.140253e6) {
385:                if (price < 1.077582e6) {
386:                    if (price < 1.04366e6) {
387:                        if (price < 1.027335e6) {
388:                            if (price < 1.020422e6) {
389:                                if (price < 1.013564e6) {
390:                                    return PriceData(
391:                                        1.013564e6,
392:                                        0,
393:                                        0.960400000000000031e18,
394:                                        1.006758e6,
395:                                        0,
396:                                        0.980000000000000093e18,
397:                                        1e18
398:                                    );
399:                                } else {
400:                                    return PriceData(
401:                                        1.020422e6,
402:                                        0,
403:                                        0.941192000000000029e18,
404:                                        1.013564e6,
405:                                        0,
406:                                        0.960400000000000031e18,
407:                                        1e18
408:                                    );
409:                                }
410:                            } else {
411:                                return PriceData(
412:                                    1.027335e6, 0, 0.922368159999999992e18, 1.020422e6, 0, 0.941192000000000029e18, 1e18
413:                                );
414:                            }
415:                        } else {
416:                            if (price < 1.041342e6) {
417:                                if (price < 1.034307e6) {
418:                                    return PriceData(
419:                                        1.034307e6,
420:                                        0,
421:                                        0.903920796799999926e18,
422:                                        1.027335e6,
423:                                        0,
424:                                        0.922368159999999992e18,
425:                                        1e18
426:                                    );
427:                                } else {
428:                                    return PriceData(
429:                                        1.041342e6,
430:                                        0,
431:                                        0.885842380864000023e18,
432:                                        1.034307e6,
433:                                        0,
434:                                        0.903920796799999926e18,
435:                                        1e18
436:                                    );
437:                                }
438:                            } else {
439:                                return PriceData(
440:                                    1.04366e6, 0, 0.880000000000000004e18, 1.041342e6, 0, 0.885842380864000023e18, 1e18
441:                                );
442:                            }
443:                        }
444:                    } else {
445:                        if (price < 1.062857e6) {
446:                            if (price < 1.055613e6) {
447:                                if (price < 1.048443e6) {
448:                                    return PriceData(
449:                                        1.048443e6,
450:                                        0,
451:                                        0.868125533246720038e18,
452:                                        1.04366e6,
453:                                        0,
454:                                        0.880000000000000004e18,
455:                                        1e18
456:                                    );
457:                                } else {
458:                                    return PriceData(
459:                                        1.055613e6,
460:                                        0,
461:                                        0.8507630225817856e18,
462:                                        1.048443e6,
463:                                        0,
464:                                        0.868125533246720038e18,
465:                                        1e18
466:                                    );
467:                                }
468:                            } else {
469:                                return PriceData(
470:                                    1.062857e6, 0, 0.833747762130149894e18, 1.055613e6, 0, 0.8507630225817856e18, 1e18
471:                                );
472:                            }
473:                        } else {
474:                            if (price < 1.070179e6) {
475:                                return PriceData(
476:                                    1.070179e6, 0, 0.81707280688754691e18, 1.062857e6, 0, 0.833747762130149894e18, 1e18
477:                                );
478:                            } else {
479:                                return PriceData(
480:                                    1.077582e6, 0, 0.800731350749795956e18, 1.070179e6, 0, 0.81707280688754691e18, 1e18
481:                                );
482:                            }
483:                        }
484:                    }
485:                } else {
486:                    if (price < 1.108094e6) {
487:                        if (price < 1.09265e6) {
488:                            if (price < 1.090025e6) {
489:                                if (price < 1.085071e6) {
490:                                    return PriceData(
491:                                        1.085071e6,
492:                                        0,
493:                                        0.784716723734800059e18,
494:                                        1.077582e6,
495:                                        0,
496:                                        0.800731350749795956e18,
497:                                        1e18
498:                                    );
499:                                } else {
500:                                    return PriceData(
501:                                        1.090025e6,
502:                                        0,
503:                                        0.774399999999999977e18,
504:                                        1.085071e6,
505:                                        0,
506:                                        0.784716723734800059e18,
507:                                        1e18
508:                                    );
509:                                }
510:                            } else {
511:                                return PriceData(
512:                                    1.09265e6, 0, 0.769022389260104022e18, 1.090025e6, 0, 0.774399999999999977e18, 1e18
513:                                );
514:                            }
515:                        } else {
516:                            if (price < 1.100323e6) {
517:                                return PriceData(
518:                                    1.100323e6, 0, 0.753641941474902044e18, 1.09265e6, 0, 0.769022389260104022e18, 1e18
519:                                );
520:                            } else {
521:                                return PriceData(
522:                                    1.108094e6, 0, 0.738569102645403985e18, 1.100323e6, 0, 0.753641941474902044e18, 1e18
523:                                );
524:                            }
525:                        }
526:                    } else {
527:                        if (price < 1.132044e6) {
528:                            if (price < 1.123949e6) {
529:                                if (price < 1.115967e6) {
530:                                    return PriceData(
531:                                        1.115967e6,
532:                                        0,
533:                                        0.723797720592495919e18,
534:                                        1.108094e6,
535:                                        0,
536:                                        0.738569102645403985e18,
537:                                        1e18
538:                                    );
539:                                } else {
540:                                    return PriceData(
541:                                        1.123949e6,
542:                                        0,
543:                                        0.709321766180645907e18,
544:                                        1.115967e6,
545:                                        0,
546:                                        0.723797720592495919e18,
547:                                        1e18
548:                                    );
549:                                }
550:                            } else {
551:                                return PriceData(
552:                                    1.132044e6, 0, 0.695135330857033051e18, 1.123949e6, 0, 0.709321766180645907e18, 1e18
553:                                );
554:                            }
555:                        } else {
556:                            if (price < 1.14011e6) {
557:                                return PriceData(
558:                                    1.14011e6, 0, 0.681471999999999967e18, 1.132044e6, 0, 0.695135330857033051e18, 1e18
559:                                );
560:                            } else {
561:                                return PriceData(
562:                                    1.140253e6, 0, 0.681232624239892393e18, 1.14011e6, 0, 0.681471999999999967e18, 1e18
563:                                );
564:                            }
565:                        }
566:                    }
567:                }
568:            } else {
569:                if (price < 2.01775e6) {
570:                    if (price < 1.403579e6) {
571:                        if (price < 1.256266e6) {
572:                            if (price < 1.195079e6) {
573:                                if (price < 1.148586e6) {
574:                                    return PriceData(
575:                                        1.148586e6,
576:                                        0,
577:                                        0.667607971755094454e18,
578:                                        1.140253e6,
579:                                        0,
580:                                        0.681232624239892393e18,
581:                                        1e18
582:                                    );
583:                                } else {
584:                                    return PriceData(
585:                                        1.195079e6,
586:                                        0,
587:                                        0.599695360000000011e18,
588:                                        1.148586e6,
589:                                        0,
590:                                        0.667607971755094454e18,
591:                                        1e18
592:                                    );
593:                                }
594:                            } else {
595:                                return PriceData(
596:                                    1.256266e6, 0, 0.527731916799999978e18, 1.195079e6, 0, 0.599695360000000011e18, 1e18
597:                                );
598:                            }
599:                        } else {
600:                            if (price < 1.325188e6) {
601:                                return PriceData(
602:                                    1.325188e6, 0, 0.464404086784000025e18, 1.256266e6, 0, 0.527731916799999978e18, 1e18
603:                                );
604:                            } else {
605:                                return PriceData(
606:                                    1.403579e6, 0, 0.408675596369920013e18, 1.325188e6, 0, 0.464404086784000025e18, 1e18
607:                                );
608:                            }
609:                        }
610:                    } else {
611:                        if (price < 1.716848e6) {
612:                            if (price < 1.596984e6) {
613:                                if (price < 1.493424e6) {
614:                                    return PriceData(
615:                                        1.493424e6,
616:                                        0,
617:                                        0.359634524805529598e18,
618:                                        1.403579e6,
619:                                        0,
620:                                        0.408675596369920013e18,
621:                                        1e18
622:                                    );
623:                                } else {
624:                                    return PriceData(
625:                                        1.596984e6,
626:                                        0,
627:                                        0.316478381828866062e18,
628:                                        1.493424e6,
629:                                        0,
630:                                        0.359634524805529598e18,
631:                                        1e18
632:                                    );
633:                                }
634:                            } else {
635:                                return PriceData(
636:                                    1.716848e6, 0, 0.278500976009402101e18, 1.596984e6, 0, 0.316478381828866062e18, 1e18
637:                                );
638:                            }
639:                        } else {
640:                            if (price < 1.855977e6) {
641:                                return PriceData(
642:                                    1.855977e6, 0, 0.245080858888273884e18, 1.716848e6, 0, 0.278500976009402101e18, 1e18
643:                                );
644:                            } else {
645:                                return PriceData(
646:                                    2.01775e6, 0, 0.215671155821681004e18, 1.855977e6, 0, 0.245080858888273884e18, 1e18
647:                                );
648:                            }
649:                        }
650:                    }
651:                } else {
652:                    if (price < 3.322705e6) {
653:                        if (price < 2.680458e6) {
654:                            if (price < 2.425256e6) {
655:                                if (price < 2.206036e6) {
656:                                    return PriceData(
657:                                        2.206036e6,
658:                                        0,
659:                                        0.189790617123079292e18,
660:                                        2.01775e6,
661:                                        0,
662:                                        0.215671155821681004e18,
663:                                        1e18
664:                                    );
665:                                } else {
666:                                    return PriceData(
667:                                        2.425256e6,
668:                                        0,
669:                                        0.167015743068309769e18,
670:                                        2.206036e6,
671:                                        0,
672:                                        0.189790617123079292e18,
673:                                        1e18
674:                                    );
675:                                }
676:                            } else {
677:                                return PriceData(
678:                                    2.680458e6, 0, 0.146973853900112583e18, 2.425256e6, 0, 0.167015743068309769e18, 1e18
679:                                );
680:                            }
681:                        } else {
682:                            if (price < 2.977411e6) {
683:                                return PriceData(
684:                                    2.977411e6, 0, 0.129336991432099091e18, 2.680458e6, 0, 0.146973853900112583e18, 1e18
685:                                );
686:                            } else {
687:                                return PriceData(
688:                                    3.322705e6, 0, 0.113816552460247203e18, 2.977411e6, 0, 0.129336991432099091e18, 1e18
689:                                );
690:                            }
691:                        }
692:                    } else {
693:                        if (price < 4.729321e6) {
694:                            if (price < 4.189464e6) {
695:                                if (price < 3.723858e6) {
696:                                    return PriceData(
697:                                        3.723858e6,
698:                                        0,
699:                                        0.100158566165017532e18,
700:                                        3.322705e6,
701:                                        0,
702:                                        0.113816552460247203e18,
703:                                        1e18
704:                                    );
705:                                } else {
706:                                    return PriceData(
707:                                        4.189464e6,
708:                                        0,
709:                                        0.088139538225215433e18,
710:                                        3.723858e6,
711:                                        0,
712:                                        0.100158566165017532e18,
713:                                        1e18
714:                                    );
715:                                }
716:                            } else {
717:                                return PriceData(
718:                                    4.729321e6, 0, 0.077562793638189589e18, 4.189464e6, 0, 0.088139538225215433e18, 1e18
719:                                );
720:                            }
721:                        } else {
722:                            if (price < 10.37089e6) {
723:                                return PriceData(
724:                                    10.37089e6, 0, 0.035714285714285712e18, 4.729321e6, 0, 0.077562793638189589e18, 1e18
725:                                );
726:                            } else {
727:                                revert("LUT: Invalid price");
728:                            }
729:                        }
730:                    }
731:                }
732:            }
733:        }
734:    }

740:    function getRatiosFromPriceSwap(uint256 price) external pure returns (PriceData memory) {	// @audit-issue
741:        if (price < 0.993344e6) {
742:            if (price < 0.834426e6) {
743:                if (price < 0.718073e6) {
744:                    if (price < 0.391201e6) {
745:                        if (price < 0.264147e6) {
746:                            if (price < 0.213318e6) {
747:                                if (price < 0.001083e6) {
748:                                    revert("LUT: Invalid price");
749:                                } else {
750:                                    return PriceData(
751:                                        0.213318e6,
752:                                        0.188693329162796575e18,
753:                                        2.410556040105746423e18,
754:                                        0.001083e6,
755:                                        0.005263157894736842e18,
756:                                        10.522774272309483479e18,
757:                                        1e18
758:                                    );
759:                                }
760:                            } else {
761:                                if (price < 0.237671e6) {
762:                                    return PriceData(
763:                                        0.237671e6,
764:                                        0.20510144474217018e18,
765:                                        2.337718072004858261e18,
766:                                        0.213318e6,
767:                                        0.188693329162796575e18,
768:                                        2.410556040105746423e18,
769:                                        1e18
770:                                    );
771:                                } else {
772:                                    return PriceData(
773:                                        0.264147e6,
774:                                        0.222936352980619729e18,
775:                                        2.26657220303422724e18,
776:                                        0.237671e6,
777:                                        0.20510144474217018e18,
778:                                        2.337718072004858261e18,
779:                                        1e18
780:                                    );
781:                                }
782:                            }
783:                        } else {
784:                            if (price < 0.323531e6) {
785:                                if (price < 0.292771e6) {
786:                                    return PriceData(
787:                                        0.292771e6,
788:                                        0.242322122805021467e18,
789:                                        2.196897480682568293e18,
790:                                        0.264147e6,
791:                                        0.222936352980619729e18,
792:                                        2.26657220303422724e18,
793:                                        1e18
794:                                    );
795:                                } else {
796:                                    return PriceData(
797:                                        0.323531e6,
798:                                        0.263393611744588529e18,
799:                                        2.128468246736633152e18,
800:                                        0.292771e6,
801:                                        0.242322122805021467e18,
802:                                        2.196897480682568293e18,
803:                                        1e18
804:                                    );
805:                                }
806:                            } else {
807:                                if (price < 0.356373e6) {
808:                                    return PriceData(
809:                                        0.356373e6,
810:                                        0.286297404070204931e18,
811:                                        2.061053544007124483e18,
812:                                        0.323531e6,
813:                                        0.263393611744588529e18,
814:                                        2.128468246736633152e18,
815:                                        1e18
816:                                    );
817:                                } else {
818:                                    return PriceData(
819:                                        0.391201e6,
820:                                        0.311192830511092366e18,
821:                                        1.994416599735895801e18,
822:                                        0.356373e6,
823:                                        0.286297404070204931e18,
824:                                        2.061053544007124483e18,
825:                                        1e18
826:                                    );
827:                                }
828:                            }
829:                        }
830:                    } else {
831:                        if (price < 0.546918e6) {
832:                            if (price < 0.466197e6) {
833:                                if (price < 0.427871e6) {
834:                                    return PriceData(
835:                                        0.427871e6,
836:                                        0.338253076642491657e18,
837:                                        1.92831441898410505e18,
838:                                        0.391201e6,
839:                                        0.311192830511092366e18,
840:                                        1.994416599735895801e18,
841:                                        1e18
842:                                    );
843:                                } else {
844:                                    return PriceData(
845:                                        0.466197e6,
846:                                        0.367666387654882243e18,
847:                                        1.86249753363281334e18,
848:                                        0.427871e6,
849:                                        0.338253076642491657e18,
850:                                        1.92831441898410505e18,
851:                                        1e18
852:                                    );
853:                                }
854:                            } else {
855:                                if (price < 0.50596e6) {
856:                                    return PriceData(
857:                                        0.50596e6,
858:                                        0.399637377885741607e18,
859:                                        1.796709969924970451e18,
860:                                        0.466197e6,
861:                                        0.367666387654882243e18,
862:                                        1.86249753363281334e18,
863:                                        1e18
864:                                    );
865:                                } else {
866:                                    return PriceData(
867:                                        0.546918e6,
868:                                        0.434388454223632148e18,
869:                                        1.73068952191306602e18,
870:                                        0.50596e6,
871:                                        0.399637377885741607e18,
872:                                        1.796709969924970451e18,
873:                                        1e18
874:                                    );
875:                                }
876:                            }
877:                        } else {
878:                            if (price < 0.631434e6) {
879:                                if (price < 0.588821e6) {
880:                                    return PriceData(
881:                                        0.588821e6,
882:                                        0.472161363286556723e18,
883:                                        1.664168452923131536e18,
884:                                        0.546918e6,
885:                                        0.434388454223632148e18,
886:                                        1.73068952191306602e18,
887:                                        1e18
888:                                    );
889:                                } else {
890:                                    return PriceData(
891:                                        0.631434e6,
892:                                        0.513218873137561538e18,
893:                                        1.596874796852916001e18,
894:                                        0.588821e6,
895:                                        0.472161363286556723e18,
896:                                        1.664168452923131536e18,
897:                                        1e18
898:                                    );
899:                                }
900:                            } else {
901:                                if (price < 0.67456e6) {
902:                                    return PriceData(
903:                                        0.67456e6,
904:                                        0.55784660123648e18,
905:                                        1.52853450260679824e18,
906:                                        0.631434e6,
907:                                        0.513218873137561538e18,
908:                                        1.596874796852916001e18,
909:                                        1e18
910:                                    );
911:                                } else {
912:                                    return PriceData(
913:                                        0.718073e6,
914:                                        0.606355001344e18,
915:                                        1.458874768183093584e18,
916:                                        0.67456e6,
917:                                        0.55784660123648e18,
918:                                        1.52853450260679824e18,
919:                                        1e18
920:                                    );
921:                                }
922:                            }
923:                        }
924:                    }
925:                } else {
926:                    if (price < 0.801931e6) {
927:                        if (price < 0.780497e6) {
928:                            if (price < 0.769833e6) {
929:                                if (price < 0.76195e6) {
930:                                    return PriceData(
931:                                        0.76195e6,
932:                                        0.659081523200000019e18,
933:                                        1.387629060213009469e18,
934:                                        0.718073e6,
935:                                        0.606355001344e18,
936:                                        1.458874768183093584e18,
937:                                        1e18
938:                                    );
939:                                } else {
940:                                    return PriceData(
941:                                        0.769833e6,
942:                                        0.668971758569680497e18,
943:                                        1.37471571145172633e18,
944:                                        0.76195e6,
945:                                        0.659081523200000019e18,
946:                                        1.387629060213009469e18,
947:                                        1e18
948:                                    );
949:                                }
950:                            } else {
951:                                if (price < 0.775161e6) {
952:                                    return PriceData(
953:                                        0.775161e6,
954:                                        0.675729049060283415e18,
955:                                        1.365968375000512491e18,
956:                                        0.769833e6,
957:                                        0.668971758569680497e18,
958:                                        1.37471571145172633e18,
959:                                        1e18
960:                                    );
961:                                } else {
962:                                    return PriceData(
963:                                        0.780497e6,
964:                                        0.682554595010387288e18,
965:                                        1.357193251389227306e18,
966:                                        0.775161e6,
967:                                        0.675729049060283415e18,
968:                                        1.365968375000512491e18,
969:                                        1e18
970:                                    );
971:                                }
972:                            }
973:                        } else {
974:                            if (price < 0.791195e6) {
975:                                if (price < 0.785842e6) {
976:                                    return PriceData(
977:                                        0.785842e6,
978:                                        0.689449085869078049e18,
979:                                        1.34838993014876074e18,
980:                                        0.780497e6,
981:                                        0.682554595010387288e18,
982:                                        1.357193251389227306e18,
983:                                        1e18
984:                                    );
985:                                } else {
986:                                    return PriceData(
987:                                        0.791195e6,
988:                                        0.696413218049573679e18,
989:                                        1.339558007037547016e18,
990:                                        0.785842e6,
991:                                        0.689449085869078049e18,
992:                                        1.34838993014876074e18,
993:                                        1e18
994:                                    );
995:                                }
996:                            } else {
997:                                if (price < 0.796558e6) {
998:                                    return PriceData(
999:                                        0.796558e6,
1000:                                        0.703447694999569495e18,
1001:                                        1.330697084427678423e18,
1002:                                        0.791195e6,
1003:                                        0.696413218049573679e18,
1004:                                        1.339558007037547016e18,
1005:                                        1e18
1006:                                    );
1007:                                } else {
1008:                                    return PriceData(
1009:                                        0.801931e6,
1010:                                        0.710553227272292309e18,
1011:                                        1.321806771708554873e18,
1012:                                        0.796558e6,
1013:                                        0.703447694999569495e18,
1014:                                        1.330697084427678423e18,
1015:                                        1e18
1016:                                    );
1017:                                }
1018:                            }
1019:                        }
1020:                    } else {
1021:                        if (price < 0.818119e6) {
1022:                            if (price < 0.807315e6) {
1023:                                if (price < 0.806314e6) {
1024:                                    return PriceData(
1025:                                        0.806314e6,
1026:                                        0.716392959999999968e18,
1027:                                        1.314544530202049311e18,
1028:                                        0.801931e6,
1029:                                        0.710553227272292309e18,
1030:                                        1.321806771708554873e18,
1031:                                        1e18
1032:                                    );
1033:                                } else {
1034:                                    return PriceData(
1035:                                        0.807315e6,
1036:                                        0.717730532598275128e18,
1037:                                        1.312886685708826162e18,
1038:                                        0.806314e6,
1039:                                        0.716392959999999968e18,
1040:                                        1.314544530202049311e18,
1041:                                        1e18
1042:                                    );
1043:                                }
1044:                            } else {
1045:                                if (price < 0.812711e6) {
1046:                                    return PriceData(
1047:                                        0.812711e6,
1048:                                        0.724980335957853717e18,
1049:                                        1.303936451137418295e18,
1050:                                        0.807315e6,
1051:                                        0.717730532598275128e18,
1052:                                        1.312886685708826162e18,
1053:                                        1e18
1054:                                    );
1055:                                } else {
1056:                                    return PriceData(
1057:                                        0.818119e6,
1058:                                        0.732303369654397684e18,
1059:                                        1.294955701044462559e18,
1060:                                        0.812711e6,
1061:                                        0.724980335957853717e18,
1062:                                        1.303936451137418295e18,
1063:                                        1e18
1064:                                    );
1065:                                }
1066:                            }
1067:                        } else {
1068:                            if (price < 0.828976e6) {
1069:                                if (price < 0.82354e6) {
1070:                                    return PriceData(
1071:                                        0.82354e6,
1072:                                        0.73970037338828043e18,
1073:                                        1.285944077302980215e18,
1074:                                        0.818119e6,
1075:                                        0.732303369654397684e18,
1076:                                        1.294955701044462559e18,
1077:                                        1e18
1078:                                    );
1079:                                } else {
1080:                                    return PriceData(
1081:                                        0.828976e6,
1082:                                        0.74717209433159637e18,
1083:                                        1.276901231112211654e18,
1084:                                        0.82354e6,
1085:                                        0.73970037338828043e18,
1086:                                        1.285944077302980215e18,
1087:                                        1e18
1088:                                    );
1089:                                }
1090:                            } else {
1091:                                return PriceData(
1092:                                    0.834426e6,
1093:                                    0.754719287203632794e18,
1094:                                    1.267826823523503732e18,
1095:                                    0.828976e6,
1096:                                    0.74717209433159637e18,
1097:                                    1.276901231112211654e18,
1098:                                    1e18
1099:                                );
1100:                            }
1101:                        }
1102:                    }
1103:                }
1104:            } else {
1105:                if (price < 0.907266e6) {
1106:                    if (price < 0.873109e6) {
1107:                        if (price < 0.851493e6) {
1108:                            if (price < 0.845379e6) {
1109:                                if (price < 0.839894e6) {
1110:                                    return PriceData(
1111:                                        0.839894e6,
1112:                                        0.762342714347103767e18,
1113:                                        1.258720525989716954e18,
1114:                                        0.834426e6,
1115:                                        0.754719287203632794e18,
1116:                                        1.267826823523503732e18,
1117:                                        1e18
1118:                                    );
1119:                                } else {
1120:                                    return PriceData(
1121:                                        0.845379e6,
1122:                                        0.770043145805155316e18,
1123:                                        1.249582020939133509e18,
1124:                                        0.839894e6,
1125:                                        0.762342714347103767e18,
1126:                                        1.258720525989716954e18,
1127:                                        1e18
1128:                                    );
1129:                                }
1130:                            } else {
1131:                                if (price < 0.850882e6) {
1132:                                    return PriceData(
1133:                                        0.850882e6,
1134:                                        0.777821359399146761e18,
1135:                                        1.240411002374896432e18,
1136:                                        0.845379e6,
1137:                                        0.770043145805155316e18,
1138:                                        1.249582020939133509e18,
1139:                                        1e18
1140:                                    );
1141:                                } else {
1142:                                    return PriceData(
1143:                                        0.851493e6,
1144:                                        0.778688000000000047e18,
1145:                                        1.239392846883276889e18,
1146:                                        0.850882e6,
1147:                                        0.777821359399146761e18,
1148:                                        1.240411002374896432e18,
1149:                                        1e18
1150:                                    );
1151:                                }
1152:                            }
1153:                        } else {
1154:                            if (price < 0.86195e6) {
1155:                                if (price < 0.856405e6) {
1156:                                    return PriceData(
1157:                                        0.856405e6,
1158:                                        0.785678140807218983e18,
1159:                                        1.231207176501035727e18,
1160:                                        0.851493e6,
1161:                                        0.778688000000000047e18,
1162:                                        1.239392846883276889e18,
1163:                                        1e18
1164:                                    );
1165:                                } else {
1166:                                    return PriceData(
1167:                                        0.86195e6,
1168:                                        0.793614283643655494e18,
1169:                                        1.221970262376178118e18,
1170:                                        0.856405e6,
1171:                                        0.785678140807218983e18,
1172:                                        1.231207176501035727e18,
1173:                                        1e18
1174:                                    );
1175:                                }
1176:                            } else {
1177:                                if (price < 0.867517e6) {
1178:                                    return PriceData(
1179:                                        0.867517e6,
1180:                                        0.801630589539045979e18,
1181:                                        1.212699992596070864e18,
1182:                                        0.86195e6,
1183:                                        0.793614283643655494e18,
1184:                                        1.221970262376178118e18,
1185:                                        1e18
1186:                                    );
1187:                                } else {
1188:                                    return PriceData(
1189:                                        0.873109e6,
1190:                                        0.809727868221258529e18,
1191:                                        1.203396114006087814e18,
1192:                                        0.867517e6,
1193:                                        0.801630589539045979e18,
1194:                                        1.212699992596070864e18,
1195:                                        1e18
1196:                                    );
1197:                                }
1198:                            }
1199:                        }
1200:                    } else {
1201:                        if (price < 0.895753e6) {
1202:                            if (price < 0.884372e6) {
1203:                                if (price < 0.878727e6) {
1204:                                    return PriceData(
1205:                                        0.878727e6,
1206:                                        0.817906937597230987e18,
1207:                                        1.194058388444914964e18,
1208:                                        0.873109e6,
1209:                                        0.809727868221258529e18,
1210:                                        1.203396114006087814e18,
1211:                                        1e18
1212:                                    );
1213:                                } else {
1214:                                    return PriceData(
1215:                                        0.884372e6,
1216:                                        0.826168623835586646e18,
1217:                                        1.18468659352065786e18,
1218:                                        0.878727e6,
1219:                                        0.817906937597230987e18,
1220:                                        1.194058388444914964e18,
1221:                                        1e18
1222:                                    );
1223:                                }
1224:                            } else {
1225:                                if (price < 0.890047e6) {
1226:                                    return PriceData(
1227:                                        0.890047e6,
1228:                                        0.834513761450087599e18,
1229:                                        1.17528052342063094e18,
1230:                                        0.884372e6,
1231:                                        0.826168623835586646e18,
1232:                                        1.18468659352065786e18,
1233:                                        1e18
1234:                                    );
1235:                                } else {
1236:                                    return PriceData(
1237:                                        0.895753e6,
1238:                                        0.84294319338392687e18,
1239:                                        1.16583998975613734e18,
1240:                                        0.890047e6,
1241:                                        0.834513761450087599e18,
1242:                                        1.17528052342063094e18,
1243:                                        1e18
1244:                                    );
1245:                                }
1246:                            }
1247:                        } else {
1248:                            if (price < 0.901491e6) {
1249:                                if (price < 0.898085e6) {
1250:                                    return PriceData(
1251:                                        0.898085e6,
1252:                                        0.846400000000000041e18,
1253:                                        1.161985895520041945e18,
1254:                                        0.895753e6,
1255:                                        0.84294319338392687e18,
1256:                                        1.16583998975613734e18,
1257:                                        1e18
1258:                                    );
1259:                                } else {
1260:                                    return PriceData(
1261:                                        0.901491e6,
1262:                                        0.851457771094875637e18,
1263:                                        1.156364822443562979e18,
1264:                                        0.898085e6,
1265:                                        0.846400000000000041e18,
1266:                                        1.161985895520041945e18,
1267:                                        1e18
1268:                                    );
1269:                                }
1270:                            } else {
1271:                                return PriceData(
1272:                                    0.907266e6,
1273:                                    0.860058354641288547e18,
1274:                                    1.146854870623147615e18,
1275:                                    0.901491e6,
1276:                                    0.851457771094875637e18,
1277:                                    1.156364822443562979e18,
1278:                                    1e18
1279:                                );
1280:                            }
1281:                        }
1282:                    }
1283:                } else {
1284:                    if (price < 0.948888e6) {
1285:                        if (price < 0.930767e6) {
1286:                            if (price < 0.918932e6) {
1287:                                if (price < 0.913079e6) {
1288:                                    return PriceData(
1289:                                        0.913079e6,
1290:                                        0.868745812768978332e18,
1291:                                        1.137310003616810228e18,
1292:                                        0.907266e6,
1293:                                        0.860058354641288547e18,
1294:                                        1.146854870623147615e18,
1295:                                        1e18
1296:                                    );
1297:                                } else {
1298:                                    return PriceData(
1299:                                        0.918932e6,
1300:                                        0.877521022998967948e18,
1301:                                        1.127730111926438461e18,
1302:                                        0.913079e6,
1303:                                        0.868745812768978332e18,
1304:                                        1.137310003616810228e18,
1305:                                        1e18
1306:                                    );
1307:                                }
1308:                            } else {
1309:                                if (price < 0.924827e6) {
1310:                                    return PriceData(
1311:                                        0.924827e6,
1312:                                        0.88638487171612923e18,
1313:                                        1.118115108274055913e18,
1314:                                        0.918932e6,
1315:                                        0.877521022998967948e18,
1316:                                        1.127730111926438461e18,
1317:                                        1e18
1318:                                    );
1319:                                } else {
1320:                                    return PriceData(
1321:                                        0.930767e6,
1322:                                        0.895338254258716493e18,
1323:                                        1.10846492868530544e18,
1324:                                        0.924827e6,
1325:                                        0.88638487171612923e18,
1326:                                        1.118115108274055913e18,
1327:                                        1e18
1328:                                    );
1329:                                }
1330:                            }
1331:                        } else {
1332:                            if (price < 0.942795e6) {
1333:                                if (price < 0.936756e6) {
1334:                                    return PriceData(
1335:                                        0.936756e6,
1336:                                        0.90438207500880452e18,
1337:                                        1.09877953361768621e18,
1338:                                        0.930767e6,
1339:                                        0.895338254258716493e18,
1340:                                        1.10846492868530544e18,
1341:                                        1e18
1342:                                    );
1343:                                } else {
1344:                                    return PriceData(
1345:                                        0.942795e6,
1346:                                        0.913517247483640937e18,
1347:                                        1.089058909134983155e18,
1348:                                        0.936756e6,
1349:                                        0.90438207500880452e18,
1350:                                        1.09877953361768621e18,
1351:                                        1e18
1352:                                    );
1353:                                }
1354:                            } else {
1355:                                if (price < 0.947076e6) {
1356:                                    return PriceData(
1357:                                        0.947076e6,
1358:                                        0.92000000000000004e18,
1359:                                        1.082198372170484424e18,
1360:                                        0.942795e6,
1361:                                        0.913517247483640937e18,
1362:                                        1.089058909134983155e18,
1363:                                        1e18
1364:                                    );
1365:                                } else {
1366:                                    return PriceData(
1367:                                        0.948888e6,
1368:                                        0.922744694427920065e18,
1369:                                        1.079303068129318754e18,
1370:                                        0.947076e6,
1371:                                        0.92000000000000004e18,
1372:                                        1.082198372170484424e18,
1373:                                        1e18
1374:                                    );
1375:                                }
1376:                            }
1377:                        }
1378:                    } else {
1379:                        if (price < 0.973868e6) {
1380:                            if (price < 0.961249e6) {
1381:                                if (price < 0.955039e6) {
1382:                                    return PriceData(
1383:                                        0.955039e6,
1384:                                        0.932065347906990027e18,
1385:                                        1.069512051592246715e18,
1386:                                        0.948888e6,
1387:                                        0.922744694427920065e18,
1388:                                        1.079303068129318754e18,
1389:                                        1e18
1390:                                    );
1391:                                } else {
1392:                                    return PriceData(
1393:                                        0.961249e6,
1394:                                        0.941480149400999999e18,
1395:                                        1.059685929936267312e18,
1396:                                        0.955039e6,
1397:                                        0.932065347906990027e18,
1398:                                        1.069512051592246715e18,
1399:                                        1e18
1400:                                    );
1401:                                }
1402:                            } else {
1403:                                if (price < 0.967525e6) {
1404:                                    return PriceData(
1405:                                        0.967525e6,
1406:                                        0.950990049900000023e18,
1407:                                        1.049824804368118425e18,
1408:                                        0.961249e6,
1409:                                        0.941480149400999999e18,
1410:                                        1.059685929936267312e18,
1411:                                        1e18
1412:                                    );
1413:                                } else {
1414:                                    return PriceData(
1415:                                        0.973868e6,
1416:                                        0.960596010000000056e18,
1417:                                        1.039928808315135234e18,
1418:                                        0.967525e6,
1419:                                        0.950990049900000023e18,
1420:                                        1.049824804368118425e18,
1421:                                        1e18
1422:                                    );
1423:                                }
1424:                            }
1425:                        } else {
1426:                            if (price < 0.986773e6) {
1427:                                if (price < 0.980283e6) {
1428:                                    return PriceData(
1429:                                        0.980283e6,
1430:                                        0.970299000000000134e18,
1431:                                        1.029998108905910481e18,
1432:                                        0.973868e6,
1433:                                        0.960596010000000056e18,
1434:                                        1.039928808315135234e18,
1435:                                        1e18
1436:                                    );
1437:                                } else {
1438:                                    return PriceData(
1439:                                        0.986773e6,
1440:                                        0.980099999999999971e18,
1441:                                        1.020032908506394831e18,
1442:                                        0.980283e6,
1443:                                        0.970299000000000134e18,
1444:                                        1.029998108905910481e18,
1445:                                        1e18
1446:                                    );
1447:                                }
1448:                            } else {
1449:                                return PriceData(
1450:                                    0.993344e6,
1451:                                    0.989999999999999991e18,
1452:                                    1.01003344631248293e18,
1453:                                    0.986773e6,
1454:                                    0.980099999999999971e18,
1455:                                    1.020032908506394831e18,
1456:                                    1e18
1457:                                );
1458:                            }
1459:                        }
1460:                    }
1461:                }
1462:            }
1463:        } else {
1464:            if (price < 1.211166e6) {
1465:                if (price < 1.09577e6) {
1466:                    if (price < 1.048893e6) {
1467:                        if (price < 1.027293e6) {
1468:                            if (price < 1.01345e6) {
1469:                                if (price < 1.006679e6) {
1470:                                    return PriceData(
1471:                                        1.006679e6,
1472:                                        1.010000000000000009e18,
1473:                                        0.990033224058159078e18,
1474:                                        0.993344e6,
1475:                                        0.989999999999999991e18,
1476:                                        1.01003344631248293e18,
1477:                                        1e18
1478:                                    );
1479:                                } else {
1480:                                    return PriceData(
1481:                                        1.01345e6,
1482:                                        1.020100000000000007e18,
1483:                                        0.980033797419900599e18,
1484:                                        1.006679e6,
1485:                                        1.010000000000000009e18,
1486:                                        0.990033224058159078e18,
1487:                                        1e18
1488:                                    );
1489:                                }
1490:                            } else {
1491:                                if (price < 1.020319e6) {
1492:                                    return PriceData(
1493:                                        1.020319e6,
1494:                                        1.030300999999999911e18,
1495:                                        0.970002111104709575e18,
1496:                                        1.01345e6,
1497:                                        1.020100000000000007e18,
1498:                                        0.980033797419900599e18,
1499:                                        1e18
1500:                                    );
1501:                                } else {
1502:                                    return PriceData(
1503:                                        1.027293e6,
1504:                                        1.040604010000000024e18,
1505:                                        0.959938599971011053e18,
1506:                                        1.020319e6,
1507:                                        1.030300999999999911e18,
1508:                                        0.970002111104709575e18,
1509:                                        1e18
1510:                                    );
1511:                                }
1512:                            }
1513:                        } else {
1514:                            if (price < 1.034375e6) {
1515:                                if (price < 1.033686e6) {
1516:                                    return PriceData(
1517:                                        1.033686e6,
1518:                                        1.050000000000000044e18,
1519:                                        0.950820553711780869e18,
1520:                                        1.027293e6,
1521:                                        1.040604010000000024e18,
1522:                                        0.959938599971011053e18,
1523:                                        1e18
1524:                                    );
1525:                                } else {
1526:                                    return PriceData(
1527:                                        1.034375e6,
1528:                                        1.051010050100000148e18,
1529:                                        0.949843744564435544e18,
1530:                                        1.033686e6,
1531:                                        1.050000000000000044e18,
1532:                                        0.950820553711780869e18,
1533:                                        1e18
1534:                                    );
1535:                                }
1536:                            } else {
1537:                                if (price < 1.041574e6) {
1538:                                    return PriceData(
1539:                                        1.041574e6,
1540:                                        1.061520150601000134e18,
1541:                                        0.93971807302139454e18,
1542:                                        1.034375e6,
1543:                                        1.051010050100000148e18,
1544:                                        0.949843744564435544e18,
1545:                                        1e18
1546:                                    );
1547:                                } else {
1548:                                    return PriceData(
1549:                                        1.048893e6,
1550:                                        1.072135352107010053e18,
1551:                                        0.929562163027227939e18,
1552:                                        1.041574e6,
1553:                                        1.061520150601000134e18,
1554:                                        0.93971807302139454e18,
1555:                                        1e18
1556:                                    );
1557:                                }
1558:                            }
1559:                        }
1560:                    } else {
1561:                        if (price < 1.071652e6) {
1562:                            if (price < 1.063925e6) {
1563:                                if (price < 1.056342e6) {
1564:                                    return PriceData(
1565:                                        1.056342e6,
1566:                                        1.082856705628080007e18,
1567:                                        0.919376643827810258e18,
1568:                                        1.048893e6,
1569:                                        1.072135352107010053e18,
1570:                                        0.929562163027227939e18,
1571:                                        1e18
1572:                                    );
1573:                                } else {
1574:                                    return PriceData(
1575:                                        1.063925e6,
1576:                                        1.093685272684360887e18,
1577:                                        0.90916219829307332e18,
1578:                                        1.056342e6,
1579:                                        1.082856705628080007e18,
1580:                                        0.919376643827810258e18,
1581:                                        1e18
1582:                                    );
1583:                                }
1584:                            } else {
1585:                                if (price < 1.070147e6) {
1586:                                    return PriceData(
1587:                                        1.070147e6,
1588:                                        1.102500000000000036e18,
1589:                                        0.900901195775543062e18,
1590:                                        1.063925e6,
1591:                                        1.093685272684360887e18,
1592:                                        0.90916219829307332e18,
1593:                                        1e18
1594:                                    );
1595:                                } else {
1596:                                    return PriceData(
1597:                                        1.071652e6,
1598:                                        1.104622125411204525e18,
1599:                                        0.89891956503043724e18,
1600:                                        1.070147e6,
1601:                                        1.102500000000000036e18,
1602:                                        0.900901195775543062e18,
1603:                                        1e18
1604:                                    );
1605:                                }
1606:                            }
1607:                        } else {
1608:                            if (price < 1.087566e6) {
1609:                                if (price < 1.079529e6) {
1610:                                    return PriceData(
1611:                                        1.079529e6,
1612:                                        1.115668346665316557e18,
1613:                                        0.888649540545595529e18,
1614:                                        1.071652e6,
1615:                                        1.104622125411204525e18,
1616:                                        0.89891956503043724e18,
1617:                                        1e18
1618:                                    );
1619:                                } else {
1620:                                    return PriceData(
1621:                                        1.087566e6,
1622:                                        1.126825030131969774e18,
1623:                                        0.878352981447521719e18,
1624:                                        1.079529e6,
1625:                                        1.115668346665316557e18,
1626:                                        0.888649540545595529e18,
1627:                                        1e18
1628:                                    );
1629:                                }
1630:                            } else {
1631:                                return PriceData(
1632:                                    1.09577e6,
1633:                                    1.1380932804332895e18,
1634:                                    0.868030806693890433e18,
1635:                                    1.087566e6,
1636:                                    1.126825030131969774e18,
1637:                                    0.878352981447521719e18,
1638:                                    1e18
1639:                                );
1640:                            }
1641:                        }
1642:                    }
1643:                } else {
1644:                    if (price < 1.15496e6) {
1645:                        if (price < 1.121482e6) {
1646:                            if (price < 1.110215e6) {
1647:                                if (price < 1.104151e6) {
1648:                                    return PriceData(
1649:                                        1.104151e6,
1650:                                        1.149474213237622333e18,
1651:                                        0.857683999872391523e18,
1652:                                        1.09577e6,
1653:                                        1.1380932804332895e18,
1654:                                        0.868030806693890433e18,
1655:                                        1e18
1656:                                    );
1657:                                } else {
1658:                                    return PriceData(
1659:                                        1.110215e6,
1660:                                        1.157625000000000126e18,
1661:                                        0.850322213751246947e18,
1662:                                        1.104151e6,
1663:                                        1.149474213237622333e18,
1664:                                        0.857683999872391523e18,
1665:                                        1e18
1666:                                    );
1667:                                }
1668:                            } else {
1669:                                if (price < 1.112718e6) {
1670:                                    return PriceData(
1671:                                        1.112718e6,
1672:                                        1.160968955369998667e18,
1673:                                        0.847313611512600207e18,
1674:                                        1.110215e6,
1675:                                        1.157625000000000126e18,
1676:                                        0.850322213751246947e18,
1677:                                        1e18
1678:                                    );
1679:                                } else {
1680:                                    return PriceData(
1681:                                        1.121482e6,
1682:                                        1.172578644923698565e18,
1683:                                        0.836920761422192294e18,
1684:                                        1.112718e6,
1685:                                        1.160968955369998667e18,
1686:                                        0.847313611512600207e18,
1687:                                        1e18
1688:                                    );
1689:                                }
1690:                            }
1691:                        } else {
1692:                            if (price < 1.139642e6) {
1693:                                if (price < 1.130452e6) {
1694:                                    return PriceData(
1695:                                        1.130452e6,
1696:                                        1.184304431372935618e18,
1697:                                        0.826506641040327228e18,
1698:                                        1.121482e6,
1699:                                        1.172578644923698565e18,
1700:                                        0.836920761422192294e18,
1701:                                        1e18
1702:                                    );
1703:                                } else {
1704:                                    return PriceData(
1705:                                        1.139642e6,
1706:                                        1.196147475686665018e18,
1707:                                        0.8160725157999702e18,
1708:                                        1.130452e6,
1709:                                        1.184304431372935618e18,
1710:                                        0.826506641040327228e18,
1711:                                        1e18
1712:                                    );
1713:                                }
1714:                            } else {
1715:                                if (price < 1.149062e6) {
1716:                                    return PriceData(
1717:                                        1.149062e6,
1718:                                        1.208108950443531393e18,
1719:                                        0.805619727489791271e18,
1720:                                        1.139642e6,
1721:                                        1.196147475686665018e18,
1722:                                        0.8160725157999702e18,
1723:                                        1e18
1724:                                    );
1725:                                } else {
1726:                                    return PriceData(
1727:                                        1.15496e6,
1728:                                        1.21550625000000001e18,
1729:                                        0.799198479643147719e18,
1730:                                        1.149062e6,
1731:                                        1.208108950443531393e18,
1732:                                        0.805619727489791271e18,
1733:                                        1e18
1734:                                    );
1735:                                }
1736:                            }
1737:                        }
1738:                    } else {
1739:                        if (price < 1.189304e6) {
1740:                            if (price < 1.168643e6) {
1741:                                if (price < 1.158725e6) {
1742:                                    return PriceData(
1743:                                        1.158725e6,
1744:                                        1.22019003994796682e18,
1745:                                        0.795149696605042422e18,
1746:                                        1.15496e6,
1747:                                        1.21550625000000001e18,
1748:                                        0.799198479643147719e18,
1749:                                        1e18
1750:                                    );
1751:                                } else {
1752:                                    return PriceData(
1753:                                        1.168643e6,
1754:                                        1.232391940347446369e18,
1755:                                        0.784663924675502389e18,
1756:                                        1.158725e6,
1757:                                        1.22019003994796682e18,
1758:                                        0.795149696605042422e18,
1759:                                        1e18
1760:                                    );
1761:                                }
1762:                            } else {
1763:                                if (price < 1.178832e6) {
1764:                                    return PriceData(
1765:                                        1.178832e6,
1766:                                        1.244715859750920917e18,
1767:                                        0.774163996557160172e18,
1768:                                        1.168643e6,
1769:                                        1.232391940347446369e18,
1770:                                        0.784663924675502389e18,
1771:                                        1e18
1772:                                    );
1773:                                } else {
1774:                                    return PriceData(
1775:                                        1.189304e6,
1776:                                        1.257163018348430139e18,
1777:                                        0.763651582672810969e18,
1778:                                        1.178832e6,
1779:                                        1.244715859750920917e18,
1780:                                        0.774163996557160172e18,
1781:                                        1e18
1782:                                    );
1783:                                }
1784:                            }
1785:                        } else {
1786:                            if (price < 1.205768e6) {
1787:                                if (price < 1.200076e6) {
1788:                                    return PriceData(
1789:                                        1.200076e6,
1790:                                        1.269734648531914534e18,
1791:                                        0.753128441185147435e18,
1792:                                        1.189304e6,
1793:                                        1.257163018348430139e18,
1794:                                        0.763651582672810969e18,
1795:                                        1e18
1796:                                    );
1797:                                } else {
1798:                                    return PriceData(
1799:                                        1.205768e6,
1800:                                        1.276281562499999911e18,
1801:                                        0.747685899578659385e18,
1802:                                        1.200076e6,
1803:                                        1.269734648531914534e18,
1804:                                        0.753128441185147435e18,
1805:                                        1e18
1806:                                    );
1807:                                }
1808:                            } else {
1809:                                return PriceData(
1810:                                    1.211166e6,
1811:                                    1.282431995017233595e18,
1812:                                    0.74259642008426785e18,
1813:                                    1.205768e6,
1814:                                    1.276281562499999911e18,
1815:                                    0.747685899578659385e18,
1816:                                    1e18
1817:                                );
1818:                            }
1819:                        }
1820:                    }
1821:                }
1822:            } else {
1823:                if (price < 1.393403e6) {
1824:                    if (price < 1.299217e6) {
1825:                        if (price < 1.259043e6) {
1826:                            if (price < 1.234362e6) {
1827:                                if (price < 1.222589e6) {
1828:                                    return PriceData(
1829:                                        1.222589e6,
1830:                                        1.295256314967406119e18,
1831:                                        0.732057459169776381e18,
1832:                                        1.211166e6,
1833:                                        1.282431995017233595e18,
1834:                                        0.74259642008426785e18,
1835:                                        1e18
1836:                                    );
1837:                                } else {
1838:                                    return PriceData(
1839:                                        1.234362e6,
1840:                                        1.308208878117080198e18,
1841:                                        0.721513591905860174e18,
1842:                                        1.222589e6,
1843:                                        1.295256314967406119e18,
1844:                                        0.732057459169776381e18,
1845:                                        1e18
1846:                                    );
1847:                                }
1848:                            } else {
1849:                                if (price < 1.246507e6) {
1850:                                    return PriceData(
1851:                                        1.246507e6,
1852:                                        1.321290966898250874e18,
1853:                                        0.710966947125877935e18,
1854:                                        1.234362e6,
1855:                                        1.308208878117080198e18,
1856:                                        0.721513591905860174e18,
1857:                                        1e18
1858:                                    );
1859:                                } else {
1860:                                    return PriceData(
1861:                                        1.259043e6,
1862:                                        1.33450387656723346e18,
1863:                                        0.700419750561125598e18,
1864:                                        1.246507e6,
1865:                                        1.321290966898250874e18,
1866:                                        0.710966947125877935e18,
1867:                                        1e18
1868:                                    );
1869:                                }
1870:                            }
1871:                        } else {
1872:                            if (price < 1.271991e6) {
1873:                                if (price < 1.264433e6) {
1874:                                    return PriceData(
1875:                                        1.264433e6,
1876:                                        1.340095640624999973e18,
1877:                                        0.695987932996588454e18,
1878:                                        1.259043e6,
1879:                                        1.33450387656723346e18,
1880:                                        0.700419750561125598e18,
1881:                                        1e18
1882:                                    );
1883:                                } else {
1884:                                    return PriceData(
1885:                                        1.271991e6,
1886:                                        1.347848915332905628e18,
1887:                                        0.689874326166576179e18,
1888:                                        1.264433e6,
1889:                                        1.340095640624999973e18,
1890:                                        0.695987932996588454e18,
1891:                                        1e18
1892:                                    );
1893:                                }
1894:                            } else {
1895:                                if (price < 1.285375e6) {
1896:                                    return PriceData(
1897:                                        1.285375e6,
1898:                                        1.361327404486234682e18,
1899:                                        0.67933309721453039e18,
1900:                                        1.271991e6,
1901:                                        1.347848915332905628e18,
1902:                                        0.689874326166576179e18,
1903:                                        1e18
1904:                                    );
1905:                                } else {
1906:                                    return PriceData(
1907:                                        1.299217e6,
1908:                                        1.374940678531097138e18,
1909:                                        0.668798587125333244e18,
1910:                                        1.285375e6,
1911:                                        1.361327404486234682e18,
1912:                                        0.67933309721453039e18,
1913:                                        1e18
1914:                                    );
1915:                                }
1916:                            }
1917:                        }
1918:                    } else {
1919:                        if (price < 1.343751e6) {
1920:                            if (price < 1.328377e6) {
1921:                                if (price < 1.313542e6) {
1922:                                    return PriceData(
1923:                                        1.313542e6,
1924:                                        1.38869008531640814e18,
1925:                                        0.658273420002602916e18,
1926:                                        1.299217e6,
1927:                                        1.374940678531097138e18,
1928:                                        0.668798587125333244e18,
1929:                                        1e18
1930:                                    );
1931:                                } else {
1932:                                    return PriceData(
1933:                                        1.328377e6,
1934:                                        1.402576986169572049e18,
1935:                                        0.647760320838866033e18,
1936:                                        1.313542e6,
1937:                                        1.38869008531640814e18,
1938:                                        0.658273420002602916e18,
1939:                                        1e18
1940:                                    );
1941:                                }
1942:                            } else {
1943:                                if (price < 1.333292e6) {
1944:                                    return PriceData(
1945:                                        1.333292e6,
1946:                                        1.407100422656250016e18,
1947:                                        0.644361360672887962e18,
1948:                                        1.328377e6,
1949:                                        1.402576986169572049e18,
1950:                                        0.647760320838866033e18,
1951:                                        1e18
1952:                                    );
1953:                                } else {
1954:                                    return PriceData(
1955:                                        1.343751e6,
1956:                                        1.416602756031267951e18,
1957:                                        0.637262115356114656e18,
1958:                                        1.333292e6,
1959:                                        1.407100422656250016e18,
1960:                                        0.644361360672887962e18,
1961:                                        1e18
1962:                                    );
1963:                                }
1964:                            }
1965:                        } else {
1966:                            if (price < 1.376232e6) {
1967:                                if (price < 1.359692e6) {
1968:                                    return PriceData(
1969:                                        1.359692e6,
1970:                                        1.430768783591580551e18,
1971:                                        0.626781729444674585e18,
1972:                                        1.343751e6,
1973:                                        1.416602756031267951e18,
1974:                                        0.637262115356114656e18,
1975:                                        1e18
1976:                                    );
1977:                                } else {
1978:                                    return PriceData(
1979:                                        1.376232e6,
1980:                                        1.445076471427496179e18,
1981:                                        0.616322188162944262e18,
1982:                                        1.359692e6,
1983:                                        1.430768783591580551e18,
1984:                                        0.626781729444674585e18,
1985:                                        1e18
1986:                                    );
1987:                                }
1988:                            } else {
1989:                                return PriceData(
1990:                                    1.393403e6,
1991:                                    1.459527236141771489e18,
1992:                                    0.605886614260108591e18,
1993:                                    1.376232e6,
1994:                                    1.445076471427496179e18,
1995:                                    0.616322188162944262e18,
1996:                                    1e18
1997:                                );
1998:                            }
1999:                        }
2000:                    }
2001:                } else {
2002:                    if (price < 2.209802e6) {
2003:                        if (price < 1.514667e6) {
2004:                            if (price < 1.415386e6) {
2005:                                if (price < 1.41124e6) {
2006:                                    return PriceData(
2007:                                        1.41124e6,
2008:                                        1.474122508503188822e18,
2009:                                        0.595478226183906334e18,
2010:                                        1.393403e6,
2011:                                        1.459527236141771489e18,
2012:                                        0.605886614260108591e18,
2013:                                        1e18
2014:                                    );
2015:                                } else {
2016:                                    return PriceData(
2017:                                        1.415386e6,
2018:                                        1.47745544378906235e18,
2019:                                        0.593119977480511928e18,
2020:                                        1.41124e6,
2021:                                        1.474122508503188822e18,
2022:                                        0.595478226183906334e18,
2023:                                        1e18
2024:                                    );
2025:                                }
2026:                            } else {
2027:                                if (price < 1.42978e6) {
2028:                                    return PriceData(
2029:                                        1.42978e6,
2030:                                        1.488863733588220883e18,
2031:                                        0.585100335536025584e18,
2032:                                        1.415386e6,
2033:                                        1.47745544378906235e18,
2034:                                        0.593119977480511928e18,
2035:                                        1e18
2036:                                    );
2037:                                } else {
2038:                                    return PriceData(
2039:                                        1.514667e6,
2040:                                        1.551328215978515557e18,
2041:                                        0.54263432113736132e18,
2042:                                        1.42978e6,
2043:                                        1.488863733588220883e18,
2044:                                        0.585100335536025584e18,
2045:                                        1e18
2046:                                    );
2047:                                }
2048:                            }
2049:                        } else {
2050:                            if (price < 1.786708e6) {
2051:                                if (price < 1.636249e6) {
2052:                                    return PriceData(
2053:                                        1.636249e6,
2054:                                        1.628894626777441568e18,
2055:                                        0.493325115988533236e18,
2056:                                        1.514667e6,
2057:                                        1.551328215978515557e18,
2058:                                        0.54263432113736132e18,
2059:                                        1e18
2060:                                    );
2061:                                } else {
2062:                                    return PriceData(
2063:                                        1.786708e6,
2064:                                        1.710339358116313546e18,
2065:                                        0.445648172809785581e18,
2066:                                        1.636249e6,
2067:                                        1.628894626777441568e18,
2068:                                        0.493325115988533236e18,
2069:                                        1e18
2070:                                    );
2071:                                }
2072:                            } else {
2073:                                if (price < 1.974398e6) {
2074:                                    return PriceData(
2075:                                        1.974398e6,
2076:                                        1.79585632602212919e18,
2077:                                        0.400069510798421513e18,
2078:                                        1.786708e6,
2079:                                        1.710339358116313546e18,
2080:                                        0.445648172809785581e18,
2081:                                        1e18
2082:                                    );
2083:                                } else {
2084:                                    return PriceData(
2085:                                        2.209802e6,
2086:                                        1.885649142323235772e18,
2087:                                        0.357031765135700119e18,
2088:                                        1.974398e6,
2089:                                        1.79585632602212919e18,
2090:                                        0.400069510798421513e18,
2091:                                        1e18
2092:                                    );
2093:                                }
2094:                            }
2095:                        }
2096:                    } else {
2097:                        if (price < 3.931396e6) {
2098:                            if (price < 2.878327e6) {
2099:                                if (price < 2.505865e6) {
2100:                                    return PriceData(
2101:                                        2.505865e6,
2102:                                        1.97993159943939756e18,
2103:                                        0.316916199929126341e18,
2104:                                        2.209802e6,
2105:                                        1.885649142323235772e18,
2106:                                        0.357031765135700119e18,
2107:                                        1e18
2108:                                    );
2109:                                } else {
2110:                                    return PriceData(
2111:                                        2.878327e6,
2112:                                        2.078928179411367427e18,
2113:                                        0.28000760254479623e18,
2114:                                        2.505865e6,
2115:                                        1.97993159943939756e18,
2116:                                        0.316916199929126341e18,
2117:                                        1e18
2118:                                    );
2119:                                }
2120:                            } else {
2121:                                if (price < 3.346057e6) {
2122:                                    return PriceData(
2123:                                        3.346057e6,
2124:                                        2.182874588381935599e18,
2125:                                        0.246470170347584949e18,
2126:                                        2.878327e6,
2127:                                        2.078928179411367427e18,
2128:                                        0.28000760254479623e18,
2129:                                        1e18
2130:                                    );
2131:                                } else {
2132:                                    return PriceData(
2133:                                        3.931396e6,
2134:                                        2.292018317801032268e18,
2135:                                        0.216340086006769544e18,
2136:                                        3.346057e6,
2137:                                        2.182874588381935599e18,
2138:                                        0.246470170347584949e18,
2139:                                        1e18
2140:                                    );
2141:                                }
2142:                            }
2143:                        } else {
2144:                            if (price < 10.709509e6) {
2145:                                if (price < 4.660591e6) {
2146:                                    return PriceData(
2147:                                        4.660591e6,
2148:                                        2.406619233691083881e18,
2149:                                        0.189535571483960663e18,
2150:                                        3.931396e6,
2151:                                        2.292018317801032268e18,
2152:                                        0.216340086006769544e18,
2153:                                        1e18
2154:                                    );
2155:                                } else {
2156:                                    return PriceData(
2157:                                        10.709509e6,
2158:                                        3e18,
2159:                                        0.103912563829966526e18,
2160:                                        4.660591e6,
2161:                                        2.406619233691083881e18,
2162:                                        0.189535571483960663e18,
2163:                                        1e18
2164:                                    );
2165:                                }
2166:                            } else {
2167:                                revert("LUT: Invalid price");
2168:                            }
2169:                        }
2170:                    }
2171:                }
2172:            }
2173:        }
2174:    }
```
[27](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L27-L734), [740](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L740-L2174), 


#### Recommendation

Regularly analyze your Solidity contracts for functions with high cyclomatic complexity. Consider refactoring these functions into smaller, more focused units. This could involve breaking down complex functions into multiple simpler functions, reducing the number of conditional branches, or simplifying logic where possible. Such refactoring improves the readability and testability of your code and reduces the likelihood of defects. Additionally, simpler functions are easier to audit and maintain, enhancing the overall security and robustness of your contract. Employ tools or metrics to periodically assess the complexity of your functions as part of your development and maintenance processes.

### Consider only defining one library/interface/contract per sol file
Combining multiple libraries, interfaces, or contracts in a single .sol file can lead to clutter, reduced readability, and versioning issues. Resolution: Adopt the best practice of defining only one library, interface, or contract per Solidity file. This modular approach enhances clarity, simplifies unit testing, and streamlines code review. Furthermore, segregating components makes version management easier, as updates to one component won't necessitate changes to a file housing multiple unrelated components. Structured file management can further assist in avoiding naming collisions and ensure smoother integration into larger systems or DApps.

```solidity
Path: ./src/WellUpgradeable.sol

5:import {Well} from "src/Well.sol";	// @audit-issue
6:import {UUPSUpgradeable} from "ozu/proxy/utils/UUPSUpgradeable.sol";
7:import {OwnableUpgradeable} from "ozu/access/OwnableUpgradeable.sol";
8:import {IERC20, SafeERC20} from "oz/token/ERC20/utils/SafeERC20.sol";
9:import {IAquifer} from "src/interfaces/IAquifer.sol";
```
[5](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L5-L9), 


```solidity
Path: ./src/functions/Stable2.sol

5:import {IBeanstalkWellFunction, IMultiFlowPumpWellFunction} from "src/interfaces/IBeanstalkWellFunction.sol";	// @audit-issue
6:import {ILookupTable} from "src/interfaces/ILookupTable.sol";
7:import {ProportionalLPToken2} from "src/functions/ProportionalLPToken2.sol";
8:import {IERC20} from "forge-std/interfaces/IERC20.sol";
```
[5](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L5-L8), 


```solidity
Path: ./src/functions/StableLUT/Stable2LUT1.sol

5:import {ILookupTable} from "src/interfaces/ILookupTable.sol";	// @audit-issue
```
[5](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L5-L5), 


#### Recommendation

Adopt a modular file structure in your Solidity projects by defining only one library, interface, or contract per file. This approach significantly enhances the clarity and readability of your code, making it easier to manage, test, and review. It also simplifies version control, as updates to individual components are isolated to their respective files, reducing the risk of unintended side effects. Organize your project directory to reflect this structure, with a clear naming convention that matches file names to their contained contracts, libraries, or interfaces. This organization not only prevents naming collisions but also facilitates smoother integration into larger systems or decentralized applications (DApps).

### Reduce deployment costs by tweaking contracts' metadata
When solidity generates the bytecode for the smart contract to be deployed, it appends metadata about the compilation at the end of the bytecode.
By default, the solidity compiler appends metadata at the end of the “actual” initcode, which gets stored to the blockchain when the constructor finishes executing.
Consider tweaking the metadata to avoid this unnecessary allocation. A full guide can be found [here](https://www.rareskills.io/post/solidity-metadata).
```solidity
/// @audit Global finding.
```
[1](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L1), 


#### Recommendation

See [this](https://www.rareskills.io/post/solidity-metadata) link, at its bottom, for full details

### All verbatim blocks are considered identical by deduplicator and can incorrectly be unified
The Solidity Team reported a bug on October 24, 2023, affecting Yul code using the verbatim builtin, specifically in the Block Deduplicator optimizer step. This bug, present since Solidity version 0.8.5, caused incorrect deduplication of verbatim assembly items surrounded by identical opcodes, considering them identical regardless of their data. The bug was confined to pure Yul compilation with optimization enabled and was unlikely to be exploited as an attack vector. The conditions triggering the bug were very specific, and its occurrence was deemed to have a low likelihood. The bug was rated with an overall low score due to these factors.
```solidity
/// @audit Global finding.
```
[1](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L1), 


#### Recommendation

Review and assess any Solidity contracts, especially those involving Yul code, that may be impacted by this deduplication bug. If your contracts rely on the Block Deduplicator optimizer and use verbatim blocks in a way that could be affected by this issue, consider updating your Solidity version to one where this bug is fixed, or adjust your contract to avoid this specific scenario. Stay informed about updates from the Solidity Team regarding this and similar issues, and regularly update your Solidity compiler to the latest version to benefit from bug fixes and optimizations. Given the specific and limited nature of this bug, its impact may be minimal, but caution is advised for contracts with complex assembly code or those heavily reliant on optimizer behaviors.

### Consider adding formal verification proofs
Formal verification is the act of proving or disproving the correctness of intended algorithms underlying a system with respect to a certain formal specification/property/invariant, using formal methods of mathematics.
```solidity
/// @audit Global finding.
```
[1](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L1), 


#### Recommendation

Consider integrating formal verification into your Solidity contract development process. This can be done by defining formal specifications and properties that your contract should adhere to and using mathematical methods to verify these aspects. Tools and platforms like Certora Prover, Scribble, or OpenZeppelin's test environment can assist in this process. Formal verification should complement traditional testing and auditing methods, offering an additional layer of security assurance. Keep in mind that formal verification requires a thorough understanding of mathematical logic and contract specifications, so it may necessitate additional resources or expertise. Nevertheless, the investment in formal verification can significantly enhance the trustworthiness and robustness of your smart contracts.

### Contracts should have full test coverage
While 100% code coverage does not guarantee that there are no bugs, it often will catch easy-to-find bugs, and will ensure that there are fewer regressions when the code invariably has to be modified. Furthermore, in order to get full coverage, code authors will often have to re-organize their code so that it is more modular, so that each component can be tested separately, which reduces interdependencies between modules and layers, and makes for code that is easier to reason about and audit.
```solidity
/// @audit Global finding.
```
[1](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L1), 


#### Recommendation

Consider writing test cases.

### Large or complicated code bases should implement invariant tests
This includes: large code bases, or code with lots of inline-assembly, complicated math, or complicated interactions between multiple contracts. Invariant fuzzers such as Echidna require the test writer to come up with invariants which should not be violated under any circumstances, and the fuzzer tests various inputs and function calls to ensure that the invariants always hold. Even code with 100% code coverage can still have bugs due to the order of the operations a user performs, and invariant fuzzers may help significantly.
```solidity
/// @audit Global finding.
```
[1](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L1), 


#### Recommendation

Consider writing invariant test cases.

### Consider adding formal verification proofs
Consider using formal verification to mathematically prove that your code does what is intended, and does not have any edge cases with unexpected behavior. The solidity compiler itself has this functionality [built in](https://docs.soliditylang.org/en/latest/smtchecker.html#smtchecker-and-formal-verification)
```solidity
/// @audit Global finding.
```
[1](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L1), 


#### Recommendation

Consider using formal verification.

### NatSpec: Body of `if` statement should be placed on a new line
According to the [Solidity style guide](https://docs.soliditylang.org/en/latest/style-guide.html#control-structures), `if` statements whose body contains a single line should look like this: solidity `if (x < 10)     x += 1; `

```solidity
Path: ./src/functions/Stable2.sol

61:        if (lut == address(0)) revert InvalidLUT();	// @audit-issue

78:        if (reserves[0] == 0 && reserves[1] == 0) return 0;	// @audit-issue

86:        if (sumReserves == 0) return 0;	// @audit-issue

98:                if (lpTokenSupply - prevReserves <= 1) return lpTokenSupply;	// @audit-issue

100:                if (prevReserves - lpTokenSupply <= 1) return lpTokenSupply;	// @audit-issue

320:        if (decimal0 > 18 || decimal1 > 18) revert InvalidTokenDecimals();	// @audit-issue
```
[61](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L61-L61), [78](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L78-L78), [86](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L86-L86), [98](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L98-L98), [100](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L100-L100), [320](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L320-L320), 


#### Recommendation

Adhere to the Solidity style guide by formatting single-line `if` statements with the body on the same line as the condition. For example, use `if (x < 10) x += 1;` for concise and clear code. This style of formatting enhances readability and maintains consistency with Solidity's recommended best practices. It's particularly effective for simple and short conditional operations within the contract's code.

### NatSpec: Contract declarations should have `@dev` tag
NatSpec comments are a critical part of Solidity's documentation system, designed to help developers and others understand the behavior and purpose of a contract. The `@dev` tag, in particular, provides context and insight into the contract's development considerations. A missing `@dev` comment can lead to misunderstandings about the contract, making it harder for others to contribute to or use the contract effectively. Therefore, it's highly recommended to include `@dev` comments in the documentation to enhance code readability and maintainability. [Dive Deeper into NatSpec Guidelines](https://docs.soliditylang.org/en/develop/natspec-format.html)

```solidity
Path: ./src/WellUpgradeable.sol

16:contract WellUpgradeable is Well, UUPSUpgradeable, OwnableUpgradeable {	// @audit-issue missing `@dev` tag
```
[16](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L16-L16), 


```solidity
Path: ./src/functions/StableLUT/Stable2LUT1.sol

13:contract Stable2LUT1 is ILookupTable {	// @audit-issue missing `@dev` tag
```
[13](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L13-L13), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).

### NatSpec: Contract declarations should have `@notice` tag
The `@notice` is used to explain to users what the contract does. The compiler interprets `///` or `/**` comments [as this tag](https://docs.soliditylang.org/en/latest/natspec-format.html#tags) if one wasn't explicitly provided.

```solidity
Path: ./src/functions/Stable2.sol

25:contract Stable2 is ProportionalLPToken2, IBeanstalkWellFunction {	// @audit-issue missing `@notice` tag
```
[25](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L25-L25), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).

### NatSpec: Use `@inheritdoc` for overriden functions.
Natural Specification (NatSpec) comments are crucial for understanding the role of function arguments in your Solidity code. Including @param tags will not only improve your code's readability but also its maintainability by clearly defining each argument's purpose. [Dive Deeper into NatSpec Guidelines](https://docs.soliditylang.org/en/develop/natspec-format.html)

```solidity
Path: ./src/WellUpgradeable.sol

33:    function init(string memory _name, string memory _symbol) external override reinitializer(2) {	// @audit-issue missing `@inheritdoc` tag

65:    function _authorizeUpgrade(address newImplmentation) internal view override {	// @audit-issue missing `@inheritdoc` tag

93:    function upgradeTo(address newImplementation) public override {	// @audit-issue missing `@inheritdoc` tag

104:    function upgradeToAndCall(address newImplementation, bytes memory data) public payable override {	// @audit-issue missing `@inheritdoc` tag

118:    function proxiableUUID() external view override notDelegatedOrIsMinimalProxy returns (bytes32) {	// @audit-issue missing `@inheritdoc` tag
```
[33](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L33-L33), [65](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L65-L65), [93](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L93-L93), [104](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L104-L104), [118](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L118-L118), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).

### NatSpec: Function `@return` tag is missing
Natural Specification (NatSpec) comments are crucial for understanding the role of function arguments in your Solidity code. Including `@return` tag will not only improve your code's readability but also its maintainability by clearly defining each argument's purpose. [Dive Deeper into NatSpec Guidelines](https://docs.soliditylang.org/en/develop/natspec-format.html)

```solidity
Path: ./src/WellUpgradeable.sol

118:    function proxiableUUID() external view override notDelegatedOrIsMinimalProxy returns (bytes32) {	// @audit-issue missing `@return` tag

122:    function getImplementation() external view returns (address) {	// @audit-issue missing `@return` tag

126:    function getVersion() external pure virtual returns (uint256) {	// @audit-issue missing `@return` tag

130:    function getInitializerVersion() external view returns (uint256) {	// @audit-issue missing `@return` tag
```
[118](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L118-L118), [122](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L122-L122), [126](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L126-L126), [130](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L130-L130), 


```solidity
Path: ./src/functions/Stable2.sol

74:    function calcLpTokenSupply(	// @audit-issue missing `@return` tag

114:    function calcReserve(	// @audit-issue missing `@return` tag

153:    function calcRate(	// @audit-issue missing `@return` tag

173:    function calcReserveAtRatioSwap(	// @audit-issue missing `@return` tag

246:    function calcReserveAtRatioLiquidity(	// @audit-issue missing `@return` tag

327:    function name() external pure returns (string memory) {	// @audit-issue missing `@return` tag

331:    function symbol() external pure returns (string memory) {	// @audit-issue missing `@return` tag

338:    function _calcRate(	// @audit-issue missing `@return` tag

357:    function getScaledReserves(	// @audit-issue missing `@return` tag

366:    function _calcReserve(	// @audit-issue missing `@return` tag

375:    function getBandC(	// @audit-issue missing `@return` tag

387:    function updateReserve(PriceData memory pd, uint256 reserve) internal pure returns (uint256) {	// @audit-issue missing `@return` tag
```
[74](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L74-L74), [114](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L114-L114), [153](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L153-L153), [173](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L173-L173), [246](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L246-L246), [327](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L327-L327), [331](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L331-L331), [338](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L338-L338), [357](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L357-L357), [366](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L366-L366), [375](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L375-L375), [387](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L387-L387), 


```solidity
Path: ./src/functions/StableLUT/Stable2LUT1.sol

27:    function getRatiosFromPriceLiquidity(uint256 price) external pure returns (PriceData memory) {	// @audit-issue missing `@return` tag

740:    function getRatiosFromPriceSwap(uint256 price) external pure returns (PriceData memory) {	// @audit-issue missing `@return` tag
```
[27](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L27-L27), [740](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L740-L740), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).

### NatSpec: Function declarations should have `@notice` tag
The `@notice` tag in NatSpec comments is used to provide important explanations to end users about what a function does. It appears that this contract's function declarations are missing `@notice` tags in their NatSpec annotations.

The absence of `@notice` tags reduces the contract's transparency and could lead to misunderstandings about a function's purpose and behavior.  [Dive Deeper into NatSpec Guidelines](https://docs.soliditylang.org/en/develop/natspec-format.html)

```solidity
Path: ./src/WellUpgradeable.sol

33:    function init(string memory _name, string memory _symbol) external override reinitializer(2) {	// @audit-issue missing `@notice` tag

118:    function proxiableUUID() external view override notDelegatedOrIsMinimalProxy returns (bytes32) {	// @audit-issue missing `@notice` tag

122:    function getImplementation() external view returns (address) {	// @audit-issue missing `@notice` tag

126:    function getVersion() external pure virtual returns (uint256) {	// @audit-issue missing `@notice` tag

130:    function getInitializerVersion() external view returns (uint256) {	// @audit-issue missing `@notice` tag
```
[33](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L33-L33), [118](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L118-L118), [122](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L122-L122), [126](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L126-L126), [130](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L130-L130), 


```solidity
Path: ./src/functions/Stable2.sol

60:    constructor(address lut) {	// @audit-issue missing `@notice` tag

153:    function calcRate(	// @audit-issue missing `@notice` tag

173:    function calcReserveAtRatioSwap(	// @audit-issue missing `@notice` tag

246:    function calcReserveAtRatioLiquidity(	// @audit-issue missing `@notice` tag

327:    function name() external pure returns (string memory) {	// @audit-issue missing `@notice` tag

331:    function symbol() external pure returns (string memory) {	// @audit-issue missing `@notice` tag

366:    function _calcReserve(	// @audit-issue missing `@notice` tag

375:    function getBandC(	// @audit-issue missing `@notice` tag
```
[60](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L60-L60), [153](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L153-L153), [173](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L173-L173), [246](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L246-L246), [327](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L327-L327), [331](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L331-L331), [366](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L366-L366), [375](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L375-L375), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).

### NatSpec: Function declarations should have `@dev` tag
Some functions have an incomplete NatSpec: add a `@dev` notation to describe the function to improve the code documentation.

```solidity
Path: ./src/WellUpgradeable.sol

33:    function init(string memory _name, string memory _symbol) external override reinitializer(2) {	// @audit-issue missing `@dev` tag

54:    function initNoWellToken() external initializer {}	// @audit-issue missing `@dev` tag

65:    function _authorizeUpgrade(address newImplmentation) internal view override {	// @audit-issue missing `@dev` tag

122:    function getImplementation() external view returns (address) {	// @audit-issue missing `@dev` tag

126:    function getVersion() external pure virtual returns (uint256) {	// @audit-issue missing `@dev` tag

130:    function getInitializerVersion() external view returns (uint256) {	// @audit-issue missing `@dev` tag
```
[33](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L33-L33), [54](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L54-L54), [65](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L65-L65), [122](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L122-L122), [126](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L126-L126), [130](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L130-L130), 


```solidity
Path: ./src/functions/Stable2.sol

60:    constructor(address lut) {	// @audit-issue missing `@dev` tag

74:    function calcLpTokenSupply(	// @audit-issue missing `@dev` tag

310:    function decodeWellData(bytes memory data) public view virtual returns (uint256[] memory decimals) {	// @audit-issue missing `@dev` tag

327:    function name() external pure returns (string memory) {	// @audit-issue missing `@dev` tag

331:    function symbol() external pure returns (string memory) {	// @audit-issue missing `@dev` tag

338:    function _calcRate(	// @audit-issue missing `@dev` tag

366:    function _calcReserve(	// @audit-issue missing `@dev` tag

375:    function getBandC(	// @audit-issue missing `@dev` tag

387:    function updateReserve(PriceData memory pd, uint256 reserve) internal pure returns (uint256) {	// @audit-issue missing `@dev` tag
```
[60](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L60-L60), [74](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L74-L74), [310](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L310-L310), [327](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L327-L327), [331](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L331-L331), [338](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L338-L338), [366](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L366-L366), [375](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L375-L375), [387](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L387-L387), 


```solidity
Path: ./src/functions/StableLUT/Stable2LUT1.sol

27:    function getRatiosFromPriceLiquidity(uint256 price) external pure returns (PriceData memory) {	// @audit-issue missing `@dev` tag

740:    function getRatiosFromPriceSwap(uint256 price) external pure returns (PriceData memory) {	// @audit-issue missing `@dev` tag
```
[27](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L27-L27), [740](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L740-L740), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).

### NatSpec: Function/Constructor `@param` tag is missing
Natural Specification (NatSpec) comments are crucial for understanding the role of function arguments in your Solidity code. Including @param tags will not only improve your code's readability but also its maintainability by clearly defining each argument's purpose. [Dive Deeper into NatSpec Guidelines](https://docs.soliditylang.org/en/develop/natspec-format.html)

```solidity
Path: ./src/WellUpgradeable.sol

33:    function init(string memory _name, string memory _symbol) external override reinitializer(2) {	// @audit-issue missing `@param` tag

65:    function _authorizeUpgrade(address newImplmentation) internal view override {	// @audit-issue missing `@param` tag

93:    function upgradeTo(address newImplementation) public override {	// @audit-issue missing `@param` tag

104:    function upgradeToAndCall(address newImplementation, bytes memory data) public payable override {	// @audit-issue missing `@param` tag
```
[33](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L33-L33), [65](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L65-L65), [93](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L93-L93), [104](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L104-L104), 


```solidity
Path: ./src/functions/Stable2.sol

60:    constructor(address lut) {	// @audit-issue missing `@param` tag

74:    function calcLpTokenSupply(	// @audit-issue missing `@param` tag

114:    function calcReserve(	// @audit-issue missing `@param` tag

153:    function calcRate(	// @audit-issue missing `@param` tag

173:    function calcReserveAtRatioSwap(	// @audit-issue missing `@param` tag

246:    function calcReserveAtRatioLiquidity(	// @audit-issue missing `@param` tag

310:    function decodeWellData(bytes memory data) public view virtual returns (uint256[] memory decimals) {	// @audit-issue missing `@param` tag

338:    function _calcRate(	// @audit-issue missing `@param` tag

357:    function getScaledReserves(	// @audit-issue missing `@param` tag

366:    function _calcReserve(	// @audit-issue missing `@param` tag

375:    function getBandC(	// @audit-issue missing `@param` tag

387:    function updateReserve(PriceData memory pd, uint256 reserve) internal pure returns (uint256) {	// @audit-issue missing `@param` tag
```
[60](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L60-L60), [74](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L74-L74), [114](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L114-L114), [153](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L153-L153), [173](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L173-L173), [246](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L246-L246), [310](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L310-L310), [338](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L338-L338), [357](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L357-L357), [366](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L366-L366), [375](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L375-L375), [387](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L387-L387), 


```solidity
Path: ./src/functions/StableLUT/Stable2LUT1.sol

27:    function getRatiosFromPriceLiquidity(uint256 price) external pure returns (PriceData memory) {	// @audit-issue missing `@param` tag

740:    function getRatiosFromPriceSwap(uint256 price) external pure returns (PriceData memory) {	// @audit-issue missing `@param` tag
```
[27](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L27-L27), [740](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L740-L740), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).

### NatSpec: Modifier declarations should have `@dev` tag
Some modifiers have an incomplete NatSpec: add a `@dev` notation to describe the modifier to improve the code documentation.

```solidity
Path: ./src/WellUpgradeable.sol

22:    modifier notDelegatedOrIsMinimalProxy() {	// @audit-issue missing `@dev` tag
```
[22](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L22-L22), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).

### NatSpec: Error declarations should have `@notice` tag
The `@notice` tag in NatSpec comments is used to provide important explanations to end users about what a error does. It appears that this contract's modifier declarations are missing `@notice` tags in their NatSpec annotations.

The absence of `@notice` tags reduces the contract's transparency and could lead to misunderstandings about a error's purpose and behavior.  [Dive Deeper into NatSpec Guidelines](https://docs.soliditylang.org/en/develop/natspec-format.html)

```solidity
Path: ./src/functions/Stable2.sol

50:    error InvalidTokenDecimals();	// @audit-issue missing `@notice` tag

51:    error InvalidLUT();	// @audit-issue missing `@notice` tag
```
[50](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L50-L50), [51](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L51-L51), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).

### NatSpec: Error declarations should have `@dev` tag
Some errors have an incomplete NatSpec: add a `@dev` notation to describe the error to improve the code documentation.

```solidity
Path: ./src/functions/Stable2.sol

50:    error InvalidTokenDecimals();	// @audit-issue missing `@dev` tag

51:    error InvalidLUT();	// @audit-issue missing `@dev` tag
```
[50](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L50-L50), [51](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L51-L51), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).
