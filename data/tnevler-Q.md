# Report
## Low Risk ##
### [L-1]: Solmate’s SafeTransferLib doesn’t check if the ERC20 contract exists
**Context:**

1. ```SafeTransferLib.safeTransferFrom(note, msg.sender, revenueAddress, _amount * trayPrice);``` [L157](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L157) 

**Description:**

The following code will not revert in case the token doesn’t exist.
```/// @dev Note that none of the functions in this library check that a token has code at all! That responsibility is delegated to the caller.``` [L9](https://github.com/transmissions11/solmate/blob/main/src/utils/SafeTransferLib.sol#L9)

**Recommendation:**

Add isContract() function:
```
function isContract(address _addr) private returns (bool isContract) {
    isContract = _addr.code.length > 0;
}
```
### [L-2]: Missing checks for address(0x0)
**Context:**

1. ```cidNFT = ICidNFT(_cidNFT);``` [L58](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L58) 
1. ```subprotocolName = _subprotocolName;``` [L59](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L59) 

**Recommendation:**

Add non-zero address checks when set address state variables.

### [L-3]: transferOwnership should be two step process
**Context:**

1. ```import {Owned} from "solmate/auth/Owned.sol";``` [L5](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L5) 
1. ```import {Owned} from "solmate/auth/Owned.sol";``` [L7](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L7) 

**Description:**

"solmate/auth/Owned.sol" implements transferOwnership function. Lack of two-step procedure for transfer of ownership makes it error-prone. It’s possible that the owner mistakenly transfers ownership to the uncontrolled account and it will break all functions with onlyOwner() modifier.

**Recommendation:**

Implement a two step process for transfer of ownership. Owner call transferOwnership()  function where nominates an account. Nominated account needs to call an acceptOwnership() function for the transfer of ownership to fully succeed. This will confirm that the nominated EOA account is a valid and active account.

### [L-4]: Use Checks Effects Interactions pattern
**Context:**

[L80-L82](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L80-L82)

**Description:**
[Checks Effects Interactions pattern](https://fravoll.github.io/solidity-patterns/checks_effects_interactions.html) reduce the attack surface for malicious contracts trying to hijack control flow after an external call.

**Recommendation:**
Change to:
```
        if (ERC721(_nftContract).ownerOf(_nftID) != msg.sender)
            revert PFPNotOwnedByCaller(msg.sender, _nftContract, _nftID);
        uint256 tokenId = ++numMinted;
```

### [L-5]: Critical changes should use two-step procedure
**Context:**

1. ```note = ERC20(_newNoteAddress);``` [L198](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L198) 
1. ```revenueAddress = _newRevenueAddress;``` [L206](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L206) 
1. ```note = ERC20(_newNoteAddress);``` [L212](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L212) 
1. ```revenueAddress = _newRevenueAddress;``` [L220](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L220) 

**Recommendation:**

The best practice is to use two-step procedure for critical changes to make them less error-prone. 

## Non-Critical Issues ##
### [N-1]: Commented code
**Context:**

```// uint256 constant EMOJIS_LE_FOURTEEN_BYTES = 420;``` [L55](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L55) 

### [N-2]: trayPrice validation can be added
**Context:**
```trayPrice = _trayPrice;```[L106](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L106)

**Recommendation:**

Define the minimum and maximum allowable price per tray to prevent setting the wrong trayPrice value, and validate it in constructor.

### [N-3]: Follow Solidity standard naming conventions
**Context:**

1. ```ICidNFT private immutable cidNFT;``` [L14](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L14) (cidNFT should start with _)
1. ```mapping(uint256 => ProfilePictureData) private pfp;``` [L32](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L32) (pfp should start with _)
1. ```mapping(uint256 => Tray.TileData[]) private nftCharacters;``` [L49](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L49) (nftCharacters should start with _)
1. ```uint256 private constant TILES_PER_TRAY = 7;``` [L19](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L19) (TILES_PER_TRAY should start with _)
1. ```uint256 private constant SUM_ODDS = 109;``` [L22](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L22) (SUM_ODDS should start with _)
1. ```uint256 private constant NUM_CHARS_EMOJIS = 420;``` [L25](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L25) (NUM_CHARS_EMOJIS should start with _)
1. ```uint256 private constant NUM_CHARS_LETTERS = 26;``` [L28](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L28) (NUM_CHARS_LETTERS should start with _)
1. ```uint256 private constant NUM_CHARS_LETTERS_NUMBERS = 36;``` [L31](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L31) (NUM_CHARS_LETTERS_NUMBERS should start with _)
1. ```uint256 private constant PRE_LAUNCH_MINT_CAP = 1_000;``` [L34](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L34) (PRE_LAUNCH_MINT_CAP should start with _)
1. ```address private revenueAddress;``` [L44](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L44) (revenueAddress should start with _)
1. ```mapping(uint256 => TileData[TILES_PER_TRAY]) private tiles;``` [L67](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L67) (tiles should start with _)
1. ```uint256 private prelaunchMinted = type(uint256).max;``` [L73](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L73) (prelaunchMinted should start with _)
1. ```bytes private constant FONT_SQUIGGLE =``` [L16](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L16) (FONT_SQUIGGLE should start with _)
1. ```bytes private constant ZALGO_ABOVE_LETTER =``` [L20](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L20) (ZALGO_ABOVE_LETTER should start with _)
1. ```uint256 private constant ZALGO_NUM_ABOVE = 46;``` [L24](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L24) (ZALGO_NUM_ABOVE should start with _)
1. ```bytes private constant ZALGO_BELOW_LETTER =``` [L27](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L27) (ZALGO_BELOW_LETTER should start with _)
1. ```uint256 private constant ZALGO_NUM_BELOW = 47;``` [L31](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L31) (ZALGO_NUM_BELOW should start with _)
1. ```bytes private constant ZALGO_OVER_LETTER = hex"CCB4CCB5CCB6CCB7CCB8";``` [L34](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L34) (ZALGO_OVER_LETTER should start with _)
1. ```uint256 private constant ZALGO_NUM_OVER = 5;``` [L37](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L37) (ZALGO_NUM_OVER should start with _)
1. ```bytes private constant EMOJIS =``` [L42](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L42) (EMOJIS should start with _)
1. ```uint256 private constant EMOJIS_LE_THREE_BYTES = 17;``` [L50](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L50) (EMOJIS_LE_THREE_BYTES should start with _)
1. ```uint256 private constant EMOJIS_LE_FOUR_BYTES = 383;``` [L51](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L51) (EMOJIS_LE_FOUR_BYTES should start with _)
1. ```uint256 private constant EMOJIS_LE_SIX_BYTES = 409;``` [L52](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L52) (EMOJIS_LE_SIX_BYTES should start with _)
1. ```uint256 private constant EMOJIS_LE_SEVEN_BYTES = 416;``` [L53](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L53) (EMOJIS_LE_SEVEN_BYTES should start with _)
1. ```uint256 private constant EMOJIS_LE_EIGHT_BYTES = 419;``` [L54](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L54) (EMOJIS_LE_EIGHT_BYTES should start with _)
1. ```uint256 private constant EMOJIS_MOD_SKIN_TONE_THREE_BYTES = 2;``` [L58](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L58) (EMOJIS_MOD_SKIN_TONE_THREE_BYTES should start with _)
1. ```uint256 private constant EMOJIS_MOD_SKIN_TONE_FOUR_BYTES = 47;``` [L59](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L59) (EMOJIS_MOD_SKIN_TONE_FOUR_BYTES should start with _)
1. ```uint256 private constant EMOJIS_BYTE_OFFSET_FOUR_BYTES = 51; // 17 * 3``` [L63](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L63) (EMOJIS_BYTE_OFFSET_FOUR_BYTES should start with _)
1. ```uint256 private constant EMOJIS_BYTE_OFFSET_SIX_BYTES = 1515; // 17 * 3 + 366 * 4``` [L64](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L64) (EMOJIS_BYTE_OFFSET_SIX_BYTES should start with _)
1. ```uint256 private constant EMOJIS_BYTE_OFFSET_SEVEN_BYTES = 1671; // 17 * 3 + 366 * 4 + 26 * 6``` [L65](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L65) (EMOJIS_BYTE_OFFSET_SEVEN_BYTES should start with _)
1. ```uint256 private constant EMOJIS_BYTE_OFFSET_EIGHT_BYTES = 1720; // 17 * 3 + 366 * 4 + 26 * 6 + 7 * 7``` [L66](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L66) (EMOJIS_BYTE_OFFSET_EIGHT_BYTES should start with _)
1. ```uint256 private constant EMOJIS_BYTE_OFFSET_FOURTEEN_BYTES = 1744; // 17 * 3 + 366 * 4 + 26 * 6 + 7 * 7 + 3 * 8``` [L67](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L67) (EMOJIS_BYTE_OFFSET_FOURTEEN_BYTES should start with _)

**Description:**

The above codes don't follow [Solidity's standard naming convention](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#naming-conventions).  
- Internal and private functions should use mixedCase format starting with an underscore.
- Structs should be named using the CapWords style. Examples: MyCoin, Position, PositionXY.
- Events should be named using the CapWords style. Examples: Deposit, Transfer, Approval, BeforeTransfer, AfterTransfer.
- Functions should use mixedCase. Examples: getBalance, transfer, verifyOwner, addMember, changeOwner.
- Function arguments should use mixedCase. Examples: initialSupply, account, recipientAddress, senderAddress, newOwner.
- Local and State Variable should use mixedCase. Examples: totalSupply, remainingSupply, balancesOf, creatorAddress, isPreSale, tokenExchangeRate.
- Constants should be named with all capital letters with underscores separating words. Examples: MAX_BLOCKS, TOKEN_NAME, TOKEN_TICKER, CONTRACT_VERSION.
- Modifier should use mixedCase. Examples: onlyBy, onlyAfter, onlyDuringThePreSale.
- Enums, in the style of simple type declarations, should be named using the CapWords style. Examples: TokenGroup, Frame, HashStyle, CharacterLocation.

### [N-4]: NatSpec is missing
**Context:**

1. ```function _beforeTokenTransfers(``` [L231](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L231) 
1. ```function _drawing(uint256 _seed) private pure returns (TileData memory tileData) {``` [L245](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L245) 

### [N-5]: Line is too long
**Context:**

1. ```/// @dev Generates an on-chain SVG with a new line after 40 bytes. Line splitting generally supports UTF-8 multibyte characters and emojis, but is not tested for arbitrary UTF-8 characters``` [L41](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L41) 
1. ```// Because we do not split on zero-width joiners, line in bytes can technically be much longer. Will be shortened to the needed length afterwards``` [L53](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L53) 
1. ```bytes memory bName = new bytes(numCharacters * 33); // Used to convert into a string. Can be 33 times longer than the string at most (longest zalgo characters is 33 bytes)``` [L117](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L117) 
1. ```uint256 charRandValue = Utils.iteratePRNG(_seed); // Iterate PRNG to not have any biasedness / correlation between random numbers``` [L247](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L247) 

**Description:**

Maximum suggested line length is [120 characters](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-length).
