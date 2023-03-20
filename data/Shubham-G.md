# Report


## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | Use nested if and, avoid multiple check combinations | 7 |
| [GAS-2](#GAS-2) | Setting the constructor to `payable` | 4 |
| [GAS-3](#GAS-3) | Use constants instead of type(uintx).max | 6 |
| [GAS-4](#GAS-4) | Use assembly to write address storage values | 3 |

## [GAS-1] Use nested if and, avoid multiple check combinations
Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

Before:
```solidity
File: canto-bio-protocol/src/Bio.sol

60:        if ((i > 0 && (i + 1) % 40 == 0) || prevByteWasContinuation || i == lengthInBytes - 1) {

```
```solidity
File: canto-namespace-protocol/src/Namespace.sol

136:         if (tileData.fontClass != 0 && _characterList[i].skinToneModifier != 0) {

```

```solidity
File: canto-namespace-protocol/src/Namespace.sol

157:               if (
                    trayOwner != msg.sender &&
                    tray.getApproved(trayID) != msg.sender &&
                    !tray.isApprovedForAll(trayOwner, msg.sender)
                   ) 

```
```solidity
File: canto-namespace-protocol/src/Namespace.sol

186:               if (nftOwner != msg.sender && getApproved[_id] != msg.sender && !isApprovedForAll[nftOwner] 
                       [msg.sender])

```
```solidity
File: canto-namespace-protocol/src/Tray.sol

175:                 if (
                         namespaceNFT != msg.sender &&
                         trayOwner != msg.sender &&
                         getApproved(_id) != msg.sender &&
                         !isApprovedForAll(trayOwner, msg.sender)
                        )

```
```solidity
File: canto-namespace-protocol/src/Tray.sol

184:                 if (numPrelaunchMinted != type(uint256).max && _id <= numPrelaunchMinted)

```
```solidity
File: canto-namespace-protocol/src/Tray.sol

240:                 if (startTokenId <= numPrelaunchMinted && to != address(0))

```

After:
**Saves approx 1800 gas**
```solidity
File: canto-bio-protocol/src/Bio.sol
60: 	     if (i > 0) {
                if ((i + 1) % 40 == 0 || prevByteWasContinuation || i == lengthInBytes - 1) {
```
**Saves approx 1910 gas**
```solidity
File: canto-namespace-protocol/src/Namespace.sol

136:         if (tileData.fontClass != 0) {
                if (_characterList[i].skinToneModifier != 0) {

```
**Saves approx 209 gas**

```solidity
File: canto-namespace-protocol/src/Namespace.sol

157:          if (trayOwner != msg.sender) {
                  if (tray.getApproved(trayID) != msg.sender) {
                      if (!tray.isApprovedForAll(trayOwner, msg.sender))

```
**Saves approx 3400 gas**
```solidity
File: canto-namespace-protocol/src/Namespace.sol

186:      if (nftOwner != msg.sender) {
              if (getApproved[_id] != msg.sender) {
                 if (!isApprovedForAll[nftOwner][msg.sender])

```
#### Using nested if in Tray.sol saves approx 22,000 (22k) gas
```solidity
File: canto-namespace-protocol/src/Tray.sol

175:      if (namespaceNFT != msg.sender) {
              if (trayOwner != msg.sender) {
                  if (getApproved(_id) != msg.sender) {
                      if (!isApprovedForAll(trayOwner, msg.sender)

```
```solidity
File: canto-namespace-protocol/src/Tray.sol

184:          if (numPrelaunchMinted != type(uint256).max) {
                 if (_id <= numPrelaunchMinted)

```
```solidity
File: canto-namespace-protocol/src/Tray.sol

240:           if (startTokenId <= numPrelaunchMinted) {
                  if (to != address(0))

```

## [GAS-2] Setting the constructor to `payable`
You can cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor payable. Making the constructor payable eliminates the need for an initial check of msg.value == 0 and saves upto **15 gas in each contract** on deployment with no security risks.

Before:
```solidity
File: canto-bio-protocol/src/Bio.sol

32:     constructor() ERC721("Biography", "Bio") {

```

```solidity
File: canto-namespace-protocol/src/Namespace.sol

73:     constructor(
        address _tray,
        address _note,
        address _revenueAddress
        ) ERC721("Namespace", "NS") Owned(msg.sender) {

```

```solidity
File: canto-namespace-protocol/src/Tray.sol

98:     constructor(
        bytes32 _initHash,
        uint256 _trayPrice,
        address _revenueAddress,
        address _note,
        address _namespaceNFT
        ) ERC721A("Namespace Tray", "NSTRAY") Owned(msg.sender) {

```

```solidity
File: canto-pfp-protocol/src/ProfilePicture.sol

57:     constructor(address _cidNFT, string memory _subprotocolName) ERC721("Profile Picture", "PFP") {
```

After:
```solidity
File: canto-bio-protocol/src/Bio.sol

32:     constructor() payable ERC721("Biography", "Bio") {

```

```solidity
File: canto-namespace-protocol/src/Namespace.sol

73:     constructor(
        address _tray,
        address _note,
        address _revenueAddress
        ) payable ERC721("Namespace", "NS") Owned(msg.sender) {

```

```solidity
File: canto-namespace-protocol/src/Tray.sol

98:     constructor(
        bytes32 _initHash,
        uint256 _trayPrice,
        address _revenueAddress,
        address _note,
        address _namespaceNFT
        ) payable ERC721A("Namespace Tray", "NSTRAY") Owned(msg.sender) {

```

```solidity
File: canto-pfp-protocol/src/ProfilePicture.sol

57:     constructor(address _cidNFT, string memory _subprotocolName) payable ERC721("Profile Picture", "PFP") {
```

## [GAS-3] Use constants instead of type(uintx).max
type(uint120).max or type(uint112).max, etc. it uses more gas in the distribution process and also for each transaction than constant usage.

```solidity
File: canto-namespace-protocol/src/Tray.sol

73:      uint256 private prelaunchMinted = type(uint256).max;

122:     if (numPrelaunchMinted != type(uint256).max) {

152:     if (prelaunchMinted == type(uint256).max) {

184:     if (numPrelaunchMinted != type(uint256).max && _id <= numPrelaunchMinted)

226:     if (prelaunchMinted != type(uint256).max) revert PrelaunchAlreadyEnded();

238:     if (numPrelaunchMinted != type(uint256).max) {
```

## [GAS-4] Use assembly to write address storage values

Before:
```solidity
File: canto-namespace-protocol/src/Namespace.sol

80:     revenueAddress = _revenueAddress;

```

```solidity
File: canto-namespace-protocol/src/Tray.sol

105:     lastHash = _initHash;

107:     revenueAddress = _revenueAddress;
```

After: 
**Saves approx 9 gas.**
```solidity
File: canto-namespace-protocol/src/Namespace.sol

80:     assembly {
            sstore(revenueAddress.slot, _revenueAddress)
        }

```

**Saves approx 27 gas.**
```solidity
File: canto-namespace-protocol/src/Tray.sol

105:     assembly {
            sstore(lastHash.slot, _initHash)
            sstore(revenueAddress.slot, _revenueAddress)
        }
```

