# Report


## Low Issues


| |Issue|
|-|:-|
| [L-1](#L-1) | Use `_safeMint` instead of `_mint`|



### L-1 Use `_safeMint` instead of `_mint`
This will ensure that the recipient is either an EOA or implements `IERC721Receiver`

*Instances (3)*:
```solidity
File: Bio.sol

126:         _mint(msg.sender, tokenId);

```

```solidity
File: Namespace.sol

177:         _mint(msg.sender, namespaceIDToMint);

```

```solidity
File: ProfilePicture.sol

86:         _mint(msg.sender, tokenId);

```




## Non Critical Issues


| |Issue|
|-|:-|
| [NC-1](#NC-1) | Constants on the left are better |
| [NC-2](#NC-2) | Order of functions not following Solidity standard practice |
| [NC-3](#NC-3) | Naming conventions of functions |
| [NC-4](#NC-4) | Use `bytes.concat()` and `string.concat()` instead of `abi.encodePacked()` |




### NC-1 Constants on the left are better
This is [safer in case you forget to put a double equals sign](https://www.moserware.com/2008/01/constants-on-left-are-better-but-this.html), as the compiler wil throw an error instead of assigning the value to the variable.

*Instances (40)*:
```solidity
File: Bio.sol

33:         if (block.chainid == 7700) {

60:             if ((i > 0 && (i + 1) % 40 == 0) || prevByteWasContinuation || i == lengthInBytes - 1) {

73:                         (nextCharacter == 0xE2 && bioTextBytes[i + 2] == 0x80 && bioTextBytes[i + 3] == 0x8D) ||

74:                         (nextCharacter == 0xF0 &&

123:         if (bytes(_bio).length == 0 || bytes(_bio).length > 200) revert InvalidBioLength(bytes(_bio).length);

```

```solidity
File: Namespace.sol

81:         if (block.chainid == 7700) {

112:         if (numCharacters > 13 || numCharacters == 0) revert InvalidNumberOfCharacters(numCharacters);

128:                 if (_characterList[j].trayID == trayID) {

130:                     if (_characterList[j].tileOffset == tileOffset) revert FusingDuplicateCharactersNotAllowed();

140:             if (tileData.fontClass == 0) {

```

```solidity
File: ProfilePicture.sol

60:         if (block.chainid == 7700) {

72:         if (nftContract == address(0)) revert PFPNoLongerOwnedByOriginalOwner(_id);

101:         if (cidNFTID == 0 || addressRegistry.getAddress(cidNFTID) != ERC721(nftContract).ownerOf(nftID)) {

```

```solidity
File: Tray.sol

110:         if (block.chainid == 7700) {

152:         if (prelaunchMinted == type(uint256).max) {

181:         if (msg.sender == namespaceNFT) {

264:                 if (tileData.fontClass == 7) {

```

```solidity
File: Utils.sol

79:         if (_fontClass == 0) {

115:                 if (_characterModifier == 1) {

117:                 } else if (_characterModifier == 2) {

119:                 } else if (_characterModifier == 3) {

121:                 } else if (_characterModifier == 4) {

123:                 } else if (_characterModifier == 5) {

130:         } else if (_fontClass == 1) {

137:         } else if (_fontClass == 7) {

177:             if (_fontClass == 2) {

180:                 if (_characterIndex == 4) return hex"F09D9192";

181:                 if (_characterIndex == 6) return hex"F09D9194";

182:                 if (_characterIndex == 14) return hex"F09D919C";

184:             } else if (_fontClass == 3) {

187:             } else if (_fontClass == 4) {

190:             } else if (_fontClass == 5) {

193:             } else if (_fontClass == 6) {

197:                 if (_characterIndex == 0) {

199:                 } else if (_characterIndex == 1) {

201:                 } else if (_characterIndex < 23 || _characterIndex == 25) {

204:                 } else if (_characterIndex == 23) {

206:                 } else if (_characterIndex == 24) {

209:             } else if (_fontClass == 8) {

212:             } else if (_fontClass == 9) {

```

### NC-2 Order of functions not following Solidity standard practice
Follow Solidity's [style guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions) for function ordering: constructor, receive/fallback, external, public, internal and private.

The following shows external functions declared after public functions.

*Instances (29)*:
```solidity
File: Bio.sol


43:     function tokenURI(uint256 _id) public view override returns (string memory) {

121:     function mint(string calldata _bio) external {

```

```solidity
File: Namespace.sol


90:     function tokenURI(uint256 _id) public view override returns (string memory) {

110:     function fuse(CharacterData[] calldata _characterList) external {

184:     function burn(uint256 _id) external {

196:     function changeNoteAddress(address _newNoteAddress) external onlyOwner {

204:     function changeRevenueAddress(address _newRevenueAddress) external onlyOwner {

```

```solidity
File: ProfilePicture.sol

70:     function tokenURI(uint256 _id) public view override returns (string memory) {

79:     function mint(address _nftContract, uint256 _nftID) external {

```

```solidity
File: Tray.sol


119:     function tokenURI(uint256 _id) public view override returns (string memory) {

150:     function buy(uint256 _amount) external {

173:     function burn(uint256 _id) external {

195:     function getTile(uint256 _trayId, uint8 _tileOffset) external view returns (TileData memory tileData) {

203:     function getTiles(uint256 _trayId) external view returns (TileData[TILES_PER_TRAY] memory tileData) {

210:     function changeNoteAddress(address _newNoteAddress) external onlyOwner {

218:     function changeRevenueAddress(address _newRevenueAddress) external onlyOwner {

225:     function endPrelaunchPhase() external onlyOwner {

```




### NC-3 Use constants for values re-used in different places

This is valid for the block chain id and the Turnstile contract address.
*Instances (3)*:
```solidity
File: Bio.sol

33:         if (block.chainid == 7700) {

35:             Turnstile turnstile = Turnstile(0xEcf044C5B4b867CFda001101c617eCd347095B44);


```

```solidity
File: Namespace.sol

81:         if (block.chainid == 7700) {

83:             Turnstile turnstile = Turnstile(0xEcf044C5B4b867CFda001101c617eCd347095B44);

```

```solidity
File: ProfilePicture.sol

60:         if (block.chainid == 7700) {

62:             Turnstile turnstile = Turnstile(0xEcf044C5B4b867CFda001101c617eCd347095B44);

```



### NC-4 Use `bytes.concat()` and `string.concat()` instead of `abi.encodePacked()`
Since version 0.8.4 for appending bytes or strings, `bytes.concat()` and `string.concat()` can be used instead of `abi.encodePacked()`

*Instances (19)*:


```solidity
File: Utils.sol

105:             bytes memory character = abi.encodePacked(

111:                 character = abi.encodePacked(character, EMOJIS[byteOffset + i]);

116:                     character = abi.encodePacked(character, hex"F09F8FBB");

118:                     character = abi.encodePacked(character, hex"F09F8FBC");

120:                     character = abi.encodePacked(character, hex"F09F8FBD");

122:                     character = abi.encodePacked(character, hex"F09F8FBE");

124:                     character = abi.encodePacked(character, hex"F09F8FBF");

136:             return abi.encodePacked(bytes1(asciiStartingIndex + uint8(_characterIndex)));

146:             bytes memory character = abi.encodePacked(bytes1(asciiStartingIndex + uint8(_characterIndex)));

150:                 character = abi.encodePacked(

159:                 character = abi.encodePacked(

168:                 character = abi.encodePacked(

198:                     return abi.encodePacked(FONT_SQUIGGLE[0], FONT_SQUIGGLE[1]);

200:                     return abi.encodePacked(FONT_SQUIGGLE[2], FONT_SQUIGGLE[3], FONT_SQUIGGLE[4]);

203:                     return abi.encodePacked(FONT_SQUIGGLE[5 + offset], FONT_SQUIGGLE[6 + offset]);

205:                     return abi.encodePacked(FONT_SQUIGGLE[47]);

207:                     return abi.encodePacked(FONT_SQUIGGLE[48], FONT_SQUIGGLE[49], FONT_SQUIGGLE[50]);

```



