
| S No. | Issue | Instances | Gas Savings (from provided tests) |
|-----|-----|-----|-----|
| [G-01] | x += y costs more gas than x = x + y for state variables | 1 | 113  
| [G-02] | Usage of uints or ints smaller than 32 bytes incurs overhead | 6 | 168

-------------

## G-01 x += y costs more gas than x = x + y for state variables

Using the addition operator instead of plus-equals saves **[113 gas](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)**.

_There is 1 instance of this issue_

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol

```
File: main/canto-namespace-protocol/src/Namespace.sol

150: numBytes += numBytesChar;
```

-----------

## G-02 Usage of uints or ints smaller than 32 bytes incurs overhead

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

[https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html](https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html) Each operation involving a `uint8` costs an extra [**22-28 gas**](https://gist.github.com/IllIllI000/9388d20c70f9a4632eb3ca7836f54977) (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving `uint256`, due to the compiler having to clear the higher bits of the memory word before operating on the `uint8`, as well as the associated stack operations of doing so. Use a larger size then downcast where needed

_There are 6 instances of this issue:_

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol

```
File: main/canto-namespace-protocol/src/Namespace.sol

125: uint8 tileOffset = _characterList[i].tileOffset;
134: uint8 characterModifier = tileData.characterModifier;
```

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol

```
File: main/canto-namespace-protocol/src/Tray.sol

195: function getTile(uint256 _trayId, uint8 _tileOffset) external view returns (TileData memory tileData) {
```

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol

```
File: main/canto-namespace-protocol/src/Utils.sol

131: uint8 asciiStartingIndex = 97;
138: uint8 asciiStartingIndex = 97;
267: function _getUtfSequence(bytes memory _startingSequence, uint8 _characterIndex)
```

----------
