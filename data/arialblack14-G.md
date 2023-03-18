# GAS OPTIMIZATION REPORT
---
### Summary of optimizations
| Number | Issue details | Instances |
|---|---|:---:|
| [G-1](#G1) |Usage of  `uint`s/`int`s smaller than 32 bytes (256 bits) incurs overhead. | 29
| [G-2](#G2) |`abi.encode()` is less efficient than `abi.encodePacked()` | 1
| [G-3](#G3) |Internal functions only called once can be inlined to save gas. | 1
| [G-4](#G4) |Multiple accesses of a mapping/array should use a local variable cache. | 7
| [G-5](#G5) |`>=` costs less gas than `>`. | 5
| [G-6](#G6) |`<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables | 1
| [G-7](#G7) |Using fixed bytes is cheaper than using  `string`. | 14

*Total: 7 issues.*

---
### Gas Optimizations

### <a id=G2=1>[G-1]</a> Usage of  `uint`s/`int`s smaller than 32 bytes (256 bits) incurs overhead.

##### Description
When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

##### Recommendation
Use `uint256` instead.

##### *Instances (29):*
File: [2023-03-canto-identity/canto-bio-protocol/src/Bio.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L77 )
```solidity
77: uint8(bioTextBytes[i + 4]) >= 187 &&
78: uint8(bioTextBytes[i + 4]) <= 191) ||
```
File: [2023-03-canto-identity/canto-namespace-protocol/src/Namespace.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L34 )
```solidity
34: uint8 tileOffset;
36: uint8 skinToneModifier;
125: uint8 tileOffset = _characterList[i].tileOffset;
134: uint8 characterModifier = tileData.characterModifier;
```
File: [2023-03-canto-identity/canto-namespace-protocol/src/Tray.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L59 )
```solidity
59: uint8 fontClass;
61: uint16 characterIndex;
63: uint8 characterModifier;
195: function getTile(uint256 _trayId, uint8 _tileOffset) external view returns (TileData memory tileData) {
250: tileData.characterIndex = uint16(charRandValue % NUM_CHARS_EMOJIS);
252: tileData.characterIndex = uint16(charRandValue % NUM_CHARS_LETTERS);
255: tileData.characterIndex = uint16(charRandValue % NUM_CHARS_LETTERS_NUMBERS);
259: tileData.fontClass = 3 + uint8((res - 80) / 8);
261: tileData.fontClass = 5 + uint8((res - 96) / 4);
263: tileData.fontClass = 7 + uint8((res - 104) / 2);
267: tileData.characterModifier = uint8(zalgoSeed % 256);
```
File: [2023-03-canto-identity/canto-namespace-protocol/src/Utils.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L12 )
```solidity
12: error EmojiDoesNotSupportSkinToneModifier(uint16 emojiIndex);
74: uint8 _fontClass,
75: uint16 _characterIndex,
131: uint8 asciiStartingIndex = 97; // Starting index for (lowercase) characters
135: return abi.encodePacked(bytes1(asciiStartingIndex + uint8(_characterIndex)));
138: uint8 asciiStartingIndex = 97;
145: bytes memory character = abi.encodePacked(bytes1(asciiStartingIndex + uint8(_characterIndex)));
215: return _getUtfSequence(startingSequence, uint8(_characterIndex));
267: function _getUtfSequence(bytes memory _startingSequence, uint8 _characterIndex)
272: uint8 lastByteVal = uint8(_startingSequence[3]);
273: uint8 lastByteSum = lastByteVal + _characterIndex;
278: _startingSequence[2] = bytes1(uint8(_startingSequence[2]) + 1);
```

### <a id=G2>[G-2]</a> `abi.encode()` is less efficient than `abi.encodePacked()`

##### Description
`abi.encode` will apply ABI encoding rules. Therefore all elementary types are padded to 32 bytes and dynamic arrays include their length. Therefore it is possible to also decode this data again (with `abi.decode`) when the type are known.

`abi.encodePacked` will only use the only use the minimal required memory to encode the data. E.g. an address will only use 20 bytes and for dynamic arrays only the elements will be stored without length. For more info see the Solidity docs for packed mode.

For the input of the `keccak` method it is important that you can ensure that the resulting bytes of the encoding are unique. So if you always encode the same types and arrays always have the same length then there is no problem. But if you switch the parameters that you encode or encode multiple dynamic arrays you might have conflicts.
For example:
`abi.encodePacked(address(0x0000000000000000000000000000000000000001), uint(0))` encodes to the same as `abi.encodePacked(uint(0x0000000000000000000000000000000000000001000000000000000000000000), address(0))` 
and `abi.encodePacked(uint[](1,2), uint[](3))` encodes to the same as `abi.encodePacked(uint[](1), uint[](2,3))`
Therefore these examples will also generate the same hashes even so they are different inputs.
On the other hand you require less memory and therefore in most cases abi.encodePacked uses less gas than abi.encode.

##### Recommendation
Use `abi.encodePacked()` where possible to save gas

##### *Instances (1):*
File: [2023-03-canto-identity/canto-namespace-protocol/src/Tray.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L162 )
```solidity
162: lastHash = keccak256(abi.encode(lastHash));
```

### <a id=G3>[G-3]</a> Internal functions only called once can be inlined to save gas.

##### Description
Not inlining costs 20 to 40 gas because of two extra `JUMP` instructions and additional stack operations needed for function calls.
Check this out: [link](https://betterprogramming.pub/solidity-gas-optimizations-and-tricks-2bcee0f9f1f2)
Note: To sum up, when there is an internal function that is only called once, there is no point for that function to exist. 

##### Recommendation
Remove the function and make it inline if it is a simple function.

##### *Instances (1):*
File: [2023-03-canto-identity/canto-namespace-protocol/src/Tray.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L276 )
```solidity
276: function _startTokenId() internal pure override returns (uint256) {
```

### <a id=G4>[G-4]</a> Multiple accesses of a mapping/array should use a local variable cache.

##### Description
The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping's value in a local storage or calldata variable when the value is accessed multiple times, saves ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations. Caching an array's struct avoids recalculating the array offsets into memory/calldata. [Similar findings](https://github.com/code-423n4/2022-06-infinity-findings/issues/186)

##### Recommendation
Use a local variable cache.

##### *Instances (7):*
File: [2023-03-canto-identity/canto-namespace-protocol/src/Namespace.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L124 )
```solidity
124: uint256 trayID = _characterList[i].trayID;
125: uint8 tileOffset = _characterList[i].tileOffset;
128: if (_characterList[j].trayID == trayID) {
130: if (_characterList[j].tileOffset == tileOffset) revert FusingDuplicateCharactersNotAllowed();
136: if (tileData.fontClass != 0 && _characterList[i].skinToneModifier != 0) {
142: characterModifier = _characterList[i].skinToneModifier;
```
File: [2023-03-canto-identity/canto-namespace-protocol/src/Utils.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L232 )
```solidity
232: characterToUnicodeBytes(_tiles[i].fontClass, _tiles[i].characterIndex, _tiles[i].characterModifier)
```

### <a id=G5>[G-5]</a> `>=` costs less gas than `>`.

##### Description
The compiler uses opcodes `GT` and `ISZERO` for solidity code that uses `>`, but only requires `LT` for `>=`, which [saves 3 gas](https://gist.github.com/IllIllI000/3dc79d25acccfa16dee4e83ffdc6ffde)

##### Recommendation
Use `>=` instead if appropriate.

##### *Instances (5):*
File: [2023-03-canto-identity/canto-bio-protocol/src/Bio.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L60 )
```solidity
60: if ((i > 0 && (i + 1) % 40 == 0) || prevByteWasContinuation || i == lengthInBytes - 1) {
123: if (bytes(_bio).length == 0 || bytes(_bio).length > 200) revert InvalidBioLength(bytes(_bio).length);
```
File: [2023-03-canto-identity/canto-namespace-protocol/src/Namespace.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L112 )
```solidity
112: if (numCharacters > 13 || numCharacters == 0) revert InvalidNumberOfCharacters(numCharacters);
```
File: [2023-03-canto-identity/canto-namespace-protocol/src/Tray.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L155 )
```solidity
155: if (startingTrayId + _amount - 1 > PRE_LAUNCH_MINT_CAP) revert MintExceedsPreLaunchAmount();
```
File: [2023-03-canto-identity/canto-namespace-protocol/src/Utils.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L132 )
```solidity
132: if (_characterIndex > 25) {
```

### <a id=G6>[G-6]</a> `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables

##### Description
Using the addition/subtraction operator instead of plus-equals/minus-equals saves **[113 gas](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)**

##### Recommendation
Use `<x> = <x> + <y>` or `<x> = <x> - <y>` instead.

##### *Instances (1):*
File: [2023-03-canto-identity/canto-namespace-protocol/src/Namespace.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L150 )
```solidity
150: numBytes += numBytesChar;
```

### <a id=G7>[G-7]</a> Using fixed bytes is cheaper than using  `string`.

##### Description
As a rule of thumb, use bytes for arbitrary-length raw byte data and `string` for arbitrary-length string (UTF-8) data. If you can limit the length to a certain number of bytes, always use one of `bytes1` to `bytes32` because they are much cheaper.

##### Recommendation
Use fixed bytes instead of `string`.

##### *Instances (14):*
File: [2023-03-canto-identity/canto-bio-protocol/src/Bio.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L43 )
```solidity
43: function tokenURI(uint256 _id) public view override returns (string memory) {
45: string memory bioText = bio[_id];
99: string memory text = '<text x="50%" y="50%" dominant-baseline="middle" text-anchor="middle">';
103: string memory json = Base64.encode(
```
File: [2023-03-canto-identity/canto-namespace-protocol/src/Namespace.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L90 )
```solidity
90: function tokenURI(uint256 _id) public view override returns (string memory) {
92: string memory json = Base64.encode(
168: string memory nameToRegister = string(bName);
188: string memory associatedName = tokenToName[_id];
```
File: [2023-03-canto-identity/canto-namespace-protocol/src/Tray.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L119 )
```solidity
119: function tokenURI(uint256 _id) public view override returns (string memory) {
132: string memory json = Base64.encode(
```
File: [2023-03-canto-identity/canto-namespace-protocol/src/Utils.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L222 )
```solidity
222: function generateSVG(Tray.TileData[] memory _tiles, bool _isTray) internal pure returns (string memory) {
223: string memory innerData;
```
File: [2023-03-canto-identity/canto-pfp-protocol/src/ProfilePicture.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L57 )
```solidity
57: constructor(address _cidNFT, string memory _subprotocolName) ERC721("Profile Picture", "PFP") {
70: function tokenURI(uint256 _id) public view override returns (string memory) {
```