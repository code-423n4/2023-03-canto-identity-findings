## 1. Cache bytes(_bio).length in variable

bytes(_bio).length is used 3 times within the mint function. Cache this variable and use that variable 3 times instead to save gas.

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L121

