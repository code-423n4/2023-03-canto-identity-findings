# QA Report for Canto Identity Subprotocols contest
## Overview
During the audit, 8 low and 9 non-critical issues were found.

№ | Title | Risk Rating  | Instance Count
--- | --- | --- | ---
L-1 | Docs may mislead users | Low | 1
L-2 | Return the amount available for minting | Low | 1
L-3 | Add comment about _safeMint | Low | 2
L-4 | Use the two-step-transfer of ownership | Low | 2
L-5 | Critical Address Changes Should Use Two-step Procedure | Low | 4
L-6 | Use Checks Effects Interactions pattern | Low | 1
L-7 | Malicious token contracts can pass the ownership check | Low | 2
L-8 | Might need to change mint() function in ProfilePicture.sol | Low | 1
NC-1 | Use gender-neutral pronouns | Non-Critical | 2
NC-2 | Validate tray price | Non-Critical | 1
NC-3 | Order of Functions | Non-Critical | 4
NC-4 | Order of Layout | Non-Critical | 3
NC-5 | Missing NatSpec | Non-Critical | 2
NC-6 | Commented code | Non-Critical | 1
NC-7 | Inconsistency when using uint and uint256 | Non-Critical | 5
NC-8 | Maximum line length exceeded | Non-Critical | 9
NC-9 | Missing leading underscores | Non-Critical | 13

## Low Risk Findings(8)
### L-1. Docs may mislead users
##### Description
Docs says: ["A fusing fee that is proportial to the length of the name is charged"](https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol#fusing). Although it seems obvious that a shorter name is more unique and therefore more expensive. Reading the protocol documentation, the user may think that the longer the name, the higher the price and, therefore, they may choose a shorter name and pay more.
##### Instances
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L113
- https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol#fusing

##### Recommendation
Change to "A fusing fee that is inversely proportional to the length of the name is charged" or "A fusing fee that is proportional to the length of the name is charged. The shorter the name, the higher the fee".
#
### L-2. Return the amount available for minting
##### Description
For convenience, the amount available for minting should be returned in the error message.
##### Instances
- [```if (startingTrayId + _amount - 1 > PRE_LAUNCH_MINT_CAP) revert MintExceedsPreLaunchAmount();```](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L155)

##### Recommendation
Change to:
```
if (startingTrayId + _amount - 1 > PRE_LAUNCH_MINT_CAP) revert MintExceedsPreLaunchAmount(PRE_LAUNCH_MINT_CAP - startingTrayId - 1);
```
#
### L-3. Add comment about _safeMint
##### Description
If the [comment in the Tray.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L167) is also valid for [ProfilePicture.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L86), then duplicate the comment in the ProfilePicture.sol. Otherwise, _safeMint can be used.
##### Instances
- [```_mint(msg.sender, tokenId);```](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L86) 
- [```_mint(msg.sender, _amount); // We do not use _safeMint on purpose here to disallow callbacks and save gas```](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L167) 

#
### L-4. Use the two-step-transfer of ownership
##### Description
"{Owned} from "solmate/auth/Owned.sol" has one-step transfer of ownership. If the owner accidentally transfers ownership to an incorrect address, protected functions may become permanently inaccessible.
##### Instances
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L5
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L7

##### Recommendation
Consider using a two-step-transfer of ownership: the current owner would nominate a new owner, and to become the new owner, the nominated account would have to approve the change, so that the address is proven to be valid.
#
### L-5. Critical Address Changes Should Use Two-step Procedure
##### Description
Lack of two-step procedure for critical operations leaves them error-prone.
##### Instances
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L196
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L204
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L210
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L218

##### Recommendation
Consider adding two step procedure on the critical functions.
#
### L-6. Use Checks Effects Interactions pattern
##### Description
For reducing the attack surface for malicious contracts trying to hijack control flow after an external call, it is best practice to use [this pattern](https://fravoll.github.io/solidity-patterns/checks_effects_interactions.html).
##### Instances
- [```Line80-82```](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L80-L82):
```
        uint256 tokenId = ++numMinted;
        if (ERC721(_nftContract).ownerOf(_nftID) != msg.sender)
            revert PFPNotOwnedByCaller(msg.sender, _nftContract, _nftID);
```

##### Recommendation
Change to:
```
        if (ERC721(_nftContract).ownerOf(_nftID) != msg.sender)
            revert PFPNotOwnedByCaller(msg.sender, _nftContract, _nftID);
        uint256 tokenId = ++numMinted;
```
#
### L-7. Malicious token contracts can pass the ownership check
##### Description
Verification that the user owns NFT can be passed using malicious token contracts, this can negatively affect the operation of the protocol.

##### Instances
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L81-L82
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L101

#
### L-8. Might need to change mint() function in ProfilePicture.sol
##### Description
The user may mint many PFP NFT with the same reference ```_nftContract``` and ```_nftID```.
##### Instances
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L79-L88

##### Recommendation
If it is not intended that the user can mint multiple PFP NFTs using the same reference NFT, then it is necessary to keep track of which NFTs the user has already used.

#
## Non-Critical Risk Findings(9)
### NC-1. Use gender-neutral pronouns
##### Instances
- [```Any user can mint a Bio NFT by calling Bio.mint and passing his biography. ```](https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-bio-protocol#minting)
- [```A user can therefore precompute which trays he will get.```](https://github.com/code-423n4/2023-03-canto-identity/tree/main/canto-namespace-protocol#tray)

##### Recommendation
Change to:
- ```Any user can mint a Bio NFT by calling Bio.mint and passing their biography. ```
- ```A user can therefore precompute which trays they will get.```

#
### NC-2. Validate tray price
##### Description
Price validation can be added.
##### Instances
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L106

##### Recommendation
Consider setting a minimum and maximum allowable price per tray and check if the function parameter matches it.
#
### NC-3. Order of Functions
##### Description
According to [Style Guide](https://docs.soliditylang.org/en/v0.8.16/style-guide.html#order-of-functions), ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier.  
Functions should be grouped according to their visibility and ordered:
1) constructor
2) receive function (if exists)
3) fallback function (if exists)
4) external
5) public
6) internal
7) private
##### Instances
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L70
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L43
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L90
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L119

##### Recommendation
Public functions should be placed after external.
#
### NC-4. Order of Layout
##### Description
According to [Order of Layout](https://docs.soliditylang.org/en/v0.8.16/style-guide.html#order-of-layout), inside each contract, library or interface, use the following order:
1) Type declarations
2) State variables
3) Events
4) Modifiers
5) Functions
##### Instances
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L21
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L30
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L57

##### Recommendation
Place structs before state variables.
#
### NC-5. Missing NatSpec
##### Description
NatSpec is missing for 2 functions in 1 contract.
##### Instances
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L231
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L245

##### Recommendation
Add NatSpec for all functions.
#
### NC-6. Commented code
##### Instances
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L55

##### Recommendation
Delete commented code.
#
### NC-7. Inconsistency when using uint and uint256
##### Description
Some variables is declared as ```uint``` and some as ```uint256```.
Below 5 instances when ```uint``` is used, while ```uint256``` is used in all other 110 instances.
##### Instances
- [```uint lengthInBytes = bioTextBytes.length;```](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L47)
- [```uint lines = (lengthInBytes - 1) / 40 + 1;```](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L49) 
- [```uint bytesOffset;```](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L55) 
- [```for (uint i; i < lengthInBytes; ++i) {```](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L56) 
- [```for (uint i; i < lines; ++i) {```](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L100) 

##### Recommendation
Stick to one style.
#
### NC-8. Maximum line length exceeded
##### Description
According to [Style Guide](https://docs.soliditylang.org/en/v0.8.16/style-guide.html#maximum-line-length), maximum suggested line length is 120 characters.  Longer lines make the code harder to read.
##### Instances
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L56
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L41
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L53
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L72
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L35
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L126
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L152
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L62
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L247

##### Recommendation
Make the lines shorter.
#
### NC-9. Missing leading underscores
##### Description
Private constants, immutables, and state variables should have a leading underscore.
##### Instances
- [```ICidNFT private immutable cidNFT;```](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L14) 
- [```mapping(uint256 => ProfilePictureData) private pfp;```](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L32) 
- [```address private revenueAddress;```](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L23) 
- [```mapping(uint256 => Tray.TileData[]) private nftCharacters;```](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L49) 
- [```uint256 private constant TILES_PER_TRAY = 7;```](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L19) 
- [```uint256 private constant SUM_ODDS = 109;```](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L22) 
- [```uint256 private constant NUM_CHARS_EMOJIS = 420;```](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L25) 
- [```uint256 private constant NUM_CHARS_LETTERS = 26;```](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L28) 
- [```uint256 private constant NUM_CHARS_LETTERS_NUMBERS = 36;```](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L31) 
- [```uint256 private constant PRE_LAUNCH_MINT_CAP = 1_000;```](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L34) 
- [```address private revenueAddress;```](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L44) 
- [```mapping(uint256 => TileData[TILES_PER_TRAY]) private tiles;```](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L67) 
- [```uint256 private prelaunchMinted = type(uint256).max;```](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L73) 

##### Recommendation
Add leading underscores where needed.