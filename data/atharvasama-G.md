# Gas Optimizations list

| Number | Details                                                                         | Instances |
| ------ | ------------------------------------------------------------------------------- | --------- |
| 1      | CAN DECLARE VARIABLE OUTSIDE LOOP TO SAVE GAS                                   | 12        |
| 2      | AVOID CONTRACT EXISTENCE CHECKS BY USING LOW LEVEL CALLS                        | 1         |
| 3      | PUBLIC FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE DECLARED EXTERNAL INSTEAD | 3         |
| 4      | OPTIMIZE NAMES TO SAVE GAS                                                      | 5         |
| 5      | INTERNAL FUNCTIONS ONLY CALLED ONCE CAN BE INLINED TO SAVE GAS                  | 1         |
| 6      | BEFORE SOME FUNCTIONS, WE SHOULD CHECK SOME VARIABLES FOR POSSIBLE GAS SAVE     | 1         |
| 7      | USAGE OF UINT/INT SMALLER THAN 32 BYTES (256 BITS) INCURS OVERHEAD              | 11        |
| 8      | USE A MORE RECENT VERSION OF SOLIDITY                                           | 5         |

# Gas Optimizations

## [G-01] CAN DECLARE VARIABLE OUTSIDE LOOP TO SAVE GAS 
Declaring the stack variable outside the loop will save gas that would otherwise be spent on declaring the variable over and over again.

[canto-bio-protocol/src/Bio.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol)
```js
57:             bytes1 character = bioTextBytes[i];

61:                 bytes1 nextCharacter;
```

[canto-namespace-protocol/src/Namespace.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol)
```js
123:             bool isLastTrayEntry = true;
124:             uint256 trayID = _characterList[i].trayID;
125:             uint8 tileOffset = _characterList[i].tileOffset;

134:             uint8 characterModifier = tileData.characterModifier;

144:             bytes memory charAsBytes = Utils.characterToUnicodeBytes(0, tileData.characterIndex, characterModifier);

146:             uint256 numBytesChar = charAsBytes.length;
```

[canto-namespace-protocol/src/Tray.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol)
```js
160:             TileData[TILES_PER_TRAY] memory trayTiledata;
```

[canto-namespace-protocol/src/Utils.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol)
```js
148:                 uint256 characterIndex = (_characterModifier % ZALGO_NUM_ABOVE) * 2;

157:                 uint256 characterIndex = (_characterModifier % ZALGO_NUM_OVER) * 2;

166:                 uint256 characterIndex = (_characterModifier % ZALGO_NUM_BELOW) * 2;
```

## [G-02] AVOID CONTRACT EXISTENCE CHECKS BY USING LOW LEVEL CALLS
Before Solidity 0.8.10 the compiler inserted extra code to check for contract existence for external function calls ```EXTCODESIZE``` (100 gas). In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

[canto-namespace-protocol/src/Namespace.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol)
```js
156:                 address trayOwner = tray.ownerOf(trayID);
```

## [G-03] PUBLIC FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE DECLARED EXTERNAL INSTEAD
Contracts are allowed to override their parents’ functions and change the visibility from public to external and can save gas by doing so.

[canto-bio-protocol/src/Bio.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol)
```js
43:     function tokenURI(uint256 _id) public view override returns (string memory) {
```

[canto-namespace-protocol/src/Namespace.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol)
```js
90:     function tokenURI(uint256 _id) public view override returns (string memory) {
```

[canto-namespace-protocol/src/Tray.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol)
```js
119:     function tokenURI(uint256 _id) public view override returns (string memory) {
```

## [G-04] OPTIMIZE NAMES TO SAVE GAS
Function hashes that have two leading zero bytes can save 128 gas each during deployment, and renaming functions to have lower method IDs will save 22 gas per call, per sorted position shifted. Prioritize the most called functions and sort and rename them according to the function hashes/method IDs. For a better understanding please refer to [this link](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92)

A lower method ID may be given to the most frequently used functions. [This](https://emn178.github.io/solidity-optimize-name/) is a useful tool that can be used for the same, if required.

## [G-05] INTERNAL FUNCTIONS ONLY CALLED ONCE CAN BE INLINED TO SAVE GAS
Not inlining costs 20 to 40 gas because of two extra ```JUMP``` instructions and additional stack operations needed for function calls.

[canto-namespace-protocol/src/Tray.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol)
```js
231:     function _beforeTokenTransfers(
232:         address, /* from*/
233:         address to,
234:         uint256 startTokenId,
235:         uint256 /* quantity*/
236:     ) internal view override {
```

## [G-06] BEFORE SOME FUNCTIONS, WE SHOULD CHECK SOME VARIABLES FOR POSSIBLE GAS SAVE
Before transfer, we should check for amount being 0 so the function doesnt run when its not gonna do anything.

[canto-namespace-protocol/src/Tray.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol)
```js
150:     function buy(uint256 _amount) external {
```

## [G-07] USAGE OF UINT/INT SMALLER THAN 32 BYTES (256 BITS) INCURS OVERHEAD
When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.
Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed.

[canto-namespace-protocol/src/Namespace.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol)
```js
34:         uint8 tileOffset;

36:         uint8 skinToneModifier;

125:             uint8 tileOffset = _characterList[i].tileOffset;

134:             uint8 characterModifier = tileData.characterModifier;
```

[canto-namespace-protocol/src/Tray.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol)
```js
59:         uint8 fontClass;

61:         uint16 characterIndex;

63:         uint8 characterModifier;
```

[canto-namespace-protocol/src/Utils.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol)
```js
131:             uint8 asciiStartingIndex = 97; // Starting index for (lowercase) characters

138:             uint8 asciiStartingIndex = 97;

272:         uint8 lastByteVal = uint8(_startingSequence[3]);
273:         uint8 lastByteSum = lastByteVal + _characterIndex;
```

## [G-08] USE A MORE RECENT VERSION OF SOLIDITY
Use a solidity version of at least 0.8.0 to get overflow protection without SafeMath
Use a solidity version of at least 0.8.2 to get simple compiler automatic inlining
Use a solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads
Use a solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than revert()/require() strings
Use a solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value