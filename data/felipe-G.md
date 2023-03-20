 # Report


## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | Use the `payable` modifier on constructor | 4 |
| [GAS-2](#GAS-2) | Use `immutable` on state variables updated only in the constructor | 1 |

### [GAS-1] Use the `payable` modifier on constructor 
Adding the `payable` modifier on constructor will simplify the initialization bytecode by removing the 'revert if msg.sender is not zero' check, resulting in cheaper contract deployment.

*Saves 204 gas per instance*

*Instances (4)*:
```solidity
File: canto-pfp-protocol/src/ProfilePicture.sol

57:    constructor(address _cidNFT, string memory _subprotocolName) ERC721("Profile Picture", "PFP") {

```

```solidity
File: canto-bio-protocol/src/Bio.sol

32:    constructor() ERC721("Biography", "Bio") {

```

```solidity
File: canto-namespace-protocol/src/Namespace.sol

73:    constructor(
74:        address _tray,
75:        address _note,
76:        address _revenueAddress
77:    ) ERC721("Namespace", "NS") Owned(msg.sender) {

```

```solidity
File: canto-namespace-protocol/src/Tray.sol

 98:    constructor(
 99:        bytes32 _initHash,
100:        uint256 _trayPrice,
101:        address _revenueAddress,
102:        address _note,
103:        address _namespaceNFT
104:    ) ERC721A("Namespace Tray", "NSTRAY") Owned(msg.sender) {

```

### [GAS-2] Use `immutable` on state variables updated only on the constructor 
`Immutable` variables are compiled into the contract's runtime bytecode, being interpreted by the EVM the same way as `constant`s. This change removes the need for the `SLOAD` instruction when reading the variable and will save 2100 gas when variable is cold and 100 when it's warm.

```solidity
File: canto-pfp-protocol/src/ProfilePicture.sol

 35:    string public subprotocolName;

```