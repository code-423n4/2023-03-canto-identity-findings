GasOptimization

### Gas Optimizations List
|     | Issue | Instances |
| --- | --- | --- |
| G-01 | Using ``delete`` instead of setting state variable/mapping to ``0`` saves gas| 2   |
| G-02 |   Using ``delete`` instead of setting ``address(0)`` saves gas| 1  |


### [G-01] Using ``delete`` instead of setting state variable/mapping to ``0`` saves gas.
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
### [G-02] Using ``delete`` instead of setting ``address(0)`` saves gas.
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L102
```solidity
 102:  nftContract = address(0);
```
can be written as shown below.
```solidity
 102: delete nftContract;
```