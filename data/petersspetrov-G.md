GasOptimization

### Gas Optimizations List
|     | Issue | Instances |
| --- | --- | --- |
| G-01 | Use `x != 0` instead of `x > 0` for `uint types`. The `!=` operator costs less gas than `>` and for uint types you can use it to check for non-zero values to save gas(saves 22 gas) | 1   |
| G-02 | `++i`/`i++` should be `unchecked{++i}`/`unchecked{i++}` when it is not possible for them to overflow, as is the case when used in `for`- and `while`-loops| 14  |
| G-03 | `++i` costs less gas compared to `i++` or `i += 1` | 1   |
| G-04 | Using ``delete`` instead of setting state variable/mapping to ``0`` saves gas | 2 |
| G-05 | Using ``delete`` instead of setting ``address(0)`` saves gas| 1 |

### [G-01\] Use `!= 0` instead of `> 0`.
Using `> 0` costs more gas than `!= 0` when used on a uint in a require() statement.
```solidity
60: if ((i > 0 && (i + 1) % 40 == 0) || prevByteWasContinuation || i == lengthInBytes - 1) {
```
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L60

### [G-02] `++i`/`i++` should be `unchecked{++i}`/`unchecked{i++}` when it is not possible for them to overflow, as is the case when used in `for`- and `while`-loops
The majority of Solidity for loops increment a uint256 variable that starts at 0.
These increment operations never need to be checked for over/underflow because the variable will never reach the max number of uint256
(will run out of gas long before that happens). The default over/underflow check wastes gas in every iteration of virtually every for loop . eg.

e.g Let’s work with a sample loop below.

```solidity
for(uint256 i; i < 10; i++){
//doSomething
}
```

can be written as shown below.

```solidity
for(uint256 i; i < 10;) {
  // loop logic
  unchecked { i++; }
}
```

https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L129
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L159
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L161
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L109
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L146
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L155
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L164
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L225
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L122
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L127https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L147
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L174
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L56
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L100

### [G-03] `++i` costs less gas compared to `i++` or `i += 1`.
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L59
```solidity
59:	 bytesOffset++;
```
can be written as shown below.
```solidity
59:  ++bytesOffset;
```
Pre-increments and pre-decrements are cheaper.
Increment:
`i += 1` is the most expensive form
`i++` costs 6 gas less than`i += 1`
`++i` costs 5 gas less than `i++` (11 gas less than i += 1)
Note that post-increments (or post-decrements) return the old value before incrementing or decrementing, hence the name post-increment:

### [G-04] Using ``delete`` instead of setting state variable/mapping to ``0`` saves gas.
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L93
```solidity
 93: bytesOffset = 0;
```
can be written as shown below.
```solidity
 93: delete bytesOffset;
```
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L103
```solidity
 103: nftID = 0;
```
can be written as shown below.
```solidity
 103: delete nftID;
```
### [G-05] Using ``delete`` instead of setting ``address(0)`` saves gas.
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L102
```solidity
 102:  nftContract = address(0);
```
can be written as shown below.
```solidity
 102: delete nftContract;
```