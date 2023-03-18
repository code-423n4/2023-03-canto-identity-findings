# QA REPORT


### Summary of non-critical issues
| Number | Issue details | Instances |
|---|---|:---:|
| [NC-1](#NC1) |Missing timelock for critical changes. | 5
| [NC-2](#NC2) |Lines are too long. | 9
| [NC-3](#NC3) |For modern and more readable code, update import usages. | 9
| [NC-4](#NC4) |Add parameter to emitted event. | 1
| [NC-5](#NC5) |Use of `bytes.concat()` instead of `abi.encodePacked()`. | 22
| [NC-6](#NC6) |Constants should be defined rather than using magic numbers | 4

*Total: 6 issues.*

---

### Non-critical Issues

### <a id=NC1>[NC-1]</a> Missing timelock for critical changes.

##### Description
A timelock should be added to functions that perform critical changes.

##### Recommendation
Add Timelock for the following functions.

##### *Instances (5):*
File: [2023-03-canto-identity/canto-namespace-protocol/src/Namespace.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L196 )
```solidity
196: function changeNoteAddress(address _newNoteAddress) external onlyOwner {
204: function changeRevenueAddress(address _newRevenueAddress) external onlyOwner {
```
File: [2023-03-canto-identity/canto-namespace-protocol/src/Tray.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L210 )
```solidity
210: function changeNoteAddress(address _newNoteAddress) external onlyOwner {
218: function changeRevenueAddress(address _newRevenueAddress) external onlyOwner {
225: function endPrelaunchPhase() external onlyOwner {
```

### <a id=NC2>[NC-2]</a> Lines are too long.

##### Description
Usually lines in source code are limited to 80 characters. Today's screens are much larger so it's reasonable to stretch this in some cases. Since the files will most likely reside in GitHub, and GitHub starts using a scroll bar in all cases when the length is over 164 characters, the lines below should be split when they reach that length

##### Recommendation
Reduce number of characters per line to improve readability.

##### *Instances (9):*
File: [2023-03-canto-identity/canto-bio-protocol/src/Bio.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L41 )
```solidity
41: /// @dev Generates an on-chain SVG with a new line after 40 bytes. Line splitting generally supports UTF-8 multibyte characters and emojis, but is not tested for arbitrary UTF-8 characters
72: // Note that we do not need to check i < lengthInBytes - 4, because we assume that it's a valid UTF8 string and these prefixes imply that another byte follows
98: memory svg = '<svg xmlns="http://www.w3.org/2000/svg" preserveAspectRatio="xMinYMin meet" viewBox="0 0 400 100"><style>text { font-family: sans-serif; font-size: 12px; }</style>';
```
File: [2023-03-canto-identity/canto-namespace-protocol/src/Namespace.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L117 )
```solidity
117: bytes memory bName = new bytes(numCharacters * 33); // Used to convert into a string. Can be 33 times longer than the string at most (longest zalgo characters is 33 bytes)
```
File: [2023-03-canto-identity/canto-namespace-protocol/src/Tray.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L62 )
```solidity
62: /// @notice For generative fonts with randomness (Zalgo), we generate and fix this on minting. For some emojis, it can be set by the user to influence the skin color
```
File: [2023-03-canto-identity/canto-namespace-protocol/src/Utils.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L21 )
```solidity
47: /// Furthermore, we store the number of emojis where the skin tone can be modified such that we can revert when this is not supported (and would result in wrong characters).
140: // We do not reuse the same seed for the following generations to avoid any symmetries, e.g. that 2 chars above would also always result in 2 chars below
195: // Hex encoding: CEB1 E182A6 C688 D483 D2BD CF9D C9A0 D48B CEB9 CA9D C699 CA85 C9B1 C9B3 CF83 CF81 CF99 C9BE CA82 C69A CF85 CA8B C9AF 78 E183A7 C8A5
247: '<svg xmlns="http://www.w3.org/2000/svg" preserveAspectRatio="xMinYMin meet" viewBox="0 0 400 200"><style>text { font-family: sans-serif; font-size: 30px; }</style>',
```

### <a id=NC3>[NC-3]</a> For modern and more readable code, update import usages.

##### Description
Solidity code is cleaner in the following way: On the principle that clearer code is better code, you should import the things you want to use. Specific imports with curly braces allow us to apply this rule better. Check out this [article](https://betterprogramming.pub/solidity-tutorial-all-about-imports-c65110e41f3a)

##### Recommendation
Import like this: `import {contract1 , contract2} from "filename.sol";`

##### *Instances (9):*
File: [2023-03-canto-identity/canto-bio-protocol/src/Bio.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L7 )
```solidity
7: import "../interface/Turnstile.sol";
```
File: [2023-03-canto-identity/canto-namespace-protocol/src/Namespace.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L7 )
```solidity
7: import "./Tray.sol";
8: import "./Utils.sol";
9: import "../interface/Turnstile.sol";
```
File: [2023-03-canto-identity/canto-namespace-protocol/src/Tray.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L10 )
```solidity
10: import "./Utils.sol";
11: import "../interface/Turnstile.sol";
```
File: [2023-03-canto-identity/canto-namespace-protocol/src/Utils.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L4 )
```solidity
4: import "./Tray.sol";
```
File: [2023-03-canto-identity/canto-pfp-protocol/src/ProfilePicture.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L5 )
```solidity
5: import "../interface/Turnstile.sol";
6: import "../interface/ICidNFT.sol";
```



### <a id=NC4>[NC-4]</a> Add parameter to emitted event.

##### Description
No parameter has been passed to an emitted event. Add some parameter to better depict what has been done on the blockchain.

##### Recommendation
Consider passing a parameter to the event.

##### *Instances (1):*
File: [2023-03-canto-identity/canto-namespace-protocol/src/Tray.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L228 )
```solidity
228: emit PrelaunchEnded();
```

### <a id=NC5>[NC-5]</a> Use of `bytes.concat()` instead of `abi.encodePacked()`.

##### Description
Rather than using `abi.encodePacked` for appending bytes, since version `0.8.4`, `bytes.concat()` is enabled.

##### Recommendation
Since version `0.8.4` for appending bytes, `bytes.concat()` can be used instead of `abi.encodePacked()`.

##### *Instances (22):*
File: [2023-03-canto-identity/canto-bio-protocol/src/Bio.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L116 )
```solidity
116: return string(abi.encodePacked("data:application/json;base64,", json));
```
File: [2023-03-canto-identity/canto-namespace-protocol/src/Namespace.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L95 )
```solidity
95: abi.encodePacked(
105: return string(abi.encodePacked("data:application/json;base64,", json));
```
File: [2023-03-canto-identity/canto-namespace-protocol/src/Tray.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L135 )
```solidity
135: abi.encodePacked(
145: return string(abi.encodePacked("data:application/json;base64,", json));
```
File: [2023-03-canto-identity/canto-namespace-protocol/src/Utils.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L104 )
```solidity
104: bytes memory character = abi.encodePacked(
110: character = abi.encodePacked(character, EMOJIS[byteOffset + i]);
115: character = abi.encodePacked(character, hex"F09F8FBB");
117: character = abi.encodePacked(character, hex"F09F8FBC");
119: character = abi.encodePacked(character, hex"F09F8FBD");
121: character = abi.encodePacked(character, hex"F09F8FBE");
123: character = abi.encodePacked(character, hex"F09F8FBF");
135: return abi.encodePacked(bytes1(asciiStartingIndex + uint8(_characterIndex)));
145: bytes memory character = abi.encodePacked(bytes1(asciiStartingIndex + uint8(_characterIndex)));
149: character = abi.encodePacked(
158: character = abi.encodePacked(
167: character = abi.encodePacked(
197: return abi.encodePacked(FONT_SQUIGGLE[0], FONT_SQUIGGLE[1]);
199: return abi.encodePacked(FONT_SQUIGGLE[2], FONT_SQUIGGLE[3], FONT_SQUIGGLE[4]);
202: return abi.encodePacked(FONT_SQUIGGLE[5 + offset], FONT_SQUIGGLE[6 + offset]);
204: return abi.encodePacked(FONT_SQUIGGLE[47]);
206: return abi.encodePacked(FONT_SQUIGGLE[48], FONT_SQUIGGLE[49], FONT_SQUIGGLE[50]);
```
### <a id=NC6>[NC-6]</a> Constants should be defined rather than using magic numbers

##### Description
Even [`assembly`](https://github.com/code-423n4/2022-05-opensea-seaport/blob/9d7ce4d08bf3c3010304a0476a785c70c0e90ae7/contracts/lib/TokenTransferrer.sol#L35-L39) can benefit from using readable constants instead of hex/numeric literals

##### Recommendation
You should declare the variable instead of magic number.

##### *Instances (4):*
File: [2023-03-canto-identity/canto-bio-protocol/src/Bio.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L33 )
```solidity
33: if (block.chainid == 7700) {
```
File: [2023-03-canto-identity/canto-namespace-protocol/src/Namespace.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L81 )
```solidity
81: if (block.chainid == 7700) {
```
File: [2023-03-canto-identity/canto-namespace-protocol/src/Tray.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L110 )
```solidity
110: if (block.chainid == 7700) {
```

File: [2023-03-canto-identity/canto-pfp-protocol/src/ProfilePicture.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L60 )
```solidity
60: if (block.chainid == 7700) {

```
