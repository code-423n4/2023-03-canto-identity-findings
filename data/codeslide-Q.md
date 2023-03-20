### Low Risk Issues List

| Number | Issues Details          | Context |
| :----: | :---------------------- | :-----: |
| [L-01] | Unsafe casting of uints |   15    |

Total: 1 issue

#### [L-01] Unsafe casting of uints

Downcasting from uint256 in Solidity does not revert on overflow. This can easily result in undesired exploitation or bugs, since developers usually assume that overflows raise errors. [OpenZeppelin's SafeCast](https://docs.openzeppelin.com/contracts/3.x/api/utils#SafeCast) restores this intuition by reverting the transaction when such an operation overflows. Using this library instead of the unchecked operations eliminates an entire class of bugs, so it’s recommended to use it always.

For example:

```solidity
// Before
tileData.characterIndex = uint16(charRandValue % NUM_CHARS_EMOJIS);
```

```solidity
// After
tileData.characterIndex = toUint16(charRandValue % NUM_CHARS_EMOJIS);
```

1. [Bio.sol Line 77](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L77)
2. [Bio.sol Line 78](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L78)
3. [Tray.sol Line 250](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L250)
4. [Tray.sol Line 252](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L252)
5. [Tray.sol Line 255](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L255)
6. [Tray.sol Line 259](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L259)
7. [Tray.sol Line 261](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L261)
8. [Tray.sol Line 263](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L263)
9. [Tray.sol Line 267](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L267)
10. [Utils.sol Line 135](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L135)
11. [Utils.sol Line 135](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L135)
12. [Utils.sol Line 145](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L145)
13. [Utils.sol Line 215](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L215)
14. [Utils.sol Line 272](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L272)
15. [Utils.sol Line 278](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L278)

### Non-Critical Issues List

| Number | Issues Details | Context |
| :-: | :-- | :-: |
| [N-01] | Function grouping and ordering | 4 |
| [N-02] | Maximum Line Length | 6 |
| [N-03] | Typo | 1 |
| [N-04] | Remove commented out code | 1 |
| [N-05] | Constants should be defined rather than using magic numbers | 13 |
| [N-06] | Some number literals can be refactored with underscore | 10 |

Total: 6 issues

#### [N-01] Functions should be grouped according to their visibility and ordered per the Solidity Style Guide

Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. See [Order of Functions](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions).

1. In [Bio.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol), the external function `mint()` function should be above the public `tokenURI()` function.
1. In [Namespace.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol), the external functions `fuse()`, `burn()`, `changeNoteAddress()`, and `changeRevenueAddress()` should be above the public `tokenURI()` function.
1. In [ProfilePicture.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol), the external function `mint()` function should be above the public `tokenURI()` function.
1. In [Tray.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol), the functions should be reordered as follows from top to bottom: (external) `buy()`, `burn()`, `getTile()`, `getTiles()`, `changeNoteAddress()`, `changeRevenueAddress()`, `endPrelaunchPhase()`, (public) `tokenURI()`, (internal) `_beforeTokenTransfers()`, `_startTokenId()`, (private) `_drawing()`.

#### [N-02] The maximum suggested line length is 120 characters per the Solidity Style Guide

Several lines are longer than this, making the code unnecessarily difficult to read and maintain. See [Maximum Line Length](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-length).

The following lines exceed 120 characters. There are also many comment lines that exceed 120 characters, but those are not listed here.

1. [Bio.sol Line 98](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L98)
1. [Utils.sol Line 21](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L21)
1. [Utils.sol Line 28](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L28)
1. [Utils.sol Line 43](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L43)
1. [Utils.sol Line 232](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L232)
1. [Utils.sol Line 247](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L247)

#### [N-03] Typo

1. [Utils.sol Line 7](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L7) "Utiltities" => "Utilities"

#### [N-04] Remove commented out code

Commented-out code is distracting and confusing for developers who read the surrounding code, and its significance is often unclear. It will not get compiled or tested when the code around it changes, so it's likely to break over time. For these reasons, commented-out code should be avoided.

1. [Utils.sol Line 55](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L55)
   ```solidity
   55    // uint256 constant EMOJIS_LE_FOURTEEN_BYTES = 420;
   ```

#### [N-05] Constants should be defined rather than using magic numbers

1. [Bio.sol Line 33](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L33) Magic number: `7700`
   ```solidity
   33    if (block.chainid == 7700) {
   ```
1. [Bio.sol Line 49](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L49) Magic number: `40`
   ```solidity
   49    uint lines = (lengthInBytes - 1) / 40 + 1;
   ```
1. [Bio.sol Line 60](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L60) Magic number: `40`
   ```solidity
   60    if ((i > 0 && (i + 1) % 40 == 0) || prevByteWasContinuation || i == lengthInBytes - 1) {
   ```
1. [Bio.sol Line 77](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L77) Magic number: `187`
   ```solidity
   77    uint8(bioTextBytes[i + 4]) >= 187 &&
   ```
1. [Bio.sol Line 78](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L78) Magic number: `191`
   ```solidity
   78    uint8(bioTextBytes[i + 4]) <= 191) ||
   ```
1. [Bio.sol Line 123](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L123) Magic number: `200`
   ```solidity
   123    if (bytes(_bio).length == 0 || bytes(_bio).length > 200) revert InvalidBioLength(bytes(_bio).length);
   ```
1. [Namespace.sol Line 81](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L81) Magic number: `7700`
   ```solidity
   81    if (block.chainid == 7700) {
   ```
1. [Namespace.sol Line 112](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L112) Magic number: `13`
   ```solidity
   112    if (numCharacters > 13 || numCharacters == 0) revert InvalidNumberOfCharacters(numCharacters);
   ```
1. [Namespace.sol Line 117](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L117) Magic number: `33`
   ```solidity
   117    bytes memory bName = new bytes(numCharacters * 33);
   ```
1. [ProfilePicture.sol Line 81](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L60) Magic number: `7700`
   ```solidity
   60    if (block.chainid == 7700) {
   ```
1. [Tray.sol Line 110](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L110) Magic number: `7700`
   ```solidity
   110    if (block.chainid == 7700) {
   ```
1. [Tray.sol Lines 253-270](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L253-L270) Magic numbers: `64`, `1`, `80`, `2`, `96`, `3`, `8`, `104`, `5,`, `96`, `4`, `108`, `7`, `256`, `9`
   ```solidity
   253   if (res < 64) {
   254       tileData.fontClass = 1;
   255       tileData.characterIndex = uint16(charRandValue % NUM_CHARS_LETTERS_NUMBERS);
   256   } else if (res < 80) {
   257       tileData.fontClass = 2;
   258   } else if (res < 96) {
   259       tileData.fontClass = 3 + uint8((res - 80) / 8);
   260   } else if (res < 104) {
   261       tileData.fontClass = 5 + uint8((res - 96) / 4);
   262   } else if (res < 108) {
   263       tileData.fontClass = 7 + uint8((res - 104) / 2);
   264       if (tileData.fontClass == 7) {
   265               // Set seed for Zalgo to ensure same characters will be always generated for this tile
   266           uint256 zalgoSeed = Utils.iteratePRNG(_seed);
   267           tileData.characterModifier = uint8(zalgoSeed % 256);
   268       }
   269   } else {
   270       tileData.fontClass = 9;
   ```
1. [Utils.sol Lines 131-144](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L131-L144) Magic numbers: `97`, `25`, `22`, `7`, `2`
   ```solidity
   131     uint8 asciiStartingIndex = 97; // Starting index for (lowercase) characters
   132     if (_characterIndex > 25) {
   133         asciiStartingIndex = 22; // Starting index for (lowercase) characters - 25
   134     }
   135     return abi.encodePacked(bytes1(asciiStartingIndex + uint8(_characterIndex)));
   136 } else if (_fontClass == 7) {
   137     // Zalgo
   138     uint8 asciiStartingIndex = 97;
   139     uint256 numAbove = (_characterModifier % 7) + 1;
   140     // We do not reuse the same seed for the following generations to avoid any symmetries, e.g. that 2 chars above would also always result in 2 chars below
   141     _characterModifier = iteratePRNG(_characterModifier);
   142     uint256 numMiddle = _characterModifier % 2;
   143     _characterModifier = iteratePRNG(_characterModifier);
   144     uint256 numBelow = (_characterModifier % 7) + 1;
   ```
   Besides those listed above, Utils.sol has many hardcoded numbers throughout. Some may not necessarily benefit from being replaced by constants, but others would certainly clarify the intent and logic of the code for the reader and maintainer. Please consider a review of this library with respect to the hardcoded numeric values.

#### [N-06] Some number literals can be refactored with underscore

Consider using underscores for number literals greater than 999 to improve readability. Example:

```solidity
// Before
if (block.chainid == 7700) {
```

```solidity
// After
if (block.chainid == 7_700) {
```

1. [Bio.sol Line 33](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L33)
1. [Namespace.sol Line 81](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L81)
1. [ProfilePicture.sol Line 60](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L60)
1. [Tray.sol Line 110](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L110)
1. [Utils.sol Lines 64-67](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L64-L67)
1. [Utils.sol Lines 258-259](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L258-L259)
