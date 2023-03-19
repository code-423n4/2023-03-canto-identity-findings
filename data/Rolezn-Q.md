## Summary<a name="Summary">

### Low Risk Issues
| |Issue|Contexts|
|-|:-|:-:|
| [LOW&#x2011;1](#LOW&#x2011;1) | Add to `blacklist` function | 9 |
| [LOW&#x2011;2](#LOW&#x2011;2) | Possible rounding issue | 1 |
| [LOW&#x2011;3](#LOW&#x2011;3) | Low Level Calls With Solidity Version 0.8.14 Can Result In Optimiser Bug | 2 |
| [LOW&#x2011;4](#LOW&#x2011;4) | Use `_safeMint` instead of `_mint` | 4 |
| [LOW&#x2011;5](#LOW&#x2011;5) | Protect your NFT from copying in POW forks | 4 |
| [LOW&#x2011;6](#LOW&#x2011;6) | Remove unused code | 2 |
| [LOW&#x2011;7](#LOW&#x2011;7) | Solmate's SafeTransferLib doesn't check whether the ERC20 contract exists | 1 |

Total: 23 contexts over 7 issues

### Non-critical Issues
| |Issue|Contexts|
|-|:-|:-:|
| [NC&#x2011;1](#NC&#x2011;1) | Add a timelock to critical functions | 4 |
| [NC&#x2011;2](#NC&#x2011;2) | Constants in comparisons should appear on the left side | 3 |
| [NC&#x2011;3](#NC&#x2011;3) | Critical Changes Should Use Two-step Procedure | 4 |
| [NC&#x2011;4](#NC&#x2011;4) | Event emit should emit a parameter | 1 |
| [NC&#x2011;5](#NC&#x2011;5) | Use `delete` to Clear Variables | 1 |
| [NC&#x2011;6](#NC&#x2011;6) | NatSpec return parameters should be included in contracts | 1 |
| [NC&#x2011;7](#NC&#x2011;7) | Initial value check is missing in Set Functions | 4 |
| [NC&#x2011;8](#NC&#x2011;8) | Lines are too long | 15 |
| [NC&#x2011;9](#NC&#x2011;9) | Implementation contract may not be initialized | 4 |
| [NC&#x2011;10](#NC&#x2011;10) | Non-usage of specific imports | 9 |
| [NC&#x2011;11](#NC&#x2011;11) | Use a more recent version of Solidity | 5 |
| [NC&#x2011;12](#NC&#x2011;12) | Use `bytes.concat()` | 22 |

Total: 73 contexts over 20 issues

## Low Risk Issues

### <a href="#Summary">[LOW&#x2011;1]</a><a name="LOW&#x2011;1"> Add to `blacklist` function

NFT thefts have increased recently, so with the addition of hacked NFTs to the platform, NFTs can be converted into liquidity. To prevent this, I recommend adding the blacklist function.

Marketplaces such as Opensea have a blacklist feature that will not list NFTs that have been reported theft, NFT projects such as Manifold have blacklist functions in their smart contracts.

Here is the project example; Manifold

Manifold Contract https://etherscan.io/address/0xe4e4003afe3765aca8149a82fc064c0b125b9e5a#code

```solidity
     modifier nonBlacklistRequired(address extension) {
         require(!_blacklistedExtensions.contains(extension), "Extension blacklisted");
         _;
     }
```


#### <ins>Recommended Mitigation Steps</ins>
Add to Blacklist function and modifier.



### <a href="#Summary">[LOW&#x2011;2]</a><a name="LOW&#x2011;2"> Possible rounding issue

Division by anything smaller than `lengthInBytes - 1` may result in the result being zero, due to solidity not supporting fractions. Consider requiring a minimum amount for the numerator to ensure that it is always larger than the denominator
This will cause lines to be equal `0` and result in failure of executing anything related to `strLines` in the function.

#### <ins>Proof Of Concept</ins>


```solidity
49: uint lines = (lengthInBytes - 1) / 40 + 1;
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-bio-protocol/src/Bio.sol#L49



### <a href="#Summary">[LOW&#x2011;3]</a><a name="LOW&#x2011;3"> Low Level Calls With Solidity Version Before 0.8.14 Can Result In Optimiser Bug

The project contracts in scope are using low level calls with solidity version before 0.8.14 which can result in optimizer bug.
https://medium.com/certora/overly-optimistic-optimizer-certora-bug-disclosure-2101e3f7994d

Simliar findings in Code4rena contests for reference:
https://code4rena.com/reports/2022-06-illuminate/#5-low-level-calls-with-solidity-version-0814-can-result-in-optimiser-bug

#### <ins>Proof Of Concept</ins>

POC can be found in the above medium reference url.

Functions that execute low level calls in contracts with solidity version under 0.8.14

```solidity
87: assembly {
        mstore(bytesLines, bytesOffset)
    }
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-bio-protocol/src/Bio.sol#L87

```solidity
165: assembly {
        mstore(bName, numBytes)
     }
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Namespace.sol#L165



#### <ins>Recommended Mitigation Steps</ins>

Consider upgrading to at least solidity v0.8.15.






### <a href="#Summary">[LOW&#x2011;4]</a><a name="LOW&#x2011;4"> Use `_safeMint` instead of `_mint`

According to openzepplin's ERC721, the use of `_mint` is discouraged, use _safeMint whenever possible.
https://docs.openzeppelin.com/contracts/3.x/api/token/erc721#ERC721-_mint-address-uint256-

#### <ins>Proof Of Concept</ins>


```solidity
126: _mint(msg.sender, tokenId);
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-bio-protocol/src/Bio.sol#L126

```solidity
177: _mint(msg.sender, namespaceIDToMint);
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Namespace.sol#L177

```solidity
167: _mint(msg.sender, _amount);
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Tray.sol#L167

```solidity
86: _mint(msg.sender, tokenId);
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-pfp-protocol/src/ProfilePicture.sol#L86



#### <ins>Recommended Mitigation Steps</ins>

Use `_safeMint` whenever possible instead of `_mint`




### <a href="#Summary">[LOW&#x2011;5]</a><a name="LOW&#x2011;5"> Protect your NFT from copying in POW forks
Ethereum has performed the long-awaited "merge" that will dramatically reduce the environmental impact of the network

There may be forked versions of Ethereum, which could cause confusion and lead to scams as duplicated NFT assets enter the market.

If the Ethereum Merge, which took place in September 2022, results in the Blockchain splitting into two Blockchains due to the 'THE DAO' attack in 2016, this could result in duplication of immutable tokens (NFTs).

In any case, duplicate NFTs will exist due to the ETH proof-of-work chain and other potential forks, and there’s likely to be some level of confusion around which assets are 'official' or 'authentic.'

Even so, there could be a frenzy for these copies, as NFT owners attempt to flip the proof-of-work versions of their valuable tokens.

As ETHPOW and any other forks spin off of the Ethereum mainnet, they will yield duplicate versions of Ethereum’s NFTs. An NFT is simply a blockchain token, and it can work as a deed of ownership to digital items like artwork and collectibles. A forked Ethereum chain will thus have duplicated deeds that point to the same tokenURI

About Merge Replay Attack: https://twitter.com/elerium115/status/1558471934924431363?s=20&t=RRheaYJwo-GmSnePwofgag

#### <ins>Proof Of Concept</ins>


```solidity
43: function tokenURI(uint256 _id) public view override returns (string memory) {

```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-bio-protocol/src/Bio.sol#L43

```solidity
90: function tokenURI(uint256 _id) public view override returns (string memory) {

```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Namespace.sol#L90

```solidity
119: function tokenURI(uint256 _id) public view override returns (string memory) {

```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Tray.sol#L119

```solidity
70: function tokenURI(uint256 _id) public view override returns (string memory) {

```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-pfp-protocol/src/ProfilePicture.sol#L70



#### <ins>Recommended Mitigation Steps</ins>

Add the following check:
```solidity
if(block.chainid != 1) { 
    revert(); 
}
```



### <a href="#Summary">[LOW&#x2011;6]</a><a name="LOW&#x2011;6"> Remove unused code
This code is not used in the main project contract files, remove it or add event-emit
Code that is not in use, suggests that they should not be present and could potentially contain insecure functionalities.

#### <ins>Proof Of Concept</ins>


```solidity
function _beforeTokenTransfers
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Tray.sol#L231

```solidity
function _startTokenId
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Tray.sol#L276




### <a href="#Summary">[LOW&#x2011;7]</a><a name="LOW&#x2011;7"> Solmate's SafeTransferLib doesn't check whether the ERC20 contract exists

Solmate's SafeTransferLib, which is often used to interact with non-compliant/unsafe ERC20 tokens, does not check whether the ERC20 contract exists. The following code will not revert in case the token doesn't exist (yet).

This is stated in the Solmate library: https://github.com/transmissions11/solmate/blob/main/src/utils/SafeTransferLib.sol#L9

#### <ins>Proof Of Concept</ins>


```solidity
6: import {SafeTransferLib} from "solmate/utils/SafeTransferLib.sol";

```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Tray.sol#L6




#### <ins>Recommended Mitigation Steps</ins>
Add a contract exist control in functions that use solmate's `SafeTransferLib`

```solidity
pragma solidity >=0.8.0;

function isContract(address _addr) private returns (bool isContract) {
    isContract = _addr.code.length > 0;
}
```



## Non Critical Issues

### <a href="#Summary">[NC&#x2011;1]</a><a name="NC&#x2011;1"> Add a timelock to critical functions

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user). Consider adding a timelock to the following functions:

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








### <a href="#Summary">[NC&#x2011;2]</a><a name="NC&#x2011;2"> Constants in comparisons should appear on the left side

Doing so will prevent <a href="https://www.moserware.com/2008/01/constants-on-left-are-better-but-this.html">typo bugs</a>

#### <ins>Proof Of Concept</ins>

```solidity
264: if (tileData.fontClass == 7) {
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Tray.sol#L264

```solidity
114: if (_characterModifier == 1) {
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L114

```solidity
101: if (cidNFTID == 0 || addressRegistry.getAddress(cidNFTID) != ERC721(nftContract).ownerOf(nftID)) {
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-pfp-protocol/src/ProfilePicture.sol#L101






### <a href="#Summary">[NC&#x2011;3]</a><a name="NC&#x2011;3"> Critical Changes Should Use Two-step Procedure

The critical procedures should be two step process.

See similar findings in previous Code4rena contests for reference:
https://code4rena.com/reports/2022-06-illuminate/#2-critical-changes-should-use-two-step-procedure

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



#### <ins>Recommended Mitigation Steps</ins>

Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two step procedure on the critical functions.



### <a href="#Summary">[NC&#x2011;4]</a><a name="NC&#x2011;4"> Event emit should emit a parameter

Some emitted events do not have any emitted parameters. It is recommended to add some parameter such as state changes or value changes when events are emitted

#### <ins>Proof Of Concept</ins>


```solidity
228: emit PrelaunchEnded()

```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Tray.sol#L228



### <a href="#Summary">[NC&#x2011;5]</a><a name="NC&#x2011;5"> Use `delete` to Clear Variables

`delete a` assigns the initial value for the type to `a`. i.e. for integers it is equivalent to `a = 0`, but it can also be used on arrays, where it assigns a dynamic array of length zero or a static array of the same length with all elements reset. For structs, it assigns a struct with all members reset. Similarly, it can also be used to set an address to zero address. It has no effect on whole mappings though (as the keys of mappings may be arbitrary and are generally unknown). However, individual keys and what they map to can be deleted: If `a` is a mapping, then `delete a[x]` will delete the value stored at `x`.

The `delete` key better conveys the intention and is also more idiomatic. Consider replacing assignments of zero with `delete` statements.

#### <ins>Proof Of Concept</ins>

```solidity
93: bytesOffset = 0;

```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-bio-protocol/src/Bio.sol#L93



### <a href="#Summary">[NC&#x2011;6]</a><a name="NC&#x2011;6"> NatSpec return parameters should be included in contracts

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation. In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.

https://docs.soliditylang.org/en/v0.8.15/natspec-format.html
#### <ins>Proof Of Concept</ins>


All in-scope contracts


#### <ins>Recommended Mitigation Steps</ins>

Include return parameters in NatSpec comments

Recommendation Code Style: (from Uniswap3)

```solidity
    /// @notice Adds liquidity for the given recipient/tickLower/tickUpper position
    /// @dev The caller of this method receives a callback in the form of IUniswapV3MintCallback#uniswapV3MintCallback
    /// in which they must pay any token0 or token1 owed for the liquidity. The amount of token0/token1 due depends
    /// on tickLower, tickUpper, the amount of liquidity, and the current price.
    /// @param recipient The address for which the liquidity will be created
    /// @param tickLower The lower tick of the position in which to add liquidity
    /// @param tickUpper The upper tick of the position in which to add liquidity
    /// @param amount The amount of liquidity to mint
    /// @param data Any data that should be passed through to the callback
    /// @return amount0 The amount of token0 that was paid to mint the given amount of liquidity. Matches the value in the callback
    /// @return amount1 The amount of token1 that was paid to mint the given amount of liquidity. Matches the value in the callback
    function mint(
        address recipient,
        int24 tickLower,
        int24 tickUpper,
        uint128 amount,
        bytes calldata data
    ) external returns (uint256 amount0, uint256 amount1);
```



### <a href="#Summary">[NC&#x2011;7]</a><a name="NC&#x2011;7"> Initial value check is missing in Set Functions

A check regarding whether the current value and the new value are the same should be added

#### <ins>Proof Of Concept</ins>

```solidity
196: function changeNoteAddress(address _newNoteAddress) external onlyOwner {
        address currentNoteAddress = address(note);
        note = ERC20(_newNoteAddress);
        emit NoteAddressUpdate(currentNoteAddress, _newNoteAddress);
    }
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Namespace.sol#L196

```solidity
204: function changeRevenueAddress(address _newRevenueAddress) external onlyOwner {
        address currentRevenueAddress = revenueAddress;
        revenueAddress = _newRevenueAddress;
        emit RevenueAddressUpdated(currentRevenueAddress, _newRevenueAddress);
    }
}
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Namespace.sol#L204

```solidity
210: function changeNoteAddress(address _newNoteAddress) external onlyOwner {
        address currentNoteAddress = address(note);
        note = ERC20(_newNoteAddress);
        emit NoteAddressUpdate(currentNoteAddress, _newNoteAddress);
    }
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Tray.sol#L210

```solidity
218: function changeRevenueAddress(address _newRevenueAddress) external onlyOwner {
        address currentRevenueAddress = revenueAddress;
        revenueAddress = _newRevenueAddress;
        emit RevenueAddressUpdated(currentRevenueAddress, _newRevenueAddress);
    }
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Tray.sol#L218





### <a href="#Summary">[NC&#x2011;8]</a><a name="NC&#x2011;8"> Lines are too long

Usually lines in source code are limited to 80 characters. Today's screens are much larger so it's reasonable to stretch this in some cases. Since the files will most likely reside in GitHub, and GitHub starts using a scroll bar in all cases when the length is over 164 characters, the lines below should be split when they reach that length
Reference: https://docs.soliditylang.org/en/v0.8.10/style-guide.html#maximum-line-length

#### <ins>Proof Of Concept</ins>

```solidity
41: /// @dev Generates an on-chain SVG with a new line after 40 bytes. Line splitting generally supports UTF-8 multibyte characters and emojis, but is not tested for arbitrary UTF-8 characters
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-bio-protocol/src/Bio.sol#L41

```solidity
53: // Because we do not split on zero-width joiners, line in bytes can technically be much longer. Will be shortened to the needed length afterwards
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-bio-protocol/src/Bio.sol#L53

```solidity
72: // Note that we do not need to check i < lengthInBytes - 4, because we assume that it's a valid UTF8 string and these prefixes imply that another byte follows
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-bio-protocol/src/Bio.sol#L72

```solidity
98: //www.w3.org/2000/svg" preserveAspectRatio="xMinYMin meet" viewBox="0 0 400 100"><style>text { font-family: sans-serif; font-size: 12px; }</style>';
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-bio-protocol/src/Bio.sol#L98

```solidity
126: // Check for duplicate characters in the provided list. 1/2 * n^2 loop iterations, but n is bounded to 13 and we do not perform any storage operations
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Namespace.sol#L126

```solidity
62: /// @notice For generative fonts with randomness (Zalgo), we generate and fix this on minting. For some emojis, it can be set by the user to influence the skin color
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Tray.sol#L62

```solidity
33: /// @notice UTF-8 encoding of possible characters that can be over a letter (i.e., in the middle) for Zalgo. All characters have a length of 2 bytes
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L33

```solidity
39: /// @notice UTF-8 encoding of all supported emojis. They are sorted by their length in bytes (all with 3 bytes first, then all with four bytes, ...)
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L39

```solidity
45: /// @notice These constants are used to efficiently parse the UTF-8 encoding of the emojis. They enable constant time lookup of any character.
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L45

```solidity
47: /// Furthermore, we store the number of emojis where the skin tone can be modified such that we can revert when this is not supported (and would result in wrong characters).
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L47

```solidity
69: /// @notice Convert a given font class, character index, and a seed (for font classes with randomness) to their Unicode representation as bytes
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L69

```solidity
140: // We do not reuse the same seed for the following generations to avoid any symmetries, e.g. that 2 chars above would also always result in 2 chars below
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L140

```solidity
195: // Hex encoding: CEB1 E182A6 C688 D483 D2BD CF9D C9A0 D48B CEB9 CA9D C699 CA85 C9B1 C9B3 CF83 CF81 CF99 C9BE CA82 C69A CF85 CA8B C9AF 78 E183A7 C8A5
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L195

```solidity
247: //www.w3.org/2000/svg" preserveAspectRatio="xMinYMin meet" viewBox="0 0 400 200"><style>text { font-family: sans-serif; font-size: 30px; }</style>',
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L247

```solidity
56: /// @param _subprotocolName Name with which the subprotocol is / will be registered in the registry. Registration will not be performed automatically
```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-pfp-protocol/src/ProfilePicture.sol#L56





### <a href="#Summary">[NC&#x2011;9]</a><a name="NC&#x2011;9"> Implementation contract may not be initialized

OpenZeppelin recommends that the initializer modifier be applied to constructors. 
Per OZs Post implementation contract should be initialized to avoid potential griefs or exploits.
https://forum.openzeppelin.com/t/uupsupgradeable-vulnerability-post-mortem/15680/5

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




### <a href="#Summary">[NC&#x2011;10]</a><a name="NC&#x2011;10"> Non-usage of specific imports

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



### <a href="#Summary">[NC&#x2011;11]</a><a name="NC&#x2011;11"> Use a more recent version of Solidity

<a href="https://blog.soliditylang.org/2021/04/21/solidity-0.8.4-release-announcement/">0.8.4</a>:
bytes.concat() instead of abi.encodePacked(<bytes>,<bytes>)

<a href="https://blog.soliditylang.org/2022/02/16/solidity-0.8.12-release-announcement/">0.8.12</a>: 
string.concat() instead of abi.encodePacked(<str>,<str>)

<a href="https://blog.soliditylang.org/2022/03/16/solidity-0.8.13-release-announcement/">0.8.13</a>: 
Ability to use using for with a list of free functions

<a href="https://blog.soliditylang.org/2022/05/18/solidity-0.8.14-release-announcement/">0.8.14</a>:

ABI Encoder: When ABI-encoding values from calldata that contain nested arrays, correctly validate the nested array length against calldatasize() in all cases.
Override Checker: Allow changing data location for parameters only when overriding external functions.

<a href="https://blog.soliditylang.org/2022/06/15/solidity-0.8.15-release-announcement/">0.8.15</a>:

Code Generation: Avoid writing dirty bytes to storage when copying bytes arrays.
Yul Optimizer: Keep all memory side-effects of inline assembly blocks.

<a href="https://blog.soliditylang.org/2022/08/08/solidity-0.8.16-release-announcement/">0.8.16</a>:

Code Generation: Fix data corruption that affected ABI-encoding of calldata values represented by tuples: structs at any nesting level; argument lists of external functions, events and errors; return value lists of external functions. The 32 leading bytes of the first dynamically-encoded value in the tuple would get zeroed when the last component contained a statically-encoded array.

<a href="https://blog.soliditylang.org/2022/09/08/solidity-0.8.17-release-announcement/">0.8.17</a>:

Yul Optimizer: Prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call.

<a href="https://blog.soliditylang.org/2023/02/22/solidity-0.8.19-release-announcement/">0.8.19</a>:
SMTChecker: New trusted mode that assumes that any compile-time available code is the actual used code, even in external calls. 
Bug Fixes: 
- Assembler: Avoid duplicating subassembly bytecode where possible.
- Code Generator: Avoid including references to the deployed label of referenced functions if they are called right away.
- ContractLevelChecker: Properly distinguish the case of missing base constructor arguments from having an unimplemented base function.
- SMTChecker: Fix internal error caused by unhandled z3 expressions that come from the solver when bitwise operators are used.
- SMTChecker: Fix internal error when using the custom NatSpec annotation to abstract free functions.
- TypeChecker: Also allow external library functions in using for.

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



#### <ins>Recommended Mitigation Steps</ins>

Consider updating to a more recent solidity version.





### <a href="#Summary">[NC&#x2011;12]</a><a name="NC&#x2011;12"> Use `bytes.concat()`

Solidity version 0.8.4 introduces `bytes.concat()` (vs `abi.encodePacked(<bytes>,<bytes>)`)

#### <ins>Proof Of Concept</ins>


```solidity
116: return string(abi.encodePacked("data:application/json;base64,", json))

```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-bio-protocol/src/Bio.sol#L116

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
105: return string(abi.encodePacked("data:application/json;base64,", json))

```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Namespace.sol#L95

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Namespace.sol#L105



```solidity
135: abi.encodePacked(
                        '{"name": "Tray #',
                        LibString.toString(_id),
                        '", "image": "data:image/svg+xml
145: return string(abi.encodePacked("data:application/json;base64,", json))

```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Tray.sol#L135

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Tray.sol#L145



```solidity
104: bytes memory character = abi.encodePacked(
                EMOJIS[byteOffset],
                EMOJIS[byteOffset + 1],
                EMOJIS[byteOffset + 2]
            )
110: character = abi.encodePacked(character, EMOJIS[byteOffset + i])
115: character = abi.encodePacked(character, hex"F09F8FBB")
117: character = abi.encodePacked(character, hex"F09F8FBC")
119: character = abi.encodePacked(character, hex"F09F8FBD")
121: character = abi.encodePacked(character, hex"F09F8FBE")
123: character = abi.encodePacked(character, hex"F09F8FBF")
135: return abi.encodePacked(bytes1(asciiStartingIndex + uint8(_characterIndex)))
145: bytes memory character = abi.encodePacked(bytes1(asciiStartingIndex + uint8(_characterIndex)))
149: character = abi.encodePacked(
                    character,
                    ZALGO_ABOVE_LETTER[characterIndex],
                    ZALGO_ABOVE_LETTER[characterIndex + 1]
                )
158: character = abi.encodePacked(
                    character,
                    ZALGO_OVER_LETTER[characterIndex],
                    ZALGO_OVER_LETTER[characterIndex + 1]
                )
167: character = abi.encodePacked(
                    character,
                    ZALGO_BELOW_LETTER[characterIndex],
                    ZALGO_BELOW_LETTER[characterIndex + 1]
                )
197: return abi.encodePacked(FONT_SQUIGGLE[0], FONT_SQUIGGLE[1])
199: return abi.encodePacked(FONT_SQUIGGLE[2], FONT_SQUIGGLE[3], FONT_SQUIGGLE[4])
202: return abi.encodePacked(FONT_SQUIGGLE[5 + offset], FONT_SQUIGGLE[6 + offset])
204: return abi.encodePacked(FONT_SQUIGGLE[47])
206: return abi.encodePacked(FONT_SQUIGGLE[48], FONT_SQUIGGLE[49], FONT_SQUIGGLE[50])

```

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L104

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L110

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L115

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L117

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L119

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L121

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L123

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L135

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L145

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L149

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L158

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L167

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L197

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L199

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L202

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L204

https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol/src/Utils.sol#L206




#### <ins>Recommended Mitigation Steps</ins>

Use `bytes.concat()` and upgrade to at least Solidity version 0.8.4 if required. 



