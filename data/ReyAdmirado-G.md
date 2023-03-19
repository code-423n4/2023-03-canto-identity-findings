| | issue |
| ----------- | ----------- |
| 1 | [Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate](#1-multiple-addressid-mappings-can-be-combined-into-a-single-mapping-of-an-addressid-to-a-struct-where-appropriate) |
| 2 | [can make the variable outside the loop to save gas](#2-can-make-the-variable-outside-the-loop-to-save-gas) |
| 3 | [ Ternary over if ... else](#3-ternary-over-if--else) |
| 4 | [public functions not called by the contract should be declared external instead](#4-public-functions-not-called-by-the-contract-should-be-declared-external-instead) |
| 5 | [Functions guaranteed to revert when called by normal users can be marked `payable`](#5-functions-guaranteed-to-revert-when-called-by-normal-users-can-be-marked-payable) |
| 6 | [Optimize names to save gas](#6-optimize-names-to-save-gas) |
| 7 | [Non-strict inequalities are cheaper than strict ones](#7-non-strict-inequalities-are-cheaper-than-strict-ones) |
| 8 | [Setting the constructor to payable](#8-setting-the-constructor-to-payable) |
| 9 | [can sort the conditions from cheapest to the most expensive](#9-can-sort-the-conditions-from-cheapest-to-the-most-expensive) |



## 1. Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate

Reads and subsequent writes can be cheaper when a function requires both values and they both fit in the same storage slot. can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations. 

`tokenToName` and `nftCharacters` are accessed once together
- [Namespace.sol#L46-L49](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L46-L49)


## 2. can make the variable outside the loop to save gas

make the variable outside and only give the value to variable inside

- [Bio.sol#L57](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L57)
- [Bio.sol#L61](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L61)

- [Namespace.sol#L123](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L123)
- [Namespace.sol#L124](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L124)
- [Namespace.sol#L125](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L125)
- [Namespace.sol#L127](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L127)
- [Namespace.sol#L133](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L133)
- [Namespace.sol#L134](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L134)
- [Namespace.sol#L144](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L144)
- [Namespace.sol#L146](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L146)
- [Namespace.sol#L147](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L147)
- [Namespace.sol#L156](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L156)

- [Tray.sol#L132](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L132)
- [Tray.sol#L160](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L160)
- [Tray.sol#L161](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L161)

- [Utils.sol#L148](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L148)
- [Utils.sol#L157](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L157)
- [Utils.sol#L166](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L166)


## 3. Ternary over if ... else

Using ternary operator instead of the if else statement saves gas.

- [Tray.sol#L152-L158](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L152-L158)


## 4. public functions not called by the contract should be declared external instead

Contracts are allowed to override their parents’ functions and change the visibility from external to public and can save gas by doing so. 

- [ProfilePicture.sol#L70](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L70)

- [Bio.sol#L43](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L43)

- [Namespace.sol#L90](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L90)

- [Tray.sol#L119](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L119)


## 5. Functions guaranteed to revert when called by normal users can be marked `payable`

If a function modifier or require such as onlyOwner-admin is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

- [Namespace.sol#L196](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L196)
- [Namespace.sol#L204](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L204)

- [Tray.sol#L210](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L210)
- [Tray.sol#L218](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L218)
- [Tray.sol#L225](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L225)


## 6. Optimize names to save gas

Contracts most called functions could simply save gas by function ordering via Method ID. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because 22 gas are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions.

See more [here](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92).
you can use this [tool](https://emn178.github.io/solidity-optimize-name/) to get the optimized version for function and properties signitures 


## 7. Non-strict inequalities are cheaper than strict ones

In the EVM, there is no opcode for non-strict inequalities (>=, <=) and two operations are performed (> + = or < + =).
consider replacing >= with the strict counterpart > and add `- 1` to the second side

- [Bio.sol#L77](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L77)
- [Bio.sol#L78](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L78)
- [Bio.sol#L79](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L79)

- [Tray.sol#L124](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L124)
- [Tray.sol#L184](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L184)
- [Tray.sol#L240](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L240)

- [Utils.sol#L86](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L86)
- [Utils.sol#L90](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L90)


## 8. Setting the constructor to payable

You can cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor payable. Making the constructor payable eliminates the need for an initial check of msg.value == 0 and saves 13 gas on deployment with no security risks.

- [ProfilePicture.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol)
- [Bio.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol)
- [Namespace.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol)
- [Tray.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol)


## 9. can sort the conditions from cheapest to the most expensive 

this will give the possibility that the cheaper condition succeeds and there is no need to do extra operations

```solidity
if (i > 0 && (i + 1) % 40 == 0 || prevByteWasContinuation || i == lengthInBytes - 1)
```
should be like this instead
```solidity
if (prevByteWasContinuation || i == lengthInBytes - 1 || (i > 0 && (i + 1) % 40 == 0))
```
- [Bio.sol#L60](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L)
