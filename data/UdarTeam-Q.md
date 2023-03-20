### Issues Template
| Letter | Name | Description |
|:--:|:-------:|:-------:|
| L  | Low risk | Potential risk |
| NC |  Non-critical | Non risky findings |
| R  | Refactor | Changing the code |
| O | Ordinary | Often found issues |

| Total Found Issues | 3 |
|:--:|:--:|

### Low Risk Issues Template
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [L-01] | Use two-step transfer ownership | 1 |


| Total Low Risk Issues | 1 |
|:--:|:--:|


### Non-Critical Issues Template
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [N-01] | Create your own import names instead of using the regular ones | 9 |


| Total Non-Critical Issues | 1 |
|:--:|:--:|

### Ordinary Issues Template
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [O-01] | Locking pragma | 5 |


| Total Ordinary Issues | 1 |
|:--:|:--:|

### [L-01] Use two-step transfer ownership

```Namespace.sol``` contract inherits abstract ```Owned.sol``` contract which has a public ```transferOwnership``` method. There is a risk to transfer the ownership to an invalid address.

```solidity
canto-namespace-protocol/src/Namespace.sol

11:    contract Namespace is ERC721, Owned {
```

```solidity
canto-namespace-protocol/lib/solmate/src/auth/Owned.sol

39:    function transferOwnership(address newOwner) public virtual onlyOwner {
40:        owner = newOwner;
41:
42:        emit OwnershipTransferred(msg.sender, newOwner);
43:    }
```

Consider using a two-step process to transfer the ownership, e.g. [Ownable2Step](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol).

### [N-01] Create your own import names instead of using the regular ones

For better readability, you should name the imports instead of using the regular ones.

#### Instances

```solidity
canto-pfp-protocol/src/ProfilePicture.sol

5:    import "../interface/Turnstile.sol";
6:    import "../interface/ICidNFT.sol";
```

```solidity
canto-bio-protocol/src/Bio.sol

7:    import "../interface/Turnstile.sol";
```

```solidity
canto-namespace-protocol/src/Namespace.sol

7:    import "./Tray.sol";
8:    import "./Utils.sol";
9:    import "../interface/Turnstile.sol";
```

```solidity
canto-namespace-protocol/src/Tray.sol

10:    import "./Utils.sol";
11:    import "../interface/Turnstile.sol";
```

```solidity
canto-namespace-protocol/src/Utils.sol

4:    import "./Tray.sol";
```

### [O-01] Locking pragma

Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

#### Instances

- canto-pfp-protocol/src/ProfilePicture.sol
- canto-bio-protocol/src/Bio.sol
- canto-namespace-protocol/src/Namespace.sol
- canto-namespace-protocol/src/Tray.sol
- canto-namespace-protocol/src/Utils.sol
