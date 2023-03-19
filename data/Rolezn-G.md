## Summary<a name="Summary">

### Gas Optimizations
| |Issue|Contexts|Estimated Gas Saved|
|-|:-|:-|:-:|
| [GAS&#x2011;1](#GAS&#x2011;1) | `abi.encode()` is less efficient than `abi.encodepacked()` | 1 | 100 |
| [GAS&#x2011;2](#GAS&#x2011;2) | Setting the `constructor` to `payable` | 4 | 52 |
| [GAS&#x2011;3](#GAS&#x2011;3) | Using fixed bytes is cheaper than using `string` | 1 | - |
| [GAS&#x2011;4](#GAS&#x2011;4) | Using `delete` statement can save gas | 1 | - |
| [GAS&#x2011;5](#GAS&#x2011;5) | Functions guaranteed to revert when called by normal users can be marked `payable` | 5 | 105 |
| [GAS&#x2011;6](#GAS&#x2011;6) | `internal` functions only called once can be inlined to save gas | 4 | 88 |
| [GAS&#x2011;7](#GAS&#x2011;7) | Optimize names to save gas | 4 | 88 |
| [GAS&#x2011;8](#GAS&#x2011;8) | `<x> += <y>` Costs More Gas Than `<x> = <x> + <y>` For State Variables | 1 | - |
| [GAS&#x2011;9](#GAS&#x2011;9) | Public Functions To External | 5 | - |
| [GAS&#x2011;10](#GAS&#x2011;10) | Save gas with the use of specific import statements | 9 | - |
| [GAS&#x2011;11](#GAS&#x2011;11) | Help The Optimizer By Saving A Storage Variable’s Reference Instead Of Repeatedly Fetching It | 6 | - |
| [GAS&#x2011;12](#GAS&#x2011;12) | String literals passed to `abi.encode()`/`abi.encodePacked()` should not be split by commas | 5 | 105 |
| [GAS&#x2011;13](#GAS&#x2011;13) | Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead | 13 | - |
| [GAS&#x2011;14](#GAS&#x2011;14) | Use solidity version 0.8.19 to gain some gas boost | 5 | 440 |
| [GAS&#x2011;15](#GAS&#x2011;15) | Use `uint256(1)`/`uint256(2)` instead for `true` and `false` boolean states | 7 | 35000 |

Total: 71 contexts over 15 issues

## Gas Optimizations

### <a href="#Summary">[GAS&#x2011;1]</a><a name="GAS&#x2011;1"> `abi.encode()` is less efficient than `abi.encodepacked()`

See for more information: https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison 

#### <ins>Proof Of Concept</ins>


```solidity
162: lastHash = keccak256(abi.encode(lastHash));
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Tray.sol#L162



### <a href="#Summary">[GAS&#x2011;2]</a><a name="GAS&#x2011;2"> Setting the `constructor` to `payable`

Saves ~13 gas per instance

#### <ins>Proof Of Concept</ins>

```solidity
32: constructor() ERC721("Biography", "Bio")
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-bio-protocol/src/Bio.sol#L32

```solidity
73: constructor(
        address _tray,
        address _note,
        address _revenueAddress
    ) ERC721("Namespace", "NS") Owned(msg.sender)
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Namespace.sol#L73

```solidity
98: constructor(
        bytes32 _initHash,
        uint256 _trayPrice,
        address _revenueAddress,
        address _note,
        address _namespaceNFT
    ) ERC721A("Namespace Tray", "NSTRAY") Owned(msg.sender)
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Tray.sol#L98

```solidity
57: constructor(address _cidNFT, string memory _subprotocolName) ERC721("Profile Picture", "PFP")
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-pfp-protocol/src/ProfilePicture.sol#L57






### <a href="#Summary">[GAS&#x2011;3]</a><a name="GAS&#x2011;3"> Using fixed bytes is cheaper than using `string`

As a rule of thumb, use `bytes` for arbitrary-length raw byte data and string for arbitrary-length `string` (UTF-8) data. If you can limit the length to a certain number of bytes, always use one of `bytes1` to `bytes32` because they are much cheaper.

#### <ins>Proof Of Concept</ins>


```solidity
99: string memory text = '<text x="50%" y="50%" dominant-baseline="middle" text-anchor="middle">';
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-bio-protocol/src/Bio.sol#L99






### <a href="#Summary">[GAS&#x2011;4]</a><a name="GAS&#x2011;4"> Using `delete` statement can save gas

#### <ins>Proof Of Concept</ins>

```solidity
93: bytesOffset = 0;

```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-bio-protocol/src/Bio.sol#L93





### <a href="#Summary">[GAS&#x2011;5]</a><a name="GAS&#x2011;5"> Functions guaranteed to revert when called by normal users can be marked `payable`

If a function modifier or require such as onlyOwner/onlyX is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

#### <ins>Proof Of Concept</ins>

```solidity
196: function changeNoteAddress(address _newNoteAddress) external onlyOwner {

```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Namespace.sol#L196

```solidity
204: function changeRevenueAddress(address _newRevenueAddress) external onlyOwner {

```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Namespace.sol#L204

```solidity
210: function changeNoteAddress(address _newNoteAddress) external onlyOwner {

```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Tray.sol#L210

```solidity
218: function changeRevenueAddress(address _newRevenueAddress) external onlyOwner {

```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Tray.sol#L218

```solidity
225: function endPrelaunchPhase() external onlyOwner {

```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Tray.sol#L225



#### <ins>Recommended Mitigation Steps</ins>
Functions guaranteed to revert when called by normal users can be marked payable.



### <a href="#Summary">[GAS&#x2011;6]</a><a name="GAS&#x2011;6"> `internal` functions only called once can be inlined to save gas

#### <ins>Proof Of Concept</ins>

```solidity
231: function _beforeTokenTransfers

```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Tray.sol#L231

```solidity
276: function _startTokenId

```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Tray.sol#L276

```solidity
73: function characterToUnicodeBytes

```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L73

```solidity
222: function generateSVG

```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L222







### <a href="#Summary">[GAS&#x2011;7]</a><a name="GAS&#x2011;7"> Optimize names to save gas

Contracts most called functions could simply save gas by function ordering via Method ID. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because 22 gas are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions. 

See more <a href="https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92">here</a>

#### <ins>Proof Of Concept</ins>

All in-scope contracts

#### <ins>Recommended Mitigation Steps</ins>
Find a lower method ID name for the most called functions for example Call() vs. Call1() is cheaper by 22 gas
For example, the function IDs in the Gauge.sol contract will be the most used; A lower method ID may be given.



### <a href="#Summary">[GAS&#x2011;8]</a><a name="GAS&#x2011;8"> `<x> += <y>` Costs More Gas Than `<x> = <x> + <y>` For State Variables

#### <ins>Proof Of Concept</ins>


```solidity
150: numBytes += numBytesChar;

```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Namespace.sol#L150





### <a href="#Summary">[GAS&#x2011;9]</a><a name="GAS&#x2011;9"> Public Functions To External

The following functions could be set external to save gas and improve code quality.
External call cost is less expensive than of public functions.

#### <ins>Proof Of Concept</ins>


```solidity
function tokenURI(uint256 _id) public view override returns (string memory) {
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-bio-protocol/src/Bio.sol#L43

```solidity
function tokenURI(uint256 _id) public view override returns (string memory) {
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Namespace.sol#L90

```solidity
function tokenURI(uint256 _id) public view override returns (string memory) {
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Tray.sol#L119

```solidity
function tokenURI(uint256 _id) public view override returns (string memory) {
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-pfp-protocol/src/ProfilePicture.sol#L70

```solidity
function getPFP(uint256 _pfpID) public view returns (address nftContract, uint256 nftID) {
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-pfp-protocol/src/ProfilePicture.sol#L94






### <a href="#Summary">[GAS&#x2011;10]</a><a name="GAS&#x2011;10"> Non-usage of specific imports

The current form of relative path import is not recommended for use because it can unpredictably pollute the namespace.
Instead, the Solidity docs recommend specifying imported symbols explicitly.
https://docs.soliditylang.org/en/v0.8.15/layout-of-source-files.html#importing-other-source-files

A good example:
```solidity
import {OwnableUpgradeable} from "openzeppelin-contracts-upgradeable/contracts/access/OwnableUpgradeable.sol";
import {SafeTransferLib} from "solmate/utils/SafeTransferLib.sol";
import {SafeCastLib} from "solmate/utils/SafeCastLib.sol";
import {ERC20} from "solmate/tokens/ERC20.sol";
import {IProducer} from "src/interfaces/IProducer.sol";
import {GlobalState, UserState} from "src/Common.sol";
```

#### <ins>Proof Of Concept</ins>


```solidity
7: import "../interface/Turnstile.sol";

```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-bio-protocol/src/Bio.sol#L7

```solidity
7: import "./Tray.sol";
8: import "./Utils.sol";
9: import "../interface/Turnstile.sol";

```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Namespace.sol#L7-L9

```solidity
10: import "./Utils.sol";
11: import "../interface/Turnstile.sol";

```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Tray.sol#L10-L11

```solidity
4: import "./Tray.sol";

```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L4

```solidity
5: import "../interface/Turnstile.sol";
6: import "../interface/ICidNFT.sol";

```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-pfp-protocol/src/ProfilePicture.sol#L5-L6




#### <ins>Recommended Mitigation Steps</ins>

Use specific imports syntax per solidity docs recommendation.





### <a href="#Summary">[GAS&#x2011;11]</a><a name="GAS&#x2011;11"> Help The Optimizer By Saving A Storage Variable's Reference Instead Of Repeatedly Fetching It

To help the optimizer, declare a storage type variable and use it instead of repeatedly fetching the reference in a map or an array.
The effect can be quite significant.
As an example, instead of repeatedly calling someMap[someIndex], save its reference like this: SomeStruct storage someStruct = someMap[someIndex] and use it.

#### <ins>Proof Of Concept</ins>


```solidity
116: Tray.TileData[] storage nftToMintCharacters = nftCharacters[namespaceIDToMint];
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Namespace.sol#L116

```solidity
169: uint256 currentRegisteredID = nameToToken[nameToRegister];
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Namespace.sol#L169

```solidity
105: EMOJIS[byteOffset],
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L105

```solidity
151: ZALGO_ABOVE_LETTER[characterIndex],
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L151

```solidity
160: ZALGO_OVER_LETTER[characterIndex],
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L160

```solidity
169: ZALGO_BELOW_LETTER[characterIndex],
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L169






### <a href="#Summary">[GAS&#x2011;12]</a><a name="GAS&#x2011;12"> String literals passed to `abi.encode()`/`abi.encodePacked()` should not be split by commas

String literals can be split into multiple parts and still be considered as a single string literal. Adding commas between each chunk makes it no longer a single string, and instead multiple strings. EACH new comma costs 21 gas due to stack operations and separate MSTOREs.

#### <ins>Proof Of Concept</ins>


```solidity
95: abi.encodePacked(
                        '{"name": "',
                        tokenToName[_id],
                        '", "image": "data:image/svg+xml;base64,',
                        Base64.encode(bytes(Utils.generateSVG(nftCharacters[_id], false))),
                        '"}'
                    )
                )
            )
        )

```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Namespace.sol#L95

```solidity
135: abi.encodePacked(
                        '{"name": "Tray #',
                        LibString.toString(_id),
                        '", "image": "data:image/svg+xml

```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Tray.sol#L135





### <a href="#Summary">[GAS&#x2011;13]</a><a name="GAS&#x2011;13"> Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contract's gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html
Each operation involving a `uint8` costs an extra 22-28 gas (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving `uint256`, due to the compiler having to clear the higher bits of the memory word before operating on the `uint8`, as well as the associated stack operations of doing so. Use a larger size then downcast where needed

#### <ins>Proof Of Concept</ins>


```solidity
34: uint8 tileOffset;
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Namespace.sol#L34

```solidity
36: uint8 skinToneModifier;
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Namespace.sol#L36

```solidity
125: uint8 tileOffset = _characterList[i].tileOffset;
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Namespace.sol#L125

```solidity
134: uint8 characterModifier = tileData.characterModifier;
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Namespace.sol#L134

```solidity
59: uint8 fontClass;
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Tray.sol#L59

```solidity
61: uint16 characterIndex;
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Tray.sol#L61

```solidity
63: uint8 characterModifier;
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Tray.sol#L63

```solidity
138: uint8 asciiStartingIndex = 97;
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L138

```solidity
272: uint8 lastByteVal = uint8(_startingSequence[3]);
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L272

```solidity
273: uint8 lastByteSum = lastByteVal + _characterIndex;
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L273






### <a href="#Summary">[GAS&#x2011;14]</a><a name="GAS&#x2011;14"> Use solidity version 0.8.19 to gain some gas boost

Upgrade to the latest solidity version 0.8.19 to get additional gas savings.
See latest release for reference: https://blog.soliditylang.org/2023/02/22/solidity-0.8.19-release-announcement/

#### <ins>Proof Of Concept</ins>


```solidity
pragma solidity >=0.8.0;
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-bio-protocol/src/Bio.sol#L2

```solidity
pragma solidity >=0.8.0;
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Namespace.sol#L2

```solidity
pragma solidity >=0.8.0;
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Tray.sol#L2

```solidity
pragma solidity >=0.8.0;
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L2

```solidity
pragma solidity >=0.8.0;
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-pfp-protocol/src/ProfilePicture.sol#L2








### <a href="#Summary">[GAS&#x2011;15]</a><a name="GAS&#x2011;15"> Use `uint256(1)`/`uint256(2)` instead for `true` and `false` boolean states

If you don't use boolean for storage you will avoid Gwarmaccess 100 gas. In addition, state changes of boolean from `true` to `false` can cost up to ~20000 gas rather than `uint256(2)` to `uint256(1)` that would cost significantly less.

#### <ins>Proof Of Concept</ins>


```solidity
67: prevByteWasContinuation = true;
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-bio-protocol/src/Bio.sol#L67

```solidity
84: prevByteWasContinuation = true;
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-bio-protocol/src/Bio.sol#L84

```solidity
92: prevByteWasContinuation = false;
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-bio-protocol/src/Bio.sol#L92

```solidity
123: bool isLastTrayEntry = true;
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Namespace.sol#L123

```solidity
129: isLastTrayEntry = false;
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Namespace.sol#L129






