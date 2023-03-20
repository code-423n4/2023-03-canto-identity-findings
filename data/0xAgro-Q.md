# QA Report
## Finding Summary

[**Low Severity**](#Low-Severity)
1. [**Unchecked Cast May Overflow**](#1-Unchecked-Cast-May-Overflow)
2. [**Single Step Owner Transfer**](#2-Single-Step-Owner-Transfer)

[**Non-Critical**](#Non-Critical)
1. [**Underscore Notation Not Used or Not Used Consistently**](#1-Underscore-Notation-Not-Used-or-Not-Used-Consistently)
2. [**Explicit Variable Not Used**](#2-Explicit-Variable-Not-Used)
3. [**bytes.concat() Can Be Used Over abi.encodePacked()**](#3-bytesconcat-can-be-used-over-abiencodepacked)
4. [**Lack of NatSpec Contract Headers**](#4-Lack-of-NatSpec-Contract-Headers)
5. [**Inconsistent Named Returns**](#5-Inconsistent-Named-Returns)
6. [**Spelling Mistakes**](#6-Spelling-Mistakes)
7. [**Unnecessary Headers**](#7-Unnecessary-Headers)
8. [**Duplicate Header**](#8-Duplicate-Header)
9. [**Incomplete Function NatSpec**](#9-Incomplete-Function-NatSpec)
10. [**Distracting Comment Asymmetry**](#10-Distracting-Comment-Asymmetry)
11. [**Commented Code**](#11-Commented-Code)
12. [**All Assembly Should Be Commented**](#12-All-Assembly-Should-Be-Commented)
13. [**ERC20 Token Recovery**](#13-ERC20-Token-Recovery)
14. [**Lack of Zero Check on Immutable Variables***](#14-Lack-of-Zero-Check-on-Immutable-Variables)
15. [**Use of Magic Numbers***](#15-Use-of-Magic-Numbers)

[**Style Guide Violations**](#Style-Guide-Violations)
1. [**Maximum Line Length**](#1-Maximum-Line-Length)
2. [**Order of Functions**](#2-Order-of-Functions)
3. [**Function Declaration Style**](#3-Function-Declaration-Style)
4. [**Single Quote String**](#4-Single-Quote-String)

# Low Severity

## 1. Unchecked Cast May Overflow

As of Solidity 0.8 overflows are handled automatically; however, not for casting. For example `uint32(4294967300)` will result in `4` without reversion. Consider using OpenZepplin's SafeCast library. Even if it seems as though a value cannot overflow, it is best to be safe.

*/canto-bio-protocol/src/Bio.sol*

```solidity
77:	uint8(bioTextBytes[i + 4]) >= 187 &&
78:	uint8(bioTextBytes[i + 4]) <= 191) ||
```

*/canto-namespace-protocol/src/Tray.sol*

```solidity
250:	tileData.characterIndex = uint16(charRandValue % NUM_CHARS_EMOJIS);
252:	tileData.characterIndex = uint16(charRandValue % NUM_CHARS_LETTERS);
255:	tileData.characterIndex = uint16(charRandValue % NUM_CHARS_LETTERS_NUMBERS);
259:	tileData.fontClass = 3 + uint8((res - 80) / 8);
261:	tileData.fontClass = 5 + uint8((res - 96) / 4);
263:	tileData.fontClass = 7 + uint8((res - 104) / 2);
267:	tileData.characterModifier = uint8(zalgoSeed % 256);
```

*/canto-namespace-protocol/src/Utils.sol*

```solidity
135:	return abi.encodePacked(bytes1(asciiStartingIndex + uint8(_characterIndex)));
145:	bytes memory character = abi.encodePacked(bytes1(asciiStartingIndex + uint8(_characterIndex)));
215:	return _getUtfSequence(startingSequence, uint8(_characterIndex));
272:	uint8 lastByteVal = uint8(_startingSequence[3]);
278:	_startingSequence[2] = bytes1(uint8(_startingSequence[2]) + 1);
```

## 2. Single Step Owner Transfer

The following contracts use single step owner transer (`solmate/auth/Owned.sol`): 
* [Tray.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L7), [Namespace.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L5).

Consider a 2 step ownership transfer like in [Ownable2Step.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol) to avoid possible transfer errors.

# Non-Critical

## 1. Underscore Notation Not Used or Not Used Consistently

Consider using underscore notation to help with contract readability (Ex. `23453` -> `23_453`).

*/canto-pfp-protocol/src/ProfilePicture.sol*

```solidity
60:	if (block.chainid == 7700) {
```

*/canto-bio-protocol/src/Bio.sol*

```solidity
33:	if (block.chainid == 7700) {
35:	Turnstile turnstile = Turnstile(0xEcf044C5B4b867CFda001101c617eCd347095B44);
```

*/canto-namespace-protocol/src/Namespace.sol*

```solidity
81:	if (block.chainid == 7700) {
83:	Turnstile turnstile = Turnstile(0xEcf044C5B4b867CFda001101c617eCd347095B44);
```

*/canto-namespace-protocol/src/Tray.sol*

```solidity
110:	if (block.chainid == 7700) {
112:	Turnstile turnstile = Turnstile(0xEcf044C5B4b867CFda001101c617eCd347095B44);
```

*/canto-namespace-protocol/src/Utils.sol*

```solidity
64:	uint256 private constant EMOJIS_BYTE_OFFSET_SIX_BYTES = 1515;
65:	uint256 private constant EMOJIS_BYTE_OFFSET_SEVEN_BYTES = 1671;
66:	uint256 private constant EMOJIS_BYTE_OFFSET_EIGHT_BYTES = 1720;
67:	uint256 private constant EMOJIS_BYTE_OFFSET_FOURTEEN_BYTES = 1744;
258:	iteratedState = _currState * 15485863;
259:	iteratedState = (iteratedState * iteratedState * iteratedState) % 2038074743;
```

## 2. Explicit Variable Not Used

As described in the [Solidity documentation](https://docs.soliditylang.org/en/v0.4.21/types.html#integers):
> "`uint` and `int` are aliases for `uint256` and `int256`, respectively". 

There are moments in the codebase where `uint` / `int` is used instead of the explicit `uint256` / `int256`. It is best to be explicit with variable names to lessen confusion. Consider replacing instances of `uint` / `int` with `uint256` / `int256`.

*/canto-bio-protocol/src/Bio.sol*

```solidity
47:	uint lengthInBytes = bioTextBytes.length;
49:	uint lines = (lengthInBytes - 1) / 40 + 1;
55:	uint bytesOffset;
56:	for (uint i; i < lengthInBytes; ++i) {
100:	for (uint i; i < lines; ++i) {
```

## 3. bytes.concat() Can Be Used Over abi.encodePacked()

Consider using `bytes.concat()` instead of `abi.encodePacked()` in contracts with Solidity version >= 0.8.4.

*/canto-bio-protocol/src/Bio.sol*

```solidity
116:	return string(abi.encodePacked("data:application/json;base64,", json));
```

*/canto-namespace-protocol/src/Namespace.sol*

```solidity
95:	abi.encodePacked(
105:	return string(abi.encodePacked("data:application/json;base64,", json));
```

*/canto-namespace-protocol/src/Tray.sol*

```solidity
135:	abi.encodePacked(
145:	return string(abi.encodePacked("data:application/json;base64,", json));
```

*/canto-namespace-protocol/src/Utils.sol*

```solidity
104:	bytes memory character = abi.encodePacked(
110:	character = abi.encodePacked(character, EMOJIS[byteOffset + i]);
115:	character = abi.encodePacked(character, hex"F09F8FBB");
117:	character = abi.encodePacked(character, hex"F09F8FBC");
119:	character = abi.encodePacked(character, hex"F09F8FBD");
121:	character = abi.encodePacked(character, hex"F09F8FBE");
123:	character = abi.encodePacked(character, hex"F09F8FBF");
135:	return abi.encodePacked(bytes1(asciiStartingIndex + uint8(_characterIndex)));
145:	bytes memory character = abi.encodePacked(bytes1(asciiStartingIndex + uint8(_characterIndex)));
149:	character = abi.encodePacked(
158:	character = abi.encodePacked(
167:	character = abi.encodePacked(
197:	return abi.encodePacked(FONT_SQUIGGLE[0], FONT_SQUIGGLE[1]);
199:	return abi.encodePacked(FONT_SQUIGGLE[2], FONT_SQUIGGLE[3], FONT_SQUIGGLE[4]);
202:	return abi.encodePacked(FONT_SQUIGGLE[5 + offset], FONT_SQUIGGLE[6 + offset]);
204:	return abi.encodePacked(FONT_SQUIGGLE[47]);
206:	return abi.encodePacked(FONT_SQUIGGLE[48], FONT_SQUIGGLE[49], FONT_SQUIGGLE[50]);
```

## 4. Lack of NatSpec Contract Headers

No contracts in scope have a fully annotated NatSpec contract header (`@title`, `@author`, etc. see [here](https://docs.soliditylang.org/en/v0.8.17/natspec-format.html) for reference) For example, no contracts in scope have a `@title` tag. Consider adding properly annotated NatSpec headers.

## 5. Inconsistent Named Returns

Some functions use named returns and others do not. It is best for code clearity to keep a consistent style.

1. No contracts only have named returns (EX. `returns(uint256 foo)`).
2. The following contracts only have non-named returns (EX. `returns(uint256)`): [Bio.sol](https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-bio-protocol/src/Bio.sol), and [Namespace.sol](https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Namespace.sol).
3. The following contracts have both: [ProfilePicture.sol](https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-pfp-protocol/src/ProfilePicture.sol), [Tray.sol](https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Tray.sol), and [Utils.sol](https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol).

## 6. Spelling Mistakes

There are some spelling mistakes throughout the codebase. Consider fixing all spelling mistakes.

*canto-pfp-protocol/src/ProfilePicture.sol*

* The word `mainnet` is misspelled as [`mainnnet`](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L61).

*canto-bio-protocol/src/Bio.sol*

* The word `mainnet` is misspelled as [`mainnnet`](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L34).

*canto-namespace-protocol/src/Namespace.sol*

* The word `mainnet` is misspelled as [`mainnnet`](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L82).
* The word `Address` is misspelled as [`Adress`](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L72)
* The word `occurrence` is misspelled as [`occurence`](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L152).

*canto-namespace-protocol/src/Tray.sol*

* The word `mainnet` is misspelled as [`mainnnet`](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L111).

*canto-namespace-protocol/src/Utils.sol*

* The word `utilities` is misspelled as [`utiltities`](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L7).

## 7. Unnecessary Headers

There are generic headers in all scoped contracts ([ex](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L9-L11)). Generic headers help with readability, but add unnecessary bulk - as long as the [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#introduction) is followed. Specifically, the two key elements that should be followed from the Style Guide are [Order of Layout](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-layout) and [Order of Functions](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions).

## 8. Duplicate Header

If it is determined that headers are neccessary, there is a duplicate `STATE` header [here](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L9-L11) and [here](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L16-L18). Consider removing duplicate headers.

It it also good to note that `ADDRESSES` are considered `STATE` yet fall under different headers [here](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L39-L54) and [here](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L12-L27).

## 9. Incomplete Function NatSpec

Function NatSpec headers are not complete. For example, the following are missing `@return` tags:

*canto-pfp-protocol/src/ProfilePicture.sol*

* [`tokenURI`](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L70).

*canto-bio-protocol/src/Bio.sol*

* [`tokenURI`](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L43).

*canto-namespace-protocol/src/Namespace.sol*

* [`tokenURI`](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L90).

*canto-namespace-protocol/src/Tray.sol*

* [`tokenURI`](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L119), [`getTile`](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L195), [`getTiles`](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L203).

*canto-namespace-protocol/src/Utils.sol*

* [`characterToUnicodeBytes`](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L73), [`generateSVG`](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L222), [`_getUtfSequence`](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L267).

It is also good to note that [`_beforeTokenTransfers`](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L231) and [`_drawing`](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L245) have no NatSpec header at all.

## 10. Distracting Comment Asymmetry

In the [`_beforeTokenTransfers`](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L231) function, empty parameter slots are commented in the form:
> `/* from*/`

To prevent sensitive readers from getting distracted, consider changing the comment to a more symmetric form like so: 
> `/* from */`

Seen [here](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L232) and [here](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L235).

## 11. Commented Code

Commented code should be removed prior to production as they no longer serve a purpose and only add bulk or distraction. 

*canto-namespace-protocol/src/Utils.sol*

```Solidity
55:	// uint256 constant EMOJIS_LE_FOURTEEN_BYTES = 420;
```

## 12. All Assembly Should Be Commented

It is best practice to heavily comment all lines of assembly. [These](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L164-L167) lines are commented; however, [these](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L87-L89) lines are not.

## 13. ERC20 Token Recovery

Consider adding a recovery function for Tokens that are accidently sent to the core contracts of the protocol. 

## 14. Lack of Zero Check on Immutable Variables

*This issue adds missed items to the automated audit report found [here](https://gist.github.com/ahmedovv123/ac550b389bcbe21043c613e6c6c1b563#nc-1-missing-checks-for-address0-when-assigning-values-to-address-state-variables)*

All immutable variables should be checked to not be the zero address to avoid owner error. The following variables are not checked in their respected constructor:

*canto-namespace-protocol/src/Tray.sol*

* [`trayPrice`](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L106), [`namespaceNFT`](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L109).

*canto-namespace-protocol/src/Namespace.sol*

* [`tray`](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L78).

*canto-pfp-protocol/src/ProfilePicture.sol*

* [`cidNFT`](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L58).

## 15. Use of Magic Numbers

*This issue adds missed items to the automated audit report found [here](https://gist.github.com/ahmedovv123/ac550b389bcbe21043c613e6c6c1b563#nc-2-constants-should-be-defined-rather-than-using-magic-numbers)*

It is best practice to replace all magic numbers with constants.

*canto-pfp-protocol/src/ProfilePicture.sol*

```Solidity
60:	if (block.chainid == 7700) {
```

*canto-bio-protocol/src/Bio.sol*

```Solidity
33:	if (block.chainid == 7700) {
```

*canto-namespace-protocol/src/Namespace.sol*

```Solidity
83:	Turnstile turnstile = Turnstile(0xEcf044C5B4b867CFda001101c617eCd347095B44);
```

*canto-namespace-protocol/src/Tray.sol*

```Solidity
110:	if (block.chainid == 7700) {
112:	Turnstile turnstile = Turnstile(0xEcf044C5B4b867CFda001101c617eCd347095B44);
```

*canto-namespace-protocol/src/Utils.sol*

```Solidity
258:	iteratedState = _currState * 15485863;
259:	iteratedState = (iteratedState * iteratedState * iteratedState) % 2038074743;
```

# Style Guide Violations

## 1. Maximum Line Length

Lines with greater length than 120 characters are used. The [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-lengthhttps://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-length) suggests that all lines should be 120 characters or less in width.

The following lines are longer than 120 characters, it is suggested to shorten these lines:

*canto-pfp-protocol/src/ProfilePicture.sol*
*  [56](https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-pfp-protocol/src/ProfilePicture.sol#L56). 

*canto-bio-protocol/src/Bio.sol*
*  [41](https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-bio-protocol/src/Bio.sol#L41), [53](https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-bio-protocol/src/Bio.sol#L53), [69](https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-bio-protocol/src/Bio.sol#L69), [72](https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-bio-protocol/src/Bio.sol#L72), [98](https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-bio-protocol/src/Bio.sol#L98). 

*canto-namespace-protocol/src/Namespace.sol*
*  [35](https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Namespace.sol#L35), [117](https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Namespace.sol#L117), [126](https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Namespace.sol#L126), [152](https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Namespace.sol#L152). 

*canto-namespace-protocol/src/Tray.sol*
*  [60](https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Tray.sol#L60), [62](https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Tray.sol#L62), [247](https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Tray.sol#L247). 

*canto-namespace-protocol/src/Utils.sol*
*  [19](https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L19), [21](https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L21), [26](https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L26), [28](https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L28), [33](https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L33), [39](https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L39), [43](https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L43), [45](https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L45), [46](https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L46), [47](https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L47), [49](https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L49), [69](https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L69), [72](https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L72), [140](https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L140), [195](https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L195), [247](https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L247), [263](https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L263). 

## 2. Order of Functions

The Solidity Style Guide suggests the following function order: constructor, receive function (if exists), fallback function (if exists), external, public, internal, private.

The following contracts are not compliant (examples are only to prove the functions are out of order NOT a full description): 

* [ProfilePicture.sol](https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-pfp-protocol/src/ProfilePicture.sol): external functions are positioned after public functions.
* [Bio.sol](https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-bio-protocol/src/Bio.sol): external functions are positioned after public functions.
* [Namespace.sol](https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Namespace.sol): external functions are positioned after public functions.
* [Tray.sol](https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Tray.sol): external functions are positioned after public functions.

## 3. Function Declaration Style

The [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#function-declaration) states that

> "[f]or short function declarations, it is recommended for the opening brace of the function body to be kept on the same line as the function declaration". 

It is not clear what length is exactly meant by "short" or "long". The [maximum line length](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-lengthhttps://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-length) of 120 characters may be an indication, and in that case any expanded function declaration under 120 characters should be on a single line. The following functions in their respective contracts are not compliant by this standard: 

*canto-namespace-protocol/src/Tray.sol*
* [`_beforeTokenTransfers`](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L231). 

## 4. Single Quote String

The [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#other-recommendations) states that:
>"[s]trings should be quoted with double-quotes instead of single-quotes". 

Consider using double-quotes to be compliant if possible.

*/canto-bio-protocol/src/Bio.sol*

```solidity
99:	string memory text = '<text x="50%" y="50%" dominant-baseline="middle" text-anchor="middle">';
101:	text = string.concat(text, '<tspan x="50%" dy="20">', strLines[i], "</tspan>");
106:	'{"name": "Bio #',
108:	'", "description": "',
110:	'", "image": "data:image/svg+xml;base64,',
112:	'"}'
```

*/canto-namespace-protocol/src/Namespace.sol*

```solidity
96:	'{"name": "',
98:	'", "image": "data:image/svg+xml;base64,',
100:	'"}'
```

*/canto-namespace-protocol/src/Tray.sol*

```solidity
136:	'{"name": "Tray #',
138:	'", "image": "data:image/svg+xml;base64,',
140:	'"}'
```

*/canto-namespace-protocol/src/Utils.sol*

```solidity
228:	'<text dominant-baseline="middle" text-anchor="middle" y="100" x="',
230:	'">',
239:	'<rect width="34" height="60" y="70" x="',
241:	'" stroke="black" stroke-width="1" fill="none"></rect>'
```
