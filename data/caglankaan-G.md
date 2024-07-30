### Another Constant With Same Value Is Already Defined
Defining multiple constants with the same value in a Solidity contract introduces unnecessary redundancy and can lead to confusion and potential errors in contract maintenance. Constants are used to define values that remain unchanged throughout the contract's lifecycle, enhancing readability and efficiency. However, when multiple constants representing the same value are defined, it not only clutters the code but also complicates the process of updating values and understanding the contract's logic. Consolidating these constants into a single definition improves code clarity and maintainability.

```solidity
Path: ./src/WellUpgradeable.sol

100:     * Calls {_authorizeUpgrade}.	// @audit-issue: Same value `20` is already defined on line `38` as variable: `PACKED_ADDRESS`

101:     * @dev `upgradeTo` was modified to support ERC-1167 minimal proxies	// @audit-issue: Same value `52` is already defined on line `39` as variable: `ONE_WORD_PLUS_PACKED_ADDRESS`
```
[100](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L100-L100), [101](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L101-L101), 


```solidity
Path: ./src/functions/Stable2.sol

44:    uint256 constant PRICE_THRESHOLD = 100; // 0.01%	// @audit-issue: Same value `100` is already defined on line `37` as variable: `A_PRECISION`
```
[44](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L44-L44), 


#### Recommendation

Identify and consolidate duplicate constants in your Solidity contracts. Examine your contract's constants to find any with identical values and refactor your code to use a single constant definition for each unique value. This may involve choosing the most descriptive and appropriate name for the consolidated constant and updating all references accordingly. Such consolidation not only makes your contract more concise and easier to read but also simplifies future updates to constant values. Document the purpose and usage of each constant clearly, ensuring that the chosen names accurately reflect their role within the contract.

### `address(this)` should be cached
Cacheing saves gas when compared to repeating the calculation at each point it is used in the contract.

```solidity
Path: ./src/WellUpgradeable.sol

23:        if (address(this) != ___self) {	// @audit-issue: `adress(this)` also used on line(s): [25]
```
[23](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L23-L23), 


#### Recommendation

To enhance gas efficiency, cache the contract's address by storing `address(this)` in a state variable at the point of contract deployment or initialization. Use this cached address throughout the contract instead of repeatedly calling `address(this)`. This practice reduces the gas cost associated with multiple computations of the contract's address, leading to more efficient contract execution, especially in scenarios with frequent usage of the contract's address.

### Revert String Size Optimization
Shortening the revert strings to fit within 32 bytes will decrease deployment time and decrease runtime Gas when the revert condition is met.

Revert strings that are longer than 32 bytes require at least one additional `mstore`, along with additional overhead to calculate memory offset, etc.



```solidity
Path: ./src/WellUpgradeable.sol

26:            require(wellImplmentation == ___self, "Function must be called by a Well bored by an aquifer");	// @audit-issue: String length is `53`

67:        require(address(this) != ___self, "Function must be called through delegatecall");	// @audit-issue: String length is `44`

72:        require(activeProxy == ___self, "Function must be called through active proxy bored by an aquifer");	// @audit-issue: String length is `64`

75:        require(	// @audit-issue: String length is `47`
76:            IAquifer(aquifer).wellImplementation(newImplmentation) != address(0),
77:            "New implementation must be a well implmentation"
78:        );

81:        require(	// @audit-issue: String length is `57`
82:            UUPSUpgradeable(newImplmentation).proxiableUUID() == _IMPLEMENTATION_SLOT,
83:            "New implementation must be a valid ERC-1967 implmentation"
84:        );
```
[26](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L26-L26), [67](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L67-L67), [72](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L72-L72), [75](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L75-L78), [81](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L81-L84), 


#### Recommendation


To optimize Gas usage in your Solidity contract, it is recommended to keep revert strings as short as possible and to ensure that they fit within 32 bytes. It is possible to use abbreviations or simplified error messages to keep the string length short. Doing so can reduce the amount of Gas used during deployment and runtime when the revert condition is met.


### Custom Errors in Solidity for Gas Efficiency
Starting from Solidity version 0.8.4, the language introduced a feature known as "custom errors". These custom errors provide a way for developers to define more descriptive and semantically meaningful error conditions without relying on string messages. Prior to this version, developers often used the `require` statement with string error messages to handle specific conditions or validations. However, every unique string used as a revert reason consumes gas, making transactions more expensive.

Custom errors, on the other hand, are identified by their name and the types of their parameters only, and they do not have the overhead of string storage. This means that, when using custom errors instead of `require` statements with string messages, the gas consumption can be significantly reduced, leading to more gas-efficient contracts.

```solidity
Path: ./src/WellUpgradeable.sol

26:            require(wellImplmentation == ___self, "Function must be called by a Well bored by an aquifer");

28:            revert("UUPSUpgradeable: must not be called through delegatecall");

67:        require(address(this) != ___self, "Function must be called through delegatecall");	// @audit-issue

72:        require(activeProxy == ___self, "Function must be called through active proxy bored by an aquifer");	// @audit-issue

75:        require(	// @audit-issue
76:            IAquifer(aquifer).wellImplementation(newImplmentation) != address(0),
77:            "New implementation must be a well implmentation"
78:        );

81:        require(	// @audit-issue
82:            UUPSUpgradeable(newImplmentation).proxiableUUID() == _IMPLEMENTATION_SLOT,
83:            "New implementation must be a valid ERC-1967 implmentation"
84:        );
```
[23](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L26-L26), [23](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L28-L28), [67](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L67-L67), [72](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L72-L72), [75](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L75-L78), [81](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L81-L84), 


```solidity
Path: ./src/functions/Stable2.sol

143:        revert("did not find convergence");	// @audit-issue
```
[143](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L143-L143), 


```solidity
Path: ./src/functions/StableLUT/Stable2LUT1.sol

35:                                    revert("LUT: Invalid price");

727:                                revert("LUT: Invalid price");

748:                                    revert("LUT: Invalid price");

2167:                                revert("LUT: Invalid price");
```
[28](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L35-L35), [28](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L727-L727), [741](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L748-L748), [741](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L2167-L2167), 


#### Recommendation


It is recommended to use custom errors instead of revert strings to reduce gas costs, especially during contract deployment. Custom errors can be defined using the error keyword and can include dynamic information.

### Avoid repeating computations
In Solidity development, repeating the same computations within a contract can lead to unnecessary gas consumption and reduce the contract's efficiency. This is particularly relevant in functions that are called frequently or involve complex calculations. Repeating computations not only wastes computational resources but also increases the cost of executing transactions. By identifying and eliminating redundant calculations, developers can optimize contract performance, reduce gas costs, and improve overall execution speed.

```solidity
Path: ./src/functions/Stable2.sol

135:                    return reserve / (10 ** (18 - decimals[j]));	// @audit-issue: Same binary operation statement in line(s) between: ['139:139']

231:                    return scaledReserves[j] / (10 ** (18 - decimals[j]));	// @audit-issue: Same binary operation statement in line(s) between: ['235:235']

296:                    return scaledReserves[j] / (10 ** (18 - decimals[j]));	// @audit-issue: Same binary operation statement in line(s) between: ['300:300']

392:                - pd.maxStepSize * (pd.targetPrice - pd.currentPrice) / (pd.lutData.highPrice - pd.lutData.lowPrice);	// @audit-issue: Same binary operation statement in line(s) between: ['397:397']
```
[135](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L135-L135), [231](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L231-L231), [296](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L296-L296), [392](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L392-L392), 


#### Recommendation

Review your Solidity contracts to identify any computations that are performed multiple times with the same inputs. Cache the results of these computations in local variables and reuse them within the function or across function calls if the state remains unchanged.

### Using Prefix Operators Costs Less Gas Than Postfix Operators in Loops
Conditions can be optimized issues in Solidity refer to situations where smart contract developers write conditional statements that can be simplified or optimized for better gas efficiency, readability, and maintainability. Optimizing conditions can lead to more cost-effective and secure smart contracts.

```solidity
Path: ./src/functions/Stable2.sol

88:        for (uint256 i = 0; i < 255; i++) {	// @audit-issue

220:        for (uint256 k; k < 255; k++) {	// @audit-issue

288:        for (uint256 k; k < 255; k++) {	// @audit-issue
```
[88](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L88-L88), [220](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L220-L220), [288](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L288-L288), 


#### Recommendation

To improve gas efficiency in your Solidity code, prefer using prefix operators (e.g., `++i` or `--i`) instead of postfix operators (e.g., `i++` or `i--`) within loops. Prefix operators typically result in lower gas costs and can contribute to more efficient contract execution.

### Increments can be `unchecked` in for-loops
Newer versions of the Solidity compiler will check for integer overflows and underflows automatically. This provides safety but increases gas costs.
When an unsigned integer is guaranteed to never overflow, the unchecked feature of Solidity can be used to save gas costs.A common case for this is for-loops using a strictly-less-than comparision in their conditional statement.

```solidity
Path: ./src/WellUpgradeable.sol

42:        for (uint256 i; i < tokensLength - 1; ++i) {	// @audit-issue

43:            for (uint256 j = i + 1; j < tokensLength; ++j) {	// @audit-issue
```
[42](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L42-L42), [43](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L43-L43), 


```solidity
Path: ./src/functions/Stable2.sol

88:        for (uint256 i = 0; i < 255; i++) {	// @audit-issue

128:        for (uint256 i; i < 255; ++i) {	// @audit-issue

220:        for (uint256 k; k < 255; k++) {	// @audit-issue

288:        for (uint256 k; k < 255; k++) {	// @audit-issue
```
[88](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L88-L88), [128](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L128-L128), [220](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L220-L220), [288](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L288-L288), 


#### Recommendation

Use unchecked math to block overflow / underflow check to save Gas.

### Divisions can be unchecked to save gas
The expression type(int).min/(-1) is the only case where division causes an overflow. Therefore, uncheck can be used to save gas in scenarios where it is certain that such an overflow will not occur.

```solidity
Path: ./src/functions/Stable2.sol

91:            dP = dP * lpTokenSupply / (scaledReserves[0] * N);	// @audit-issue

92:            dP = dP * lpTokenSupply / (scaledReserves[1] * N);	// @audit-issue

94:            lpTokenSupply = (Ann * sumReserves / A_PRECISION + (dP * N)) * lpTokenSupply	// @audit-issue

95:                / (((Ann - A_PRECISION) * lpTokenSupply / A_PRECISION) + ((N + 1) * dP));	// @audit-issue

135:                    return reserve / (10 ** (18 - decimals[j]));	// @audit-issue

139:                    return reserve / (10 ** (18 - decimals[j]));	// @audit-issue

187:        pd.targetPrice = scaledRatios[i] * PRICE_PRECISION / scaledRatios[j];	// @audit-issue

196:        uint256 parityReserve = lpTokenSupply / 2;	// @audit-issue

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
[91](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L91-L91), [92](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L92-L92), [94](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L94-L94), [95](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L95-L95), [135](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L135-L135), [139](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L139-L139), [187](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L187-L187), [196](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L196-L196), [201](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L201-L201), [202](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L202-L202), [207](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L207-L207), [208](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L208-L208), [215](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L215-L215), [217](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L217-L217), [231](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L231-L231), [235](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L235-L235), [260](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L260-L260), [269](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L269-L269), [275](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L275-L275), [283](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L283-L283), [285](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L285-L285), [296](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L296-L296), [300](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L300-L300), [372](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L372-L372), [380](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L380-L380), [381](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L381-L381), [392](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L392-L392), [397](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L397-L397), 


#### Recommendation

Utilize 'unchecked' blocks in Solidity for divisions where overflow is impossible, such as when 'type(int).min/(-1)' is not a concern. This can save gas by bypassing overflow checks in these specific cases.

### Add `unchecked {}` for subtractions where the operands cannot underflow because of a previous `require()` or `if`-statement
Unchecked keyword can be added to such scenerios: 
`require(a <= b); x = b - a` => `require(a <= b); unchecked { x = b - a }`

```solidity
Path: ./src/functions/Stable2.sol

98:                if (lpTokenSupply - prevReserves <= 1) return lpTokenSupply;	// @audit-issue

100:                if (prevReserves - lpTokenSupply <= 1) return lpTokenSupply;	// @audit-issue

134:                if (reserve - prevReserve <= 1) {	// @audit-issue

138:                if (prevReserve - reserve <= 1) {	// @audit-issue

215:            pd.maxStepSize = scaledReserves[j] * (pd.lutData.lowPriceJ - pd.lutData.highPriceJ) / pd.lutData.lowPriceJ;	// @audit-issue

217:            pd.maxStepSize = scaledReserves[j] * (pd.lutData.highPriceJ - pd.lutData.lowPriceJ) / pd.lutData.highPriceJ;	// @audit-issue

230:                if (pd.currentPrice - pd.targetPrice <= PRICE_THRESHOLD) {	// @audit-issue

234:                if (pd.targetPrice - pd.currentPrice <= PRICE_THRESHOLD) {	// @audit-issue

283:            pd.maxStepSize = scaledReserves[j] * (pd.lutData.lowPriceJ - pd.lutData.highPriceJ) / pd.lutData.lowPriceJ;	// @audit-issue

285:            pd.maxStepSize = scaledReserves[j] * (pd.lutData.highPriceJ - pd.lutData.lowPriceJ) / pd.lutData.highPriceJ;	// @audit-issue

295:                if (pd.currentPrice - pd.targetPrice <= PRICE_THRESHOLD) {	// @audit-issue

299:                if (pd.targetPrice - pd.currentPrice <= PRICE_THRESHOLD) {	// @audit-issue

392:                - pd.maxStepSize * (pd.targetPrice - pd.currentPrice) / (pd.lutData.highPrice - pd.lutData.lowPrice);	// @audit-issue

397:                + pd.maxStepSize * (pd.currentPrice - pd.targetPrice) / (pd.lutData.highPrice - pd.lutData.lowPrice);	// @audit-issue
```
[98](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L98-L98), [100](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L100-L100), [134](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L134-L134), [138](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L138-L138), [215](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L215-L215), [217](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L217-L217), [230](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L230-L230), [234](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L234-L234), [283](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L283-L283), [285](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L285-L285), [295](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L295-L295), [299](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L299-L299), [392](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L392-L392), [397](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L397-L397), 


#### Recommendation

In scenarios where subtraction cannot result in underflow due to prior `require()` or `if`-statements, wrap these operations in an `unchecked` block to save gas. This optimization should only be applied when the safety of the operation is assured. Carefully analyze each case to confirm that underflow is impossible before implementing `unchecked` blocks, as incorrect usage can lead to vulnerabilities in the contract.

### Stack variable is only used once
If the variable is only accessed once, it's cheaper to use the assigned value directly that one time, and save the 3 gas the extra stack assignment would spend

```solidity
Path: ./src/WellUpgradeable.sol

25:            address wellImplmentation = IAquifer(aquifer).wellImplementation(address(this));	// @audit-issue: wellImplmentation used only on line: 26

71:        address activeProxy = IAquifer(aquifer).wellImplementation(_getImplementation());	// @audit-issue: activeProxy used only on line: 72
```
[25](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L25-L25), [71](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L71-L71), 


```solidity
Path: ./src/functions/Stable2.sol

79:        uint256[] memory decimals = decodeWellData(data);	// @audit-issue: decimals used only on line: 81

159:        uint256[] memory decimals = decodeWellData(data);	// @audit-issue: decimals used only on line: 160

163:        uint256 lpTokenSupply = calcLpTokenSupply(scaledReserves, abi.encode(18, 18));	// @audit-issue: lpTokenSupply used only on line: 165
```
[79](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L79-L79), [159](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L159-L159), [163](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L163-L163), 


#### Recommendation

Eliminate single-use stack variables in Solidity to optimize gas consumption. Directly use the assigned value in the place of the variable. This approach saves the 3 gas typically used for the extra stack assignment, streamlining the function's execution and enhancing overall gas efficiency.

### `Private` functions only called once can be inlined to save gas
If a private function is only used once, there is no need to modularize it, unless the function calling it would otherwise be too long and complex. Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

```solidity
Path: ./src/functions/Stable2.sol

366:    function _calcReserve(	// @audit-issue

375:    function getBandC(	// @audit-issue
```
[366](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L366-L366), [375](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L375-L375), 


#### Recommendation

Inline 'private' functions in Solidity that are called only once to save gas. This avoids the additional gas cost of 20 to 40 units associated with extra JUMP instructions and stack operations required for separate function calls, unless the calling function becomes too complex.

### Inline modifiers used only once
In Solidity, modifiers provide a way to add reusable conditions or logic to functions. However, if a modifier is used only once, the modularization may not be beneficial in terms of gas efficiency. The overhead associated with a modifier – including two extra JUMP instructions and additional stack operations for the function call – results in an extra cost of about 20 to 40 gas. In cases where a modifier is unique to a single function and the function’s complexity does not necessitate breaking out logic, inlining the modifier's code directly within the function can be more gas-efficient. This approach avoids the overhead of a separate modifier call while maintaining code clarity.

```solidity
Path: ./src/WellUpgradeable.sol

22:    modifier notDelegatedOrIsMinimalProxy() {	// @audit-issue
```
[22](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L22-L22), 


#### Recommendation

Evaluate the use of modifiers in your Solidity contracts, particularly those applied to only one function. If a modifier is unique to a single function, consider inlining its logic within the function itself to save gas. This is especially advantageous for simpler functions where the additional clarity provided by a separate modifier does not outweigh the gas cost of its use. Ensure that the function remains clear and maintainable after inlining the modifier's logic, and document the rationale for inlining to maintain code readability and facilitate future maintenance.

### Optimizing Arithmetic with Shift Operators for Division and Multiplication by Powers of Two
In computational operations, especially within contexts where efficiency matters, certain arithmetic operations can be optimized. One such optimization is leveraging shift operators for division and multiplication by powers of two. This stems from the binary nature of numbers in computing, where shifting bits to the left (using `<<`) effectively multiplies a number by 2 for each position shifted, and shifting bits to the right (using `>>`) divides the number by 2 for each position shifted.

In Solidity, and many other programming languages, using shift operators can be more gas-efficient than traditional arithmetic operations for these specific cases. For instance, instead of performing `value * 4`, one can use `value << 2`, and instead of `value / 4`, one can use `value >> 2`. These optimizations can result in reduced gas costs and faster execution times.


```solidity
Path: ./src/functions/Stable2.sol

196:        uint256 parityReserve = lpTokenSupply / 2;	// @audit-issue

372:        return (reserve * reserve + c) / (reserve * 2 + b - lpTokenSupply);	// @audit-issue
```
[196](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L196-L196), [372](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L372-L372), 


#### Recommendation

When performing division or multiplication by powers of two in Solidity, consider using shift operators (`<<` for multiplication and `>>` for division) for improved gas efficiency. This optimization takes advantage of the binary nature of computing and can lead to reduced gas costs and faster execution times.

### Consider pre-calculating the address of `address(this)` to save gas
Use `foundry`'s [`script.sol`](https://book.getfoundry.sh/reference/forge-std/compute-create-address) or `solady`'s [`LibRlp.sol`](https://github.com/Vectorized/solady/blob/main/src/utils/LibRLP.sol) to save the value in a constant, which will avoid having to spend gas to push the value on the stack every time it's used.

```solidity
Path: ./src/WellUpgradeable.sol

23:        if (address(this) != ___self) {	// @audit-issue

25:            address wellImplmentation = IAquifer(aquifer).wellImplementation(address(this));	// @audit-issue

67:        require(address(this) != ___self, "Function must be called through delegatecall");	// @audit-issue
```
[23](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L23-L23), [25](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L25-L25), [67](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L67-L67), 


#### Recommendation

To enhance gas efficiency, cache the contract's address by storing `address(this)` in a state variable at the point of contract deployment or initialization. Use this cached address throughout the contract instead of repeatedly calling `address(this)`. This practice reduces the gas cost associated with multiple computations of the contract's address, leading to more efficient contract execution, especially in scenarios with frequent usage of the contract's address.

### Counting down in for statements is more gas efficient
Looping downwards in Solidity is more gas efficient due to how the EVM compares variables. In a 'for' loop that counts down, the end condition is usually to compare with zero, which is cheaper than comparing with another number. As such, restructure your loops to count downwards where possible.

```solidity
Path: ./src/WellUpgradeable.sol

42:        for (uint256 i; i < tokensLength - 1; ++i) {	// @audit-issue

43:            for (uint256 j = i + 1; j < tokensLength; ++j) {	// @audit-issue
```
[42](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L42-L42), [43](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L43-L43), 


```solidity
Path: ./src/functions/Stable2.sol

88:        for (uint256 i = 0; i < 255; i++) {	// @audit-issue

128:        for (uint256 i; i < 255; ++i) {	// @audit-issue

220:        for (uint256 k; k < 255; k++) {	// @audit-issue

288:        for (uint256 k; k < 255; k++) {	// @audit-issue
```
[88](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L88-L88), [128](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L128-L128), [220](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L220-L220), [288](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L288-L288), 


#### Recommendation

Where feasible, refactor `for` loops in your Solidity contracts to count downwards. Adjust the loop initialization, condition, and iteration statements to decrement the loop variable and terminate the loop when it reaches zero. This approach can lead to gas savings, making your contract more efficient in terms of execution costs. Ensure that this refactoring aligns with the logic and requirements of your contract, and thoroughly test to confirm that the revised loop behavior matches the intended functionality.

### Consider using solady's 'FixedPointMathLib'
Using Solady's "FixedPointMathLib" for multiplication or division operations in Solidity can lead to significant gas savings. This library is designed to optimize fixed-point arithmetic operations, which are common in financial calculations involving tokens or currencies. By implementing more efficient algorithms and assembly optimizations, "FixedPointMathLib" minimizes the computational resources required for these operations. For contracts that frequently perform such calculations, integrating this library can reduce transaction costs, thereby enhancing overall performance and cost-effectiveness. However, developers must ensure compatibility with their existing codebase and thoroughly test for accuracy and expected behavior to avoid any unintended consequences.

```solidity
Path: ./src/functions/Stable2.sol

83:        uint256 Ann = a * N * N;	// @audit-issue

91:            dP = dP * lpTokenSupply / (scaledReserves[0] * N);	// @audit-issue

92:            dP = dP * lpTokenSupply / (scaledReserves[1] * N);	// @audit-issue

94:            lpTokenSupply = (Ann * sumReserves / A_PRECISION + (dP * N)) * lpTokenSupply	// @audit-issue

95:                / (((Ann - A_PRECISION) * lpTokenSupply / A_PRECISION) + ((N + 1) * dP));	// @audit-issue

124:        (uint256 c, uint256 b) = getBandC(a * N * N, lpTokenSupply, j == 0 ? scaledReserves[1] : scaledReserves[0]);	// @audit-issue

135:                    return reserve / (10 ** (18 - decimals[j]));	// @audit-issue

139:                    return reserve / (10 ** (18 - decimals[j]));	// @audit-issue

187:        pd.targetPrice = scaledRatios[i] * PRICE_PRECISION / scaledRatios[j];	// @audit-issue

196:        uint256 parityReserve = lpTokenSupply / 2;	// @audit-issue

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

362:        scaledReserves[0] = reserves[0] * 10 ** (18 - decimals[0]);	// @audit-issue

363:        scaledReserves[1] = reserves[1] * 10 ** (18 - decimals[1]);	// @audit-issue

372:        return (reserve * reserve + c) / (reserve * 2 + b - lpTokenSupply);	// @audit-issue

380:        c = lpTokenSupply * lpTokenSupply / (reserves * N) * lpTokenSupply * A_PRECISION / (Ann * N);	// @audit-issue

381:        b = reserves + (lpTokenSupply * A_PRECISION / Ann);	// @audit-issue

392:                - pd.maxStepSize * (pd.targetPrice - pd.currentPrice) / (pd.lutData.highPrice - pd.lutData.lowPrice);	// @audit-issue

397:                + pd.maxStepSize * (pd.currentPrice - pd.targetPrice) / (pd.lutData.highPrice - pd.lutData.lowPrice);	// @audit-issue
```
[83](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L83-L83), [91](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L91-L91), [92](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L92-L92), [94](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L94-L94), [95](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L95-L95), [124](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L124-L124), [135](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L135-L135), [139](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L139-L139), [187](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L187-L187), [196](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L196-L196), [201](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L201-L201), [202](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L202-L202), [207](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L207-L207), [208](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L208-L208), [215](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L215-L215), [217](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L217-L217), [231](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L231-L231), [235](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L235-L235), [260](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L260-L260), [269](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L269-L269), [275](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L275-L275), [283](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L283-L283), [285](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L285-L285), [296](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L296-L296), [300](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L300-L300), [362](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L362-L362), [363](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L363-L363), [372](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L372-L372), [380](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L380-L380), [381](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L381-L381), [392](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L392-L392), [397](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L397-L397), 


#### Recommendation

Consider integrating Solady's 'FixedPointMathLib' into your Solidity contracts for optimized fixed-point arithmetic operations. This library can provide substantial gas savings and enhance the performance of your contract. Before integration, evaluate how 'FixedPointMathLib' aligns with your contract’s requirements. Ensure thorough testing for accuracy and compatibility with your existing contract logic. Carefully document any changes and keep track of how these optimizations affect your contract's operations to maintain transparency and reliability in your application. Adopting 'FixedPointMathLib' should be a considered decision, balancing the benefits of gas efficiency with the need for maintaining code clarity and functionality.

### Refactor modifiers to call a local function
Modifiers code is copied in all instances where it's used, increasing bytecode size. If deployment gas costs are a concern for this contract, refactoring modifiers into functions can reduce bytecode size significantly at the cost of one JUMP.

```solidity
Path: ./src/WellUpgradeable.sol

22:    modifier notDelegatedOrIsMinimalProxy() {	// @audit-issue
```
[22](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L22-L22), 


#### Recommendation

Evaluate your contract's use of modifiers, particularly those applied across multiple functions, to identify candidates for refactoring into functions. Convert these modifiers into external or public functions that perform the same checks or actions. Then, replace the modifier usage in function declarations with calls to these functions at the start of your functions

### Use `do while` loops intead of for loops
A `do while` loop will cost less gas since the condition is not being checked for the first iteration.
```solidity
uint256 i = 1;
do {                   
    param2 += i;
    i++;
}
while (i < 50);
``` 
is better than
```solidity
for(uint256 i = 1; i< 50; i++){
    param1 += i;
}
```


```solidity
Path: ./src/WellUpgradeable.sol

42:        for (uint256 i; i < tokensLength - 1; ++i) {	// @audit-issue

43:            for (uint256 j = i + 1; j < tokensLength; ++j) {	// @audit-issue
```
[42](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L42-L42), [43](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L43-L43), 


```solidity
Path: ./src/functions/Stable2.sol

88:        for (uint256 i = 0; i < 255; i++) {	// @audit-issue

128:        for (uint256 i; i < 255; ++i) {	// @audit-issue

220:        for (uint256 k; k < 255; k++) {	// @audit-issue

288:        for (uint256 k; k < 255; k++) {	// @audit-issue
```
[88](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L88-L88), [128](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L128-L128), [220](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L220-L220), [288](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L288-L288), 


#### Recommendation

Where appropriate, consider using a `do while` loop instead of a `for` loop in your Solidity contracts. This is especially beneficial when the first iteration of the loop does not require a condition check. Refactor your loop logic to fit the `do while` structure for more gas-efficient execution. However, ensure that the loop's logic and termination conditions are correctly implemented to avoid infinite loops or other logical errors. Always balance gas efficiency with code readability and the specific requirements of your contract's logic.

### Using XOR (^) and AND (&) bitwise equivalents for gas optimizations
Given 4 variables a, b, c and d represented as such:
```
0 0 0 0 0 1 1 0 <- a
0 1 1 0 0 1 1 0 <- b
0 0 0 0 0 0 0 0 <- c
1 1 1 1 1 1 1 1 <- d
```
To have a == b means that every 0 and 1 match on both variables. Meaning that a XOR (operator ^) would evaluate to 0 ((a ^ b) == 0), as it excludes by definition any equalities.Now, if a != b, this means that there’s at least somewhere a 1 and a 0 not matching between a and b, making (a ^ b) != 0.Both formulas are logically equivalent and using the XOR bitwise operator costs actually the same amount of gas.However, it is much cheaper to use the bitwise OR operator (|) than comparing the truthy or falsy values.These are logically equivalent too, as the OR bitwise operator (|) would result in a 1 somewhere if any value is not 0 between the XOR (^) statements, meaning if any XOR (^) statement verifies that its arguments are different.


```solidity
Path: ./src/WellUpgradeable.sol

26:            require(wellImplmentation == ___self, "Function must be called by a Well bored by an aquifer");	// @audit-issue

44:                if (_tokens[i] == _tokens[j]) {	// @audit-issue

72:        require(activeProxy == ___self, "Function must be called through active proxy bored by an aquifer");	// @audit-issue

82:            UUPSUpgradeable(newImplmentation).proxiableUUID() == _IMPLEMENTATION_SLOT,	// @audit-issue
```
[26](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L26-L26), [44](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L44-L44), [72](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L72-L72), [82](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L82-L82), 


```solidity
Path: ./src/functions/Stable2.sol

61:        if (lut == address(0)) revert InvalidLUT();	// @audit-issue

78:        if (reserves[0] == 0 && reserves[1] == 0) return 0;	// @audit-issue

86:        if (sumReserves == 0) return 0;	// @audit-issue

124:        (uint256 c, uint256 b) = getBandC(a * N * N, lpTokenSupply, j == 0 ? scaledReserves[1] : scaledReserves[0]);	// @audit-issue

179:        uint256 i = j == 1 ? 0 : 1;	// @audit-issue

252:        uint256 i = j == 1 ? 0 : 1;	// @audit-issue

314:        if (decimal0 == 0) {	// @audit-issue

317:        if (decimal0 == 0) {	// @audit-issue
```
[61](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L61-L61), [78](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L78-L78), [86](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L86-L86), [124](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L124-L124), [179](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L179-L179), [252](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L252-L252), [314](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L314-L314), [317](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L317-L317), 


#### Recommendation

Review your Solidity contracts to identify any computations that are performed multiple times with the same inputs. Cache the results of these computations in local variables and reuse them within the function or across function calls if the state remains unchanged.

### The result of a function call should be cached rather than re-calling the function
The function calls in solidity are expensive. If the same result of the same function calls are to be used several times, the result should be cached to reduce the gas consumption of repeated calls.        

```solidity
Path: ./src/WellUpgradeable.sol

71:        address activeProxy = IAquifer(aquifer).wellImplementation(_getImplementation());	// @audit-issue: Function call `IAquifer` is called multiple times at lines [76].
```
[71](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L71-L71), 


```solidity
Path: ./src/functions/Stable2.sol

193:        uint256 lpTokenSupply = calcLpTokenSupply(scaledReserves, abi.encode(18, 18));	// @audit-issue: Function call `encode` is called multiple times at lines [224].
```
[193](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L193-L193), 


```solidity
Path: ./src/functions/StableLUT/Stable2LUT1.sol

35:                                    revert("LUT: Invalid price");	// @audit-issue: Function call `revert` is called multiple times at lines [727].

748:                                    revert("LUT: Invalid price");	// @audit-issue: Function call `revert` is called multiple times at lines [2167].
```
[35](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L35-L35), [748](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/StableLUT/Stable2LUT1.sol#L748-L748), 


#### Recommendation

Cache the result of function calls in Solidity instead of making repeated calls to the same function. This practice significantly reduces gas consumption by minimizing costly function call operations.

### Expression `bytes('')` is cheaper than `new bytes(0)`
In Solidity, initializing empty byte arrays can be done using either `bytes('')` or `new bytes(0)`. Contrary to a common belief, `bytes('')` is actually more gas-efficient than `new bytes(0)`. The expression `bytes('')` directly initializes an empty byte array, while `new bytes(0)` creates a new dynamic array and initializes its length to zero, which involves slightly more computation and consequently, a higher gas cost. This difference, though minimal, can be significant in contracts where such initializations are frequent, particularly in scenarios requiring high gas efficiency.

```solidity
Path: ./src/WellUpgradeable.sol

95:        _upgradeToAndCallUUPS(newImplementation, new bytes(0), false);	// @audit-issue
```
[95](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L95-L95), 


#### Recommendation

Use `new bytes(0)` instead of `bytes("")` when initializing an empty byte array in Solidity. This approach is more gas-efficient as it avoids the overhead associated with string processing and conversion. Review your contracts to identify and replace instances where `bytes("")` is used for this purpose, applying this simple yet effective optimization to enhance contract performance and reduce gas costs.

### Empty Blocks Should Be Removed Or Emit Something
The code should be refactored such that empty blocks no longer exist, or the block should do something useful, such as emitting an event or reverting. Empty blocks can introduce confusion and potential errors when the code is modified in the future.


```solidity
Path: ./src/WellUpgradeable.sol

54:    function initNoWellToken() external initializer {}	// @audit-issue
```
[54](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L54-L54), 


#### Recommendation

Remove empty blocks in Solidity code to avoid confusion and potential errors. If a block is necessary, ensure it performs a useful action, like emitting an event or reverting, to justify its presence and clarify its purpose.

### `abi.encode()` is less efficient than `abi.encodePacked()`
When working with Solidity, it is important to pay attention to gas efficiency in order to optimize smart contracts. In this blog post, we will compare the gas costs of `abi.encode()` and `abi.encodepacked()` and explore why the latter is more efficient.Source: [reference](https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison)


```solidity
Path: ./src/functions/Stable2.sol

163:        uint256 lpTokenSupply = calcLpTokenSupply(scaledReserves, abi.encode(18, 18));	// @audit-issue

193:        uint256 lpTokenSupply = calcLpTokenSupply(scaledReserves, abi.encode(18, 18));	// @audit-issue

224:            scaledReserves[i] = calcReserve(scaledReserves, i, lpTokenSupply, abi.encode(18, 18));	// @audit-issue

291:            pd.currentPrice = calcRate(scaledReserves, i, j, abi.encode(18, 18));	// @audit-issue

350:        rate = _reserves[i] - calcReserve(_reserves, i, lpTokenSupply, abi.encode(18, 18));	// @audit-issue
```
[163](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L163-L163), [193](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L193-L193), [224](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L224-L224), [291](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L291-L291), [350](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L350-L350), 


#### Recommendation

Optimize gas usage by preferring 'abi.encodePacked()' over 'abi.encode()' where appropriate, as 'abi.encodePacked()' is generally more gas-efficient. However, ensure its suitability based on the data type and structure requirements, as 'abi.encodePacked()' can lead to ambiguity in certain cases.

### Use calldata instead of memory for function arguments that do not get mutated
Mark data types as `calldata` instead of `memory` where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as `calldata`. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies `memory` storage.

```solidity
Path: ./src/WellUpgradeable.sol

33:    function init(string memory _name, string memory _symbol) external override reinitializer(2) {	// @audit-issue

104:    function upgradeToAndCall(address newImplementation, bytes memory data) public payable override {	// @audit-issue
```
[33](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L33-L33), [104](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L104-L104), 


```solidity
Path: ./src/functions/Stable2.sol

74:    function calcLpTokenSupply(
75:        uint256[] memory reserves,	// @audit-issue
76:        bytes memory data
77:    ) public view returns (uint256 lpTokenSupply) {

74:    function calcLpTokenSupply(
75:        uint256[] memory reserves,
76:        bytes memory data	// @audit-issue
77:    ) public view returns (uint256 lpTokenSupply) {

114:    function calcReserve(
115:        uint256[] memory reserves,	// @audit-issue
116:        uint256 j,
117:        uint256 lpTokenSupply,
118:        bytes memory data
119:    ) public view returns (uint256 reserve) {

114:    function calcReserve(
115:        uint256[] memory reserves,
116:        uint256 j,
117:        uint256 lpTokenSupply,
118:        bytes memory data	// @audit-issue
119:    ) public view returns (uint256 reserve) {

153:    function calcRate(
154:        uint256[] memory reserves,	// @audit-issue
155:        uint256 i,
156:        uint256 j,
157:        bytes memory data
158:    ) public view returns (uint256 rate) {

153:    function calcRate(
154:        uint256[] memory reserves,
155:        uint256 i,
156:        uint256 j,
157:        bytes memory data	// @audit-issue
158:    ) public view returns (uint256 rate) {

173:    function calcReserveAtRatioSwap(
174:        uint256[] memory reserves,	// @audit-issue
175:        uint256 j,
176:        uint256[] memory ratios,
177:        bytes calldata data
178:    ) external view returns (uint256 reserve) {

173:    function calcReserveAtRatioSwap(
174:        uint256[] memory reserves,
175:        uint256 j,
176:        uint256[] memory ratios,	// @audit-issue
177:        bytes calldata data
178:    ) external view returns (uint256 reserve) {

310:    function decodeWellData(bytes memory data) public view virtual returns (uint256[] memory decimals) {	// @audit-issue

338:    function _calcRate(
339:        uint256[] memory reserves,	// @audit-issue
340:        uint256 i,
341:        uint256 j,
342:        uint256 lpTokenSupply
343:    ) internal view returns (uint256 rate) {

357:    function getScaledReserves(
358:        uint256[] memory reserves,	// @audit-issue
359:        uint256[] memory decimals
360:    ) internal pure returns (uint256[] memory scaledReserves) {

357:    function getScaledReserves(
358:        uint256[] memory reserves,
359:        uint256[] memory decimals	// @audit-issue
360:    ) internal pure returns (uint256[] memory scaledReserves) {

387:    function updateReserve(PriceData memory pd, uint256 reserve) internal pure returns (uint256) {	// @audit-issue
```
[75](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L74-L77), [76](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L74-L77), [115](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L114-L119), [118](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L114-L119), [154](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L153-L158), [157](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L153-L158), [174](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L173-L178), [176](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L173-L178), [310](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L310-L310), [339](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L338-L343), [358](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L357-L360), [359](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L357-L360), [387](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L387-L387), 


#### Recommendation

To optimize gas usage in your Solidity functions, mark data types as `calldata` instead of `memory` wherever applicable. This prevents unnecessary data loading into memory. Use `calldata` for function arguments that do not require changes within the function, except when passing them into another function that explicitly requires `memory` storage.

### Nesting `if` statements that uses `&&` saves gas
In Solidity, the way conditional checks are structured can impact the gas consumption of a transaction. When conditions are combined using `&&` within an `if` statement, Solidity short-circuits the evaluation, meaning that if the first condition is `false`, the subsequent conditions won't be evaluated. This behavior can lead to gas savings compared to using separate nested `if` statements because not all conditions might need to be checked every time. By efficiently structuring these conditional checks, contracts can optimize the gas required for execution, leading to reduced costs for users.


```solidity
Path: ./src/functions/Stable2.sol

78:        if (reserves[0] == 0 && reserves[1] == 0) return 0;	// @audit-issue
```
[78](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L78-L78), 


#### Recommendation


When multiple conditions need to be checked successively, try to combine them in a single `if` statement using `&&` instead of nesting separate `if` statements. This will leverage short-circuit evaluation for potential gas savings.


### Operator `>=`/`<=` costs less gas than operator `>`/`<`
The compiler uses opcodes `GT` and `ISZERO` for code that uses `>`, but only requires `LT` for `>=`. A similar behaviour applies for `>`, which uses opcodes `LT` and `ISZERO`, but only requires `GT` for `<=`.


```solidity
Path: ./src/functions/Stable2.sol

320:        if (decimal0 > 18 || decimal1 > 18) revert InvalidTokenDecimals();	// @audit-issue
```
[320](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L320-L320), 


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

Opt for '>=', '<=' operators over '>' and '<' in Solidity where logically appropriate, as they consume less gas. This is because '>=' and '<=' require only one opcode ('LT' or 'GT'), compared to the two opcodes ('GT'/'LT' and 'ISZERO') needed for '>' and '<'.

### Constructor Can Be Marked As Payable
`payable` functions cost less gas to execute, since the compiler does not have to add extra checks to ensure that a payment wasn't provided.

A `constructor` can safely be marked as `payable`, since only the deployer would be able to pass funds, and the project itself would not pass any funds.



```solidity
Path: ./src/functions/Stable2.sol

60:    constructor(address lut) {	// @audit-issue
```
[60](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L60-L60), 


#### Recommendation

Mark constructors as 'payable' in Solidity contracts to reduce gas costs, as this eliminates the need for the compiler to add checks against incoming payments. This is safe because only the deployer can send funds during contract creation, and typically no funds are sent at this stage.

### Initializers Can Be Marked As Payable
`payable` functions cost less gas to execute, since the compiler does not have to add extra checks to ensure that a payment wasn't provided.

A `initializer` can safely be marked as `payable`, since only the deployer would be able to pass funds, and the project itself would not pass any funds.



```solidity
Path: ./src/WellUpgradeable.sol

33:    function init(string memory _name, string memory _symbol) external override reinitializer(2) {	// @audit-issue
```
[33](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L33-L33), 


#### Recommendation

Mark initializers as 'payable' in Solidity contracts to reduce gas costs, as this eliminates the need for the compiler to add checks against incoming payments. This is safe because only the deployer can send funds during contract creation, and typically no funds are sent at this stage.

### Optimize names to save gas
`public`/`external` function names and `public` member variable names can be optimized to save gas. Below are the interfaces/abstract contracts that can be optimized so that the most frequently-called functions use the least amount of gas possible during method lookup. Method IDs that have two leading zero bytes can save 128 gas each during deployment, and renaming functions to have lower method IDs will save 22 gas per call, [per sorted position shifted](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92).


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

Optimize gas usage by renaming 'public'/'external' functions and 'public' member variables in Solidity. Aim for shorter and more efficient names, especially for frequently called functions. This can save gas during deployment and reduce gas costs per call due to lower method ID sorting positions.

### Consider activating `via-ir` for deploying
The IR-based code generator was developed to make code generation more performant by enabling optimization passes that can be applied across functions.

It is possible to activate the IR-based code generator through the command line by using the flag `--via-ir`or by including the option `{"viaIR": true}`.

Keep in mind that compiling with this option may take longer. However, you can simply test it before deploying your code. If you find that it provides better performance, you can add the `--via-ir` flag to your deploy command.
```solidity
/// @audit Global finding.
```
[1](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L1), 


#### Recommendation

Consider activating `via-ir`.

### Optimize Deployment Size by Fine-tuning IPFS Hash
The Solidity compiler appends 53 bytes of metadata to the smart contract code, incurring an extra cost of 10,600 gas. This additional expense arises from 200 gas per bytecode, plus calldata cost, which amounts to 16 gas for non-zero bytes and 4 gas for zero bytes. This results in a maximum of 848 extra gas in calldata cost.

Reducing this cost is crucial for the following reasons:

The metadata's 53-byte addition leads to a deployment cost increase of 10,600 gas. It can also result in an additional calldata cost of up to 848 gas. Ways to Minimize Gas Consumption:

Employ the `--no-cbor-metadata` compiler option to exclude metadata. Be cautious as this might impact contract verification. Search for code comments that yield an IPFS hash with more zeros, thereby reducing calldata costs.

```solidity
/// @audit Global finding.
```
[1](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L1), 


#### Recommendation

To optimize deployment size and reduce associated costs, consider the following strategies:
1. **Exclude Metadata with Compiler Option**: Use the `solc` compiler’s `--metadata-hash none` or `--no-cbor-metadata` option to prevent the inclusion of metadata in the compiled bytecode. This action reduces the bytecode size, thus lowering deployment gas costs. However, exercise caution with this approach, as it might affect the ability to verify the contract on platforms like Etherscan.

2. **Optimize IPFS Hash for More Zeros**: If excluding metadata is not desirable, another approach involves optimizing code comments or elements that influence the metadata hash generation to achieve an IPFS hash with a higher proportion of zeros. Since calldata costs are lower for zero bytes, a metadata hash with more zeros can reduce the calldata costs associated with contract interactions.

Example for excluding metadata:
```bash
solc --metadata-hash none YourContract.sol
```


### Assembly: Use scratch space for building calldata
If an external call's calldata can fit into two or fewer words, use the scratch space to build the calldata, rather than allowing Solidity to do a memory expansion.

```solidity
Path: ./src/WellUpgradeable.sol

25:            address wellImplmentation = IAquifer(aquifer).wellImplementation(address(this));	// @audit-issue

71:        address activeProxy = IAquifer(aquifer).wellImplementation(_getImplementation());	// @audit-issue

76:            IAquifer(aquifer).wellImplementation(newImplmentation) != address(0),	// @audit-issue

82:            UUPSUpgradeable(newImplmentation).proxiableUUID() == _IMPLEMENTATION_SLOT,	// @audit-issue
```
[25](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L25-L25), [71](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L71-L71), [76](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L76-L76), [82](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L82-L82), 


```solidity
Path: ./src/functions/Stable2.sol

63:        a = ILookupTable(lut).getAParameter();	// @audit-issue

190:        pd.lutData = ILookupTable(lookupTable).getRatiosFromPriceSwap(pd.targetPrice);	// @audit-issue

263:        pd.lutData = ILookupTable(lookupTable).getRatiosFromPriceLiquidity(pd.targetPrice);	// @audit-issue
```
[63](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L63-L63), [190](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L190-L190), [263](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L263-L263), 


#### Recommendation

Review your smart contracts to identify and remove `block.number` and `block.timestamp` from the parameters of emitted events. Instead of manually adding these fields, rely on the Ethereum blockchain's inherent inclusion of this information within the block context. Simplify your event definitions to include only the essential data specific to the event's purpose, excluding universally available block metadata.

### Use assembly to check for `address(0)`
In Solidity, it's a common practice to check whether an Ethereum address variable is set to the zero address (`address(0)`) to handle various scenarios, such as token transfers or contract interactions. Typically, this check is performed using a conditional statement like `if (addressVariable == address(0))`.

However, using this approach in high-frequency or gas-sensitive operations can lead to unnecessary gas costs. A more gas-efficient alternative is to use inline assembly to perform the zero address check, which can significantly reduce gas consumption, especially in loops or complex contract logic.

By utilizing inline assembly for this specific check, you can optimize gas usage and make your Solidity code more efficient.

```solidity
Path: ./src/WellUpgradeable.sol

76:            IAquifer(aquifer).wellImplementation(newImplmentation) != address(0),	// @audit-issue
```
[76](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/WellUpgradeable.sol#L76-L76), 


```solidity
Path: ./src/functions/Stable2.sol

61:        if (lut == address(0)) revert InvalidLUT();	// @audit-issue
```
[61](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L61-L61), 


#### Recommendation

To optimize gas usage in your Solidity code, consider using inline assembly for checking `address(0)`. This approach can significantly reduce gas costs, especially in high-frequency or gas-sensitive operations, leading to more efficient contract execution.

### Use assembly to check for `0`
Using assembly to check for zero can save gas by allowing more direct access to the evm and reducing some of the overhead associated with high-level operations in solidity.

```solidity
Path: ./src/functions/Stable2.sol

78:        if (reserves[0] == 0 && reserves[1] == 0) return 0;	// @audit-issue

86:        if (sumReserves == 0) return 0;	// @audit-issue

124:        (uint256 c, uint256 b) = getBandC(a * N * N, lpTokenSupply, j == 0 ? scaledReserves[1] : scaledReserves[0]);	// @audit-issue

314:        if (decimal0 == 0) {	// @audit-issue

317:        if (decimal0 == 0) {	// @audit-issue
```
[78](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L78-L78), [86](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L86-L86), [124](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L124-L124), [314](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L314-L314), [317](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L317-L317), 


#### Recommendation

To optimize gas usage in your Solidity code, consider using inline assembly for checking `0`. This approach can significantly reduce gas costs, especially in high-frequency or gas-sensitive operations, leading to more efficient contract execution.

### Use assembly to write `address` storage values
Using assembly `{ sstore(state.slot, addr)}` instead of `state = addr` can save gas.


```solidity
Path: ./src/functions/Stable2.sol

62:        lookupTable = lut;	// @audit-issue
```
[62](https://github.com/code-423n4/2024-07-basin/blob/7d5aacbb144d0ba0bc358dfde6e0cc913d25310e/./src/functions/Stable2.sol#L62-L62), 


#### Recommendation

To reduce gas costs in your Solidity code, consider using assembly with `{ sstore(state.slot, addr) }` for writing `address` storage values instead of `state = addr`. This approach can result in significant gas savings.