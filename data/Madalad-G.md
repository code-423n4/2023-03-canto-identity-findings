# Gas Optimizations Summary
| |Issue|Instances|
|:-:|:-|:-:|
|[G-01]|`abi.encodePacked` is more gas efficient than `abi.encode`|1|
|[G-02]|Functions guaranteed to revert when called by normal users can be marked `payable`|5|
|[G-03]|Use assembly to calculate hashes|1|
|[G-04]|Division/Multiplication by 2 should use bit shifting|5|
|[G-05]|Use `constant`, `immutable` where applicable|13|
|[G-06]|Use `unchecked` for operations that cannot overflow/underflow|16|
|[G-07]|Change `public` functions to `external`|3|
|[G-08]|Use a more recent version of Solidity|4|
|[G-09]|Use named return values|8|

Total issues: 9

Total instances: 56

&nbsp;
# Gas Optimizations
## [G-01] `abi.encodePacked` is more gas efficient than `abi.encode`

`abi.encode` pads all elementary types to 32 bytes, whereas `abi.encodePacked` will only use the minimal required memory to encode the data. See [here](https://docs.soliditylang.org/en/v0.8.11/abi-spec.html?highlight=encodepacked#non-standard-packed-mode) for more info.

Instances: 1
```solidity
File: canto-namespace-protocol/src/Tray.sol

162:                 lastHash = keccak256(abi.encode(lastHash));

```
- [canto-namespace-protocol/src/Tray.sol#L162](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L162)


&nbsp;
## [G-02] Functions guaranteed to revert when called by normal users can be marked `payable`

If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost (2400 per instance).

Instances: 5
```solidity
File: canto-namespace-protocol/src/Namespace.sol

196:     function changeNoteAddress(address _newNoteAddress) external onlyOwner {

204:     function changeRevenueAddress(address _newRevenueAddress) external onlyOwner {

```
- [canto-namespace-protocol/src/Namespace.sol#L196](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L196)
- [canto-namespace-protocol/src/Namespace.sol#L204](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L204)

```solidity
File: canto-namespace-protocol/src/Tray.sol

210:     function changeNoteAddress(address _newNoteAddress) external onlyOwner {

218:     function changeRevenueAddress(address _newRevenueAddress) external onlyOwner {

225:     function endPrelaunchPhase() external onlyOwner {

```
- [canto-namespace-protocol/src/Tray.sol#L210](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L210)
- [canto-namespace-protocol/src/Tray.sol#L218](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L218)
- [canto-namespace-protocol/src/Tray.sol#L225](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L225)


&nbsp;
## [G-03] Use assembly to calculate hashes

Saves 5000 deployment gas per instance and 80 runtime gas per instance.

### Unoptimized
```solidity
function solidityHash(uint256 a, uint256 b) public view {
	//unoptimized
	keccak256(abi.encodePacked(a, b));
}
```

### Optimized
```solidity
function assemblyHash(uint256 a, uint256 b) public view {
	//optimized
	assembly {
		mstore(0x00, a)
		mstore(0x20, b)
		let hashedVal := keccak256(0x00, 0x40)
	}
}
```

Instances: 1
```solidity
File: canto-namespace-protocol/src/Tray.sol

162:                 lastHash = keccak256(abi.encode(lastHash));

```
- [canto-namespace-protocol/src/Tray.sol#L162](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L162)


&nbsp;
## [G-04] Division/Multiplication by 2 should use bit shifting

`<x> * 2` is equivalent to `<x> << 1` and `<x> / 2` is the same as `<x> >> 1`. The `MUL` and `DIV` opcodes cost 5 gas, whereas `SHL` and `SHR` only cost 3 gas.

Instances: 5
```solidity
File: canto-namespace-protocol/src/Tray.sol

263:                 tileData.fontClass = 7 + uint8((res - 104) / 2);

```
- [canto-namespace-protocol/src/Tray.sol#L263](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L263)

```solidity
File: canto-namespace-protocol/src/Utils.sol

148:                 uint256 characterIndex = (_characterModifier % ZALGO_NUM_ABOVE) * 2;

157:                 uint256 characterIndex = (_characterModifier % ZALGO_NUM_OVER) * 2;

166:                 uint256 characterIndex = (_characterModifier % ZALGO_NUM_BELOW) * 2;

201:                     uint256 offset = (_characterIndex - 2) * 2;

```
- [canto-namespace-protocol/src/Utils.sol#L148](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L148)
- [canto-namespace-protocol/src/Utils.sol#L157](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L157)
- [canto-namespace-protocol/src/Utils.sol#L166](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L166)
- [canto-namespace-protocol/src/Utils.sol#L201](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L201)


&nbsp;
## [G-05] Use `constant`, `immutable` where applicable

Use immutable if you want to assign a permanent value at construction. Use constants if you already know the permanent value. Both get directly embedded in bytecode, saving SLOAD. Variables only set in the constructor and never edited afterwards should be marked as immutable, as it would avoid the expensive storage-writing operation in the constructor (around 20 000 gas per variable) and replace the expensive storage-reading operations (around 2100 gas per reading) to a less expensive value reading (3 gas).

Instances: 13
```solidity
File: canto-pfp-protocol/src/ProfilePicture.sol

35:     string public subprotocolName;

41:         address indexed minter,

42:         uint256 indexed pfpNftID,

43:         address indexed referencedContract,

44:         uint256 referencedNftId

```
- [canto-pfp-protocol/src/ProfilePicture.sol#L35](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L35)
- [canto-pfp-protocol/src/ProfilePicture.sol#L41](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L41)
- [canto-pfp-protocol/src/ProfilePicture.sol#L42](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L42)
- [canto-pfp-protocol/src/ProfilePicture.sol#L43](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L43)
- [canto-pfp-protocol/src/ProfilePicture.sol#L44](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L44)

```solidity
File: canto-namespace-protocol/src/Namespace.sol

74:         address _tray,

75:         address _note,

76:         address _revenueAddress

```
- [canto-namespace-protocol/src/Namespace.sol#L74](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L74)
- [canto-namespace-protocol/src/Namespace.sol#L75](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L75)
- [canto-namespace-protocol/src/Namespace.sol#L76](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L76)

```solidity
File: canto-namespace-protocol/src/Tray.sol

99:         bytes32 _initHash,

100:         uint256 _trayPrice,

101:         address _revenueAddress,

102:         address _note,

103:         address _namespaceNFT

```
- [canto-namespace-protocol/src/Tray.sol#L99](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L99)
- [canto-namespace-protocol/src/Tray.sol#L100](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L100)
- [canto-namespace-protocol/src/Tray.sol#L101](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L101)
- [canto-namespace-protocol/src/Tray.sol#L102](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L102)
- [canto-namespace-protocol/src/Tray.sol#L103](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L103)


&nbsp;
## [G-06] Use `unchecked` for operations that cannot overflow/underflow

By bypassing Solidity's built in overflow/underflow checks using `unchecked`, we can save gas. This is especially beneficial for the index variable within for loops (saves 120 gas per iteration).

Instances: 16
```solidity
File: canto-bio-protocol/src/Bio.sol

56:         for (uint i; i < lengthInBytes; ++i) {

59:             bytesOffset++;

100:         for (uint i; i < lines; ++i) {

```
- [canto-bio-protocol/src/Bio.sol#L56](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L56)
- [canto-bio-protocol/src/Bio.sol#L59](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L59)
- [canto-bio-protocol/src/Bio.sol#L100](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L100)

```solidity
File: canto-namespace-protocol/src/Namespace.sol

122:         for (uint256 i; i < numCharacters; ++i) {

127:             for (uint256 j = i + 1; j < numCharacters; ++j) {

147:             for (uint256 j; j < numBytesChar; ++j) {

150:             numBytes += numBytesChar;

174:         for (uint256 i; i < numUniqueTrays; ++i) {

```
- [canto-namespace-protocol/src/Namespace.sol#L122](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L122)
- [canto-namespace-protocol/src/Namespace.sol#L127](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L127)
- [canto-namespace-protocol/src/Namespace.sol#L147](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L147)
- [canto-namespace-protocol/src/Namespace.sol#L150](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L150)
- [canto-namespace-protocol/src/Namespace.sol#L174](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L174)

```solidity
File: canto-namespace-protocol/src/Tray.sol

129:         for (uint256 i; i < TILES_PER_TRAY; ++i) {

159:         for (uint256 i; i < _amount; ++i) {

161:             for (uint256 j; j < TILES_PER_TRAY; ++j) {

```
- [canto-namespace-protocol/src/Tray.sol#L129](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L129)
- [canto-namespace-protocol/src/Tray.sol#L159](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L159)
- [canto-namespace-protocol/src/Tray.sol#L161](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L161)

```solidity
File: canto-namespace-protocol/src/Utils.sol

109:             for (uint256 i = 3; i < numBytes; ++i) {

146:             for (uint256 i; i < numAbove; ++i) {

155:             for (uint256 i; i < numMiddle; ++i) {

164:             for (uint256 i; i < numBelow; ++i) {

225:         for (uint256 i; i < _tiles.length; ++i) {

```
- [canto-namespace-protocol/src/Utils.sol#L109](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L109)
- [canto-namespace-protocol/src/Utils.sol#L146](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L146)
- [canto-namespace-protocol/src/Utils.sol#L155](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L155)
- [canto-namespace-protocol/src/Utils.sol#L164](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L164)
- [canto-namespace-protocol/src/Utils.sol#L225](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L225)


&nbsp;
## [G-07] Change `public` functions to `external`

Functions marked as `public` that are not called internally should be set to `external` to save gas and improve code quality. External call cost is less expensive than of public functions.

Instances: 3
```solidity
File: canto-bio-protocol/src/Bio.sol

43:     function tokenURI(uint256 _id) public view override returns (string memory) {

```
- [canto-bio-protocol/src/Bio.sol#L43](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L43)

```solidity
File: canto-namespace-protocol/src/Namespace.sol

90:     function tokenURI(uint256 _id) public view override returns (string memory) {

```
- [canto-namespace-protocol/src/Namespace.sol#L90](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L90)

```solidity
File: canto-namespace-protocol/src/Tray.sol

119:     function tokenURI(uint256 _id) public view override returns (string memory) {

```
- [canto-namespace-protocol/src/Tray.sol#L119](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L119)


&nbsp;
## [G-08] Use a more recent version of Solidity

Use a Solidity version of at least 0.8.2 to get simple compiler automatic inlining.

Use a Solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads.

Use a Solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than `revert()`/`require()` strings.

Use a Solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value.

Use a Solidity version of at least 0.8.12 to get string.concat() to be used instead of abi.encodePacked(<str>,<str>).

Use a solidity version of at least 0.8.13 to get the ability to use using for with a list of free functions.

Instances: 4
```solidity
File: canto-pfp-protocol/src/ProfilePicture.sol

2: pragma solidity >=0.8.0;

```
- [canto-pfp-protocol/src/ProfilePicture.sol#L2](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L2)

```solidity
File: canto-bio-protocol/src/Bio.sol

2: pragma solidity >=0.8.0;

```
- [canto-bio-protocol/src/Bio.sol#L2](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L2)

```solidity
File: canto-namespace-protocol/src/Namespace.sol

2: pragma solidity >=0.8.0;

```
- [canto-namespace-protocol/src/Namespace.sol#L2](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L2)

```solidity
File: canto-namespace-protocol/src/Tray.sol

2: pragma solidity >=0.8.0;

```
- [canto-namespace-protocol/src/Tray.sol#L2](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L2)


&nbsp;
## [G-09] Use named return values

Using named return values instead of explicitly calling `return` saves gas.

Instances: 8
```solidity
File: canto-pfp-protocol/src/ProfilePicture.sol

70:     function tokenURI(uint256 _id) public view override returns (string memory) {

```
- [canto-pfp-protocol/src/ProfilePicture.sol#L70](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L70)

```solidity
File: canto-bio-protocol/src/Bio.sol

43:     function tokenURI(uint256 _id) public view override returns (string memory) {

```
- [canto-bio-protocol/src/Bio.sol#L43](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L43)

```solidity
File: canto-namespace-protocol/src/Namespace.sol

90:     function tokenURI(uint256 _id) public view override returns (string memory) {

```
- [canto-namespace-protocol/src/Namespace.sol#L90](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L90)

```solidity
File: canto-namespace-protocol/src/Tray.sol

119:     function tokenURI(uint256 _id) public view override returns (string memory) {

276:     function _startTokenId() internal pure override returns (uint256) {

```
- [canto-namespace-protocol/src/Tray.sol#L119](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L119)
- [canto-namespace-protocol/src/Tray.sol#L276](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L276)

```solidity
File: canto-namespace-protocol/src/Utils.sol

77:     ) internal pure returns (bytes memory) {

222:     function generateSVG(Tray.TileData[] memory _tiles, bool _isTray) internal pure returns (string memory) {

270:         returns (bytes memory)

```
- [canto-namespace-protocol/src/Utils.sol#L77](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L77)
- [canto-namespace-protocol/src/Utils.sol#L222](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L222)
- [canto-namespace-protocol/src/Utils.sol#L270](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L270)


&nbsp;
