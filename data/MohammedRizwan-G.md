## Summary

### Gas Optimizations
| |Issue|Instances| |
|-|:-|:-:|:-:|
| [G&#x2011;01] | Avoid compound assignment operator in state variables | 1 |

### [G&#x2011;01]  Avoid compound assignment operator in state variables
Using compound assignment operators for state variables (like State += X or State -= X …) it’s more expensive than using operator assignment (like State = State + X or State = State - X …).
There are 1 instance of this issue.

```solidity
File: canto-namespace-protocol/src/Namespace.sol

150            numBytes += numBytesChar;
```
[Link to code](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L150)
