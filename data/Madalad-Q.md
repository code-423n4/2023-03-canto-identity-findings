# Low Risk Summary
| |Issue|Instances|
|:-:|:-|:-:|
|[L-01]|Use `_safeMint` instead of `_mint`|3|
|[L-02]|Downcasting `uint` or `int` may result in overflow|6|

Total issues: 2

Total instances: 9

&nbsp;
# Non-Critical Summary
| |Issue|Instances|
|:-:|:-|:-:|
|[N-01]|Use fixed compiler version|4|
|[N-02]|Lines too long|12|
|[N-03]|Update import usages|9|
|[N-04]|Use scientific notation/underscores for large values|2|
|[N-05]|Missing `address(0)` checks in constructor/initialize|3|

Total issues: 5

Total instances: 30

&nbsp;
# Low Risk Issues
## [L-01] Use `_safeMint` instead of `_mint`

Concerning ERC721 tokens, `_mint` can result in tokens being lost if they are sent to a contract that does not implement `IERC721Receiver.onERC721Received`. OpenZeppelin encourages the use of `_safeMint` over `_mint` for this reason.

Instances: 3
```solidity
File: canto-pfp-protocol/src/ProfilePicture.sol

86:         _mint(msg.sender, tokenId);

```
- [canto-pfp-protocol/src/ProfilePicture.sol#L86](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L86)

```solidity
File: canto-bio-protocol/src/Bio.sol

126:         _mint(msg.sender, tokenId);

```
- [canto-bio-protocol/src/Bio.sol#L126](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L126)

```solidity
File: canto-namespace-protocol/src/Namespace.sol

177:         _mint(msg.sender, namespaceIDToMint);

```
- [canto-namespace-protocol/src/Namespace.sol#L177](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L177)

&nbsp;
## [L-02] Downcasting `uint` or `int` may result in overflow

Consider using OpenZeppelin's `SafeCast` library to prevent unexpected overflows.

Instances: 6
```solidity
File: canto-namespace-protocol/src/Tray.sol

259:                 tileData.fontClass = 3 + uint8((res - 80) / 8);

261:                 tileData.fontClass = 5 + uint8((res - 96) / 4);

263:                 tileData.fontClass = 7 + uint8((res - 104) / 2);

```
- [canto-namespace-protocol/src/Tray.sol#L259](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L259)
- [canto-namespace-protocol/src/Tray.sol#L261](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L261)
- [canto-namespace-protocol/src/Tray.sol#L263](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L263)

```solidity
File: canto-namespace-protocol/src/Utils.sol

135:             return abi.encodePacked(bytes1(asciiStartingIndex + uint8(_characterIndex)));

145:             bytes memory character = abi.encodePacked(bytes1(asciiStartingIndex + uint8(_characterIndex)));

215:             return _getUtfSequence(startingSequence, uint8(_characterIndex));

```
- [canto-namespace-protocol/src/Utils.sol#L135](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L135)
- [canto-namespace-protocol/src/Utils.sol#L145](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L145)
- [canto-namespace-protocol/src/Utils.sol#L215](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L215)

&nbsp;

&nbsp;

# Non-Critical Issues
## [N-01] Use fixed compiler version

A floating pragma risks a different compiler version being used in production vs testing, which poses security risks.

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
## [N-02] Lines too long

[Solidity's style guide](https://docs.soliditylang.org/en/v0.8.10/style-guide.html#maximum-line-length) states lines should be kept within 79 characters.
 
Also, if lines exceed 164 characters then a horizontal scroll bar will be required when viewing the file on Github.

Extensions such as prettier are a simple solution.

Instances: 12
```solidity
File: canto-bio-protocol/src/Bio.sol

41:     /// @dev Generates an on-chain SVG with a new line after 40 bytes. Line splitting generally supports UTF-8 multibyte characters and emojis, but is not tested for arbitrary UTF-8 characters

72:                         // Note that we do not need to check i < lengthInBytes - 4, because we assume that it's a valid UTF8 string and these prefixes imply that another byte follows

98:             memory svg = '<svg xmlns="http://www.w3.org/2000/svg" preserveAspectRatio="xMinYMin meet" viewBox="0 0 400 100"><style>text { font-family: sans-serif; font-size: 12px; }</style>';

```
- [canto-bio-protocol/src/Bio.sol#L41](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L41)
- [canto-bio-protocol/src/Bio.sol#L72](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L72)
- [canto-bio-protocol/src/Bio.sol#L98](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L98)

```solidity
File: canto-namespace-protocol/src/Namespace.sol

117:         bytes memory bName = new bytes(numCharacters * 33); // Used to convert into a string. Can be 33 times longer than the string at most (longest zalgo characters is 33 bytes)

```
- [canto-namespace-protocol/src/Namespace.sol#L117](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L117)

```solidity
File: canto-namespace-protocol/src/Tray.sol

62:         /// @notice For generative fonts with randomness (Zalgo), we generate and fix this on minting. For some emojis, it can be set by the user to influence the skin color

```
- [canto-namespace-protocol/src/Tray.sol#L62](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L62)

```solidity
File: canto-namespace-protocol/src/Utils.sol

21:         hex"CC80CC81CC82CC83CC84CC85CC86CC87CC88CC89CC8ACC8BCC8CCC8DCC8ECC8FCC90CC91CC92CC93CC94CC95CC9ACC9BCCBDCCBECCBFCD80CD81CD82CD83CD84CD86CD8ACD8BCD8CCD90CD91CD92CD97CD98CD9BCD9DCD9ECDA0CDA1";

28:         hex"CC96CC97CC98CC99CC9CCC9DCC9ECC9FCCA0CCA1CCA2CCA3CCA4CCA5CCA6CCA7CCA8CCA9CCAACCABCCACCCADCCAECCAFCCB0CCB1CCB2CCB3CCB9CCBACCBBCCBCCD85CD87CD88CD89CD8DCD8ECD93CD94CD95CD96CD99CD9ACD9CCD9FCDA2";

43:         hex"E29CA8E29C85E29D97E29AA1E29895E2AD90E29D8CE29ABDE29D93E28FB0E2AD95E29AABE29ABEE29894E29AAAE29C8BE29C8AF09F9882F09FA4A3F09F98ADF09F9898F09FA5B0F09F988DF09F988AF09F8E89F09F9881F09F9295F09FA5BAF09F9885F09F94A5F09F9984F09F9886F09FA497F09F9889F09F8E82F09FA494F09F9982F09F98B3F09FA5B3F09F988EF09F929CF09F9894F09F9296F09F9180F09F988BF09F988FF09F98A2F09F9297F09F98A9F09F92AFF09F8CB9F09F929EF09F8E88F09F9299F09F9883F09F98A1F09F9290F09F989CF09F9988F09F9884F09FA4A4F09FA4AAF09F9880F09F928BF09F9280F09F9294F09F988CF09F9293F09FA4A9F09F9983F09F98ACF09F98B1F09F98B4F09FA4ADF09F9890F09F8C9EF09F9892F09F9887F09F8CB8F09F9888F09F8EB6F09F8E8AF09FA5B5F09F989EF09F929AF09F96A4F09F92B0F09F989AF09F9191F09F8E81F09F92A5F09F9891F09FA5B4F09F92A9F09FA4AEF09F98A4F09FA4A2F09F8C9FF09F98A5F09F8C88F09F929BF09F989DF09F98ABF09F98B2F09F94B4F09F8CBBF09FA4AFF09FA4ACF09F9895F09F8D80F09F92A6F09FA68BF09FA4A8F09F8CBAF09F98B9F09F8CB7F09F929DF09F92A4F09F90B0F09F9893F09F9298F09F8DBBF09F989FF09F98A3F09FA790F09F98A0F09FA4A0F09F98BBF09F8C99F09F989BF09F998AF09FA7A1F09FA4A1F09FA4ABF09F8CBCF09FA582F09F98B7F09FA493F09FA5B6F09F98B6F09F9896F09F8EB5F09F9899F09F8D86F09FA491F09F9897F09F90B6F09F8D93F09F9185F09F9184F09F8CBFF09F9AA8F09F93A3F09F8D91F09F8D83F09F98AEF09F928EF09F93A2F09F8CB1F09F9981F09F8DB7F09F98AAF09F8C9AF09F8F86F09F8D92F09F9289F09F92A2F09F9B92F09F98B8F09F90BEF09F9A80F09F8EAFF09F8DBAF09F938CF09F93B7F09F92A8F09F8D95F09F8FA0F09F93B8F09F9087F09F9AA9F09F98B0F09F8C8AF09F9095F09F92ABF09F98B5F09F8EA4F09F8FA1F09FA580F09FA4A7F09F8DBEF09F8DB0F09F8D81F09F98AFF09F928CF09F92B8F09FA781F09F98BAF09F92A7F09F92A3F09FA490F09F8D8EF09F90B7F09F90A5F09F938DF09F8E80F09FA587F09F8C9DF09F94ABF09F90B1F09F90A3F09F8EA7F09F929FF09F91B9F09F928DF09F8DBCF09F92A1F09F98BDF09F8D8AF09F98A8F09F8DABF09FA7A2F09FA495F09F9AABF09F8EBCF09F90BBF09F93B2F09F91BBF09F91BFF09F8CAEF09F8DADF09F909FF09F90B8F09F909DF09F9088F09F94B5F09F94AAF09F98A7F09F8C84F09F98BEF09F93B1F09F8D87F09F8CB4F09F90A2F09F8C83F09F91BDF09F8D8CF09F93BAF09F9494F09F8C85F09FA684F09F8EA5F09F8D8BF09FA59AF09F92B2F09F939AF09F9094F09F8EB8F09FA583F09F98BFF09F9A97F09F8C8EF09F948AF09FA685F09F9ABFF09FA686F09F8D89F09F8DACF09FA7B8F09F8DA8F09F939DF09F93A9F09F92B5F09F92ADF09F8C8DF09F8DBFF09FA7BFF09F8F80F09F8D8FF09F8CB3F09F9989F09F98A6F09F8DB9F09F8DA6F09F9B91F09F8D94F09F8D82F09F9092F09F8DAAF09F9980F09F8D97F09F8CA0F09F8EACF09F8CB5F09F8D84F09F9090F09F8DA9F09FA681F09F939EF09F8D85F09F908DF09F92ACF09FA5A4F09F98BCF09F8CBEF09FA780F09F8EAEF09FA7A0F09F8C8FF09F949DF09F8C89F09FA492F09F9197F09F8CB2F09F8D9CF09F90A6F09F8DAFF09F8F85F09F90BCF09F9284F09F91BAF09F949EF09F8E86F09F8EA8F09F8D9EF09F8E87F09FA69CF09F9091F09F9099F09FA68DF09F9497F09F9396F09F94B9F09FA593F09FA592F09F8DB8F09F918DF09F998FF09FA4A6F09FA4B7F09F918FF09F918CF09F92AAF09F9189F09FA49EF09F998CF09F9187F09F998BF09F9188F09F918BF09F9695F09F9283F09F918AF09F8F83F09FA498F09FA49DF09FA499F09F9AB6F09F9285F09FA49FF09F918EF09F9987F09F91B6F09FA4B2F09F9186F09F95BAF09F9281F09F9985F09FA79AF09FA4B8F09F9190F09FA49AF09F91BCF09F91A7F09FA49CF09FA4B0F09FA798F09F9986F09F91B8F09F91A6F09F9B8CF09FA49BF09F91AEE29DA4EFB88FE298BAEFB88FE299A5EFB88FE29DA3EFB88FE29C8CEFB88FE29880EFB88FE298B9EFB88FE280BCEFB88FE298A0EFB88FE29EA1EFB88FE29AA0EFB88FE29C94EFB88FE2989DEFB88FE2AC87EFB88FE29D84EFB88FE28189EFB88FE2988EEFB88FE29C9DEFB88FE29898EFB88FE29C88EFB88FE296B6EFB88FE29C8DEFB88FE2AC85EFB88FE29881EFB88FE29891EFB88FE299BBEFB88FF09F9181EFB88FF09F9690EFB88FF09F97A3EFB88FF09F8CA7EFB88FF09F958AEFB88FF09F8FB5EFB88FF09F8F96EFB88FF09F87BAF09F87B8F09F87A7F09F87B7F09F87BAF09F87B2F09F8FB3EFB88FE2808DF09F8C88";

47:     /// Furthermore, we store the number of emojis where the skin tone can be modified such that we can revert when this is not supported (and would result in wrong characters).

140:             // We do not reuse the same seed for the following generations to avoid any symmetries, e.g. that 2 chars above would also always result in 2 chars below

195:                 // Hex encoding: CEB1 E182A6 C688 D483 D2BD CF9D C9A0 D48B CEB9 CA9D C699 CA85 C9B1 C9B3 CF83 CF81 CF99 C9BE CA82 C69A CF85 CA8B C9AF 78 E183A7 C8A5

247:                 '<svg xmlns="http://www.w3.org/2000/svg" preserveAspectRatio="xMinYMin meet" viewBox="0 0 400 200"><style>text { font-family: sans-serif; font-size: 30px; }</style>',

```
- [canto-namespace-protocol/src/Utils.sol#L21](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L21)
- [canto-namespace-protocol/src/Utils.sol#L28](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L28)
- [canto-namespace-protocol/src/Utils.sol#L43](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L43)
- [canto-namespace-protocol/src/Utils.sol#L47](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L47)
- [canto-namespace-protocol/src/Utils.sol#L140](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L140)
- [canto-namespace-protocol/src/Utils.sol#L195](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L195)
- [canto-namespace-protocol/src/Utils.sol#L247](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L247)

&nbsp;
## [N-03] Update import usages

To improve readability and adhere to the rule of modularity and modular programming: only import what you need. Specific imports with curly braces allow us to apply this rule better.

For example:
```
import {ERC721} from "solmate/tokens/ERC721.sol";
```

Instances: 9
```solidity
File: canto-pfp-protocol/src/ProfilePicture.sol

5: import "../interface/Turnstile.sol";

6: import "../interface/ICidNFT.sol";

```
- [canto-pfp-protocol/src/ProfilePicture.sol#L5](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L5)
- [canto-pfp-protocol/src/ProfilePicture.sol#L6](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L6)

```solidity
File: canto-bio-protocol/src/Bio.sol

7: import "../interface/Turnstile.sol";

```
- [canto-bio-protocol/src/Bio.sol#L7](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L7)

```solidity
File: canto-namespace-protocol/src/Namespace.sol

7: import "./Tray.sol";

8: import "./Utils.sol";

9: import "../interface/Turnstile.sol";

```
- [canto-namespace-protocol/src/Namespace.sol#L7](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L7)
- [canto-namespace-protocol/src/Namespace.sol#L8](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L8)
- [canto-namespace-protocol/src/Namespace.sol#L9](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L9)

```solidity
File: canto-namespace-protocol/src/Tray.sol

10: import "./Utils.sol";

11: import "../interface/Turnstile.sol";

```
- [canto-namespace-protocol/src/Tray.sol#L10](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L10)
- [canto-namespace-protocol/src/Tray.sol#L11](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L11)

```solidity
File: canto-namespace-protocol/src/Utils.sol

4: import "./Tray.sol";

```
- [canto-namespace-protocol/src/Utils.sol#L4](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L4)

&nbsp;
## [N-04] Use scientific notation/underscores for large values

e.g. `1e6` or `1_000_000` instead of `1000000`.

Instances: 2
```solidity
File: canto-namespace-protocol/src/Utils.sol

258:             iteratedState = _currState * 15485863;

259:             iteratedState = (iteratedState * iteratedState * iteratedState) % 2038074743;

```
- [canto-namespace-protocol/src/Utils.sol#L258](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L258)
- [canto-namespace-protocol/src/Utils.sol#L259](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L259)

&nbsp;
## [N-05] Missing `address(0)` checks in constructor/initialize

Failing to check for invalid parameters on deployment may result in an erroneous input and require an expensive redeployment.

Instances: 3
```solidity
File: canto-pfp-protocol/src/ProfilePicture.sol

57:     constructor(address _cidNFT, string memory _subprotocolName) ERC721("Profile Picture", "PFP") {

```
- [canto-pfp-protocol/src/ProfilePicture.sol#L57](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L57)

```solidity
File: canto-namespace-protocol/src/Namespace.sol

73:     constructor(

```
- [canto-namespace-protocol/src/Namespace.sol#L73](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L73)

```solidity
File: canto-namespace-protocol/src/Tray.sol

98:     constructor(

```
- [canto-namespace-protocol/src/Tray.sol#L98](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L98)

&nbsp;
