### Summary

### Low risk issues
| |Issue|Instances| |
|-|:-|:-:|:-:|
| [L&#x2011;01] | Constructor lacks address(0) checks for immutable variable | 7 |
| [L&#x2011;02] | Immutable tray price lacks zero value checks  | 1 |

### Non-critical issues
| |Issue|Instances| |
|-|:-|:-:|:-:|
| [N&#x2011;01] | Do not use floating solidity compiler version. Lock the solidity compiler version | 4 |
| [N&#x2011;02] | Use a more recent version of solidity | 4 |
| [N&#x2011;03] | For functions, follow Solidity standard naming conventions | 3 |

### Low risk issues

### [L&#x2011;01]   Constructor lacks address(0) checks for immutable variable
Zero-address check should be used to avoid in the constructors, to avoid the risk of setting a storage variable as address(0) at the time of deployment.
There are 7 instances of this issue.

1- In ProfilePicture.sol contract, cidNFT is immutable therefore any address error in constructor will result in redeployment of contract.

```solidity
File: canto-pfp-protocol/src/ProfilePicture.sol

14         ICidNFT private immutable cidNFT;
---

57    constructor(address _cidNFT, string memory _subprotocolName) ERC721("Profile Picture", "PFP") {
58        cidNFT = ICidNFT(_cidNFT);
59        subprotocolName = _subprotocolName;
60        if (block.chainid == 7700) {
```
[Link to code](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L57-L60)

### Recommended Mitigation steps
It is recommended to add address(0) check in constructor.

```solidity
File: canto-pfp-protocol/src/ProfilePicture.sol

         ICidNFT private immutable cidNFT;
---

    constructor(address _cidNFT, string memory _subprotocolName) ERC721("Profile Picture", "PFP") {
+       require(_cidNFT != address(0), "cidNFT can not be zero address");
        cidNFT = ICidNFT(_cidNFT);
        subprotocolName = _subprotocolName;
        if (block.chainid == 7700) {
```

2- In Namespace.sol contract, tray is immutable therefore any address error in constructor will result in redeployment of contract. It is also recommended to have address(0) checks for note and revenue address in constructor, though i am aware that note and revenue address can be updated via functions. 

```solidity
File: canto-namespace-protocol/src/Namespace.sol

17    Tray public immutable tray;
18
19    /// @notice Reference to the $NOTE TOKEN
20    ERC20 public note;
21
22    /// @notice Wallet that receives the revenue
23    address private revenueAddress;
---

73    constructor(
74        address _tray,
75        address _note,
76        address _revenueAddress
77    ) ERC721("Namespace", "NS") Owned(msg.sender) {
78        tray = Tray(_tray);
79        note = ERC20(_note);
80        revenueAddress = _revenueAddress;
81        if (block.chainid == 7700) {
```
[Link to code](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L73-L81)

### Recommended Mitigation steps
It is recommended to add address(0) check in constructor.

```solidity
File: canto-namespace-protocol/src/Namespace.sol

    Tray public immutable tray;

    /// @notice Reference to the $NOTE TOKEN
    ERC20 public note;

    /// @notice Wallet that receives the revenue
    address private revenueAddress;
---

    constructor(
        address _tray,
        address _note,
        address _revenueAddress
    ) ERC721("Namespace", "NS") Owned(msg.sender) {
+       require(_tray !=  address(0), "tray address can not be zero address");
+       require(_note !=  address(0), "note address can not be zero address");
+       require(_revenueAddress !=  address(0), "revenue address can not be zero address");
        tray = Tray(_tray);
        note = ERC20(_note);
        revenueAddress = _revenueAddress;
        if (block.chainid == 7700) {
```

3- In Tray.sol contract, namespaceNFT is immutable therefore any address error in constructor will result in redeployment of contract. It is also recommended to have address(0) checks for note and revenue address in constructor, though i am aware that note and revenue address can be updated via functions. 

```solidity
File: canto-namespace-protocol/src/Tray.sol

44    address private revenueAddress;
---

47    ERC20 public note;
---

50    address public immutable namespaceNFT;
---

98    constructor(
99        bytes32 _initHash,
100        uint256 _trayPrice,
101        address _revenueAddress,
102        address _note,
103        address _namespaceNFT
104    ) ERC721A("Namespace Tray", "NSTRAY") Owned(msg.sender) {
105        lastHash = _initHash;
106        trayPrice = _trayPrice;
107        revenueAddress = _revenueAddress;
```
[Link to code](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L98-L107)

### Recommended Mitigation steps
It is recommended to add address(0) check in constructor.

```solidity
File: canto-namespace-protocol/src/Tray.sol

    address private revenueAddress;
---

    ERC20 public note;
---

    address public immutable namespaceNFT;
---

    constructor(
        bytes32 _initHash,
        uint256 _trayPrice,
        address _revenueAddress,
        address _note,
        address _namespaceNFT
    ) ERC721A("Namespace Tray", "NSTRAY") Owned(msg.sender) {
+       require(_revenueAddress != address(0), "revenue address can not be zero address");
+       require(_note != address(0), "note address can not be zero address");
+       require(_namespaceNFT != address(0), "namespaceNFT address can not be zero address");
        lastHash = _initHash;
        trayPrice = _trayPrice;
        revenueAddress = _revenueAddress;
```


### [L&#x2011;02]   Immutable tray price lacks zero value checks
In Tray.sol contract, trayPrice is immutable therefore any value error in constructor will result in redeployment of contract. 

```solidity
File: canto-namespace-protocol/src/Tray.sol

37    uint256 public immutable trayPrice;
---

98    constructor(
99        bytes32 _initHash,
100        uint256 _trayPrice,
101        address _revenueAddress,
102        address _note,
103        address _namespaceNFT
104    ) ERC721A("Namespace Tray", "NSTRAY") Owned(msg.sender) {
105        lastHash = _initHash;
106        trayPrice = _trayPrice;
```
[Link to code](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L98-L106)

If the value of trayPrice is mistakenly set to 0, It will also affect in buy() amount calculations, whatever the amount the buyer will pay it will be equal to 0 and zero amount will be transferred to revenue address. 

```solidity
File: canto-namespace-protocol/src/Tray.sol

157            SafeTransferLib.safeTransferFrom(note, msg.sender, revenueAddress, _amount * trayPrice);
```
[Link to code](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L157)

### Recommended Mitigation steps
It is recommended to add zero value checks in constructor for immutable variables.

```solidity
File: canto-namespace-protocol/src/Tray.sol

    uint256 public immutable trayPrice;
---

    constructor(
        bytes32 _initHash,
        uint256 _trayPrice,
        address _revenueAddress,
        address _note,
        address _namespaceNFT
    ) ERC721A("Namespace Tray", "NSTRAY") Owned(msg.sender) {
+       require (trayPrice != 0, "trayPrice must be greater than 0");
        lastHash = _initHash;
        trayPrice = _trayPrice;
```

### Non-critical issues
### [N&#x2011;01]   Do not use floating solidity compiler version. Lock the solidity compiler version
Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.
There are 4 instances of this issue.

```solidity
File: canto-pfp-protocol/src/ProfilePicture.sol

2       pragma solidity >=0.8.0;
```
[Link to code](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L2)

```solidity
File: canto-bio-protocol/src/Bio.sol

2       pragma solidity >=0.8.0;
```
[Link to code](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L2)

```solidity
File: canto-namespace-protocol/src/Namespace.sol

2       pragma solidity >=0.8.0;
```
[Link to code](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L2)

```solidity
File: canto-namespace-protocol/src/Tray.sol

2       pragma solidity >=0.8.0;
```
[Link to code](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L2)

### [N&#x2011;02]    Use a more recent version of solidity
Recommend to use latest solidity compiler version(0.8.18 or 0.8.19). This will safeguard from underflow and overflow conditions. This modification will make the contracts more gas optimized. safeMath library wont be explicitely required as ^0.8.0 version has already taken  the underflow and overflow conditions by default.
1- Use a solidity version of at least 0.8 to get default underflow/overflow checks.
2- Use a solidity version of at least 0.8.2 to get simple compiler automatic inlining.
3- Use a solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads.
4- Use a solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than
revert()/require() strings.
5- Use a solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value.
There are 4 instances of this issue.

```solidity
File: canto-pfp-protocol/src/ProfilePicture.sol

2       pragma solidity >=0.8.0;
```
[Link to code](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L2)

```solidity
File: canto-bio-protocol/src/Bio.sol

2       pragma solidity >=0.8.0;
```
[Link to code](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L2)

```solidity
File: canto-namespace-protocol/src/Namespace.sol

2       pragma solidity >=0.8.0;
```
[Link to code](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L2)

```solidity
File: canto-namespace-protocol/src/Tray.sol

2       pragma solidity >=0.8.0;
```
[Link to code](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L2)

### [N&#x2011;03]    For functions, follow Solidity standard naming conventions
Proper use of _ as a function name prefix should be taken care and a common pattern is to prefix internal and private function names with _.
This _ pattern is applied partially in smart contracts, It should be followed in all smart contracts. 
There are 3 instances of this issue.

```solidity
File: canto-namespace-protocol/src/Utils.sol

73    function characterToUnicodeBytes(
74        uint8 _fontClass,
75        uint16 _characterIndex,
76        uint256 _characterModifier
77    ) internal pure returns (bytes memory) {
```
[Link to code](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L73-L77)

```solidity
File: canto-namespace-protocol/src/Utils.sol

222    function generateSVG(Tray.TileData[] memory _tiles, bool _isTray) internal pure returns (string memory) {
```
[Link to code](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L222)

```solidity
File: canto-namespace-protocol/src/Utils.sol

256    function iteratePRNG(uint256 _currState) internal pure returns (uint256 iteratedState) {
```
[Link to code](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L256)
