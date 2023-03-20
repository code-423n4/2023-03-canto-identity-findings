## Summary

### Low-Risk Issues
| |Issue|Instances|
|-|:-|:-:|
| [L&#x2011;01] | `indexed` keyword for reference type variables such as `string` in events may lead to data loss. | 2 |


Total: 2 instances over 1 issue


### Non-critical Issues
| |Issue|Instances|
|-|:-|:-:|
| [N&#x2011;01] | Use a more recent version of solidity | 5 |
| [N&#x2011;02] | File is missing NatSpec | 1 |
| [N&#x2011;03] | Consider using ```delete``` rather than assigning zero to default values | 2 |

Total: 8 instances over 3 issues

Note: The table above was created considering the **automatic findings** and thus, those are not included.



## Low-Risk Issues

### [L&#x2011;01]  `indexed` keyword for reference type variables such as `string` in events may lead to data loss.
when the ```indexed``` keyword is used for reference typed variables such as string, it will return the hash of the mentioned string.
Thus, the event which is supposed to inform all of the applications subscribed to its emitting transaction (e.g. front-end of the DApp),
would get a meaningless and obscure 32 bytes that correspond to keccak256 of an encoded string. For more information about the
indexed events, one can check here(https://docs.soliditylang.org/en/v0.8.17/abi-spec.html?highlight=indexed#events).

*There are 2 instances of this issue:*

```solidity
    event NamespaceFused(address indexed fuser, uint256 indexed namespaceId, string indexed name);
```
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L54


```solidity
    event BioAdded(address indexed minter, uint256 indexed nftID, string indexed bio);
```
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L23



## Non-critical Issues

### [N&#x2011;01]  Use a more recent version of solidity.
Using version 0.8.17 for the solidity compiler is better.

*There are 5 instances of this issue:*

```solidity
    pragma solidity >=0.8.0;
```
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L2
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L2
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L2
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L2
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L2



### [N&#x2011;02]  File is missing NatSpec
Some functions miss NatSpec (@inheritdoc)

*There is 1 instance of this issue:*

```solidity
    function _drawing(uint256 _seed) private pure returns (TileData memory tileData) {...}
}
```
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L245



### [N&#x2011;03]  Consider using ```delete``` rather than assigning zero to default values

*There are 2 instances of this issue:*

```solidity
    nftContract = address(0);
  
```
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L102

```solidity
    nftID = 0;
  
```
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L103