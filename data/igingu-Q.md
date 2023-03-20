# [L-01] ProfilePicture: Misleading comments and Error names

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L69 also reverts if ProfilePicture.sol NFT with ```_id``` is not linked to any CIDNFT, because of [if (cidNFTID == 0)](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L101). Error name should also change, or be split into two, one error for ProfilePicture NFT not linked, another error for ```nftContract.nftID``` NFT no longer in posession of associated CIDNFT owner.

# [L-02] Literal values should be stored in constants, to greatly improve whole codebase's readability

The whole codebases uses a lot of literal values (especially ```Namespace``` protocol). These should be stored as constant values in the contracts (which don't incur additional gas cost), in order to make the code more self-explanatory => comments explaining the values could be removed, reducing maintenance time in the future.

## Recommendation:
```
canto-bio-protocol/src/Bio.sol
    uint256 private constant CANTO_MAINNET_BLOCKID = 7700;
    uint256 private constant MAX_BIO_LENGTH = 200;
    uint256 private constant MAX_NUM_CHARS_BEFORE_NEWLINE = 40;
    string private constant HTML_BEGINNING = "<text x="50%" y="50%" dominant-baseline="middle" text-anchor="middle">";

    // would also save gas, instead of creating the in-memory variable ```string memory svg```
    string private constant SVG_HTML = "<svg xmlns="http://www.w3.org/2000/svg" preserveAspectRatio="xMinYMin meet"; viewBox="0 0 400 100"><style>text { font-family: sans-serif; font-size: 12px; }</style>";

canto-namespace-protocol/src/Utils.sol
    uint256 private constant ASCII_STARTING_INDEX = 97;
    uint256 private constant ITERATE_PRNG_MULTIPLIER = 15485863;
    uint256 private constant ITERATE_PRNG_MODULO = 2038074743;

canto-namespace-protocol/src/Namespace.sol
    uint256 private constant MAX_BYTES_PER_CHARACTER = 33;
    uint256 private constant FONT_CLASS_EMOJIS = 0;
```

# [L-03] Bio: ```Bio.tokenURI``` function is very hard to understand and has redundant check

```Bio.tokenURI``` should have more spacing and comments, explaining what each code block does.

In ```(i > 0 && (i + 1) % 40 == 0)```, the ```i > 0``` check is redundant since ```i``` will always be ```> 0``` if ```(i + 1) % 40 == 0``` and ```i``` is an ```unsigned integer```.

## Recommendation:
Refactor function as:
```
    function tokenURI2(uint256 _id) public view override returns (string memory) {
        if (_ownerOf[_id] == address(0)) revert TokenNotMinted(_id);

        string storage bioText = bio[_id];
        bytes memory bioTextBytes = bytes(bioText);
        uint lengthInBytes = bioTextBytes.length;

        // Insert a new line after 40 characters, taking into account unicode character
        // Note: for 39 characters, need 1 newline (after all 39)
        //       for 40 characters, need 1 newline (after all 40)
        //       for 41 characters, need 2 newlines (after first 40, and after last 1)
        uint lines = (lengthInBytes - 1) / 40 + 1;
        string[] memory strLines = new string[](lines);

        bool prevByteWasContinuation;
        uint256 insertedLines;
        // Because we do not split on zero-width joiners, line in bytes can technically be much longer. Will be shortened to the needed length afterwards
        bytes memory bytesLines = new bytes(80);
        uint bytesOffset;

        for (uint i; i < lengthInBytes - 1; ++i) {
            // put current character in resulting array
            bytes1 character = bioTextBytes[i];
            bytesLines[bytesOffset] = character;
            bytesOffset++;
            
            // Note - if am at last character before newline should come,
            //        or prevByteWasContinuation,
            //        or I reached the last character,
            //        should potentially insert line break
            if ((i > 0 && (i + 1) % 40 == 0) || prevByteWasContinuation || i == lengthInBytes - 1) {
                bytes1 nextCharacter;
                if (i != lengthInBytes - 1) {
                    nextCharacter = bioTextBytes[i + 1];
                }
                if (nextCharacter & 0xC0 == 0x80) {
                    // Unicode continuation byte, top two bits are 10
                    // Note - https://stackoverflow.com/questions/9356169/utf-8-continuation-bytes
                    prevByteWasContinuation = true;
                } else {
                    // Do not split when the prev. or next character is a zero width joiner. Otherwise, 👨‍👧‍👦 could become 👨>‍👧‍👦
                    // Furthermore, do not split when next character is skin tone modifier to avoid 🤦‍♂️\n🏻
                    if (
                        // Note that we do not need to check i < lengthInBytes - 4, because we assume that it's a valid UTF8 string and these prefixes imply that another byte follows
                        // Note - check if next character is beginning of Zero Width Joiner 
                        //         https://emojipedia.org/zero-width-joiner/#:~:text=Zero%20Width%20Joiner%20(ZWJ)%20is,invisible%20character%20when%20used%20alone.
                        (nextCharacter == 0xE2 && bioTextBytes[i + 2] == 0x80 && bioTextBytes[i + 3] == 0x8D) ||
                        // Note - check if next character is a skin modifier
                        (nextCharacter == 0xF0 &&
                            bioTextBytes[i + 2] == 0x9F &&
                            bioTextBytes[i + 3] == 0x8F &&
                            uint8(bioTextBytes[i + 4]) >= 187 &&
                            uint8(bioTextBytes[i + 4]) <= 191) ||
                        // Note - check for ending of Zero Width Joiner 
                        //        https://emojipedia.org/zero-width-joiner/#:~:text=Zero%20Width%20Joiner%20(ZWJ)%20is,invisible%20character%20when%20used%20alone.
                        (i >= 2 &&
                            bioTextBytes[i - 2] == 0xE2 &&
                            bioTextBytes[i - 1] == 0x80 &&
                            bioTextBytes[i] == 0x8D)
                    ) {
                        prevByteWasContinuation = true;
                    } else {
                        // Note - Trim down bytesLines to correct length, by changing length directly
                        assembly {
                            mstore(bytesLines, bytesOffset)
                        }

                        // Note - We have our newline
                        strLines[insertedLines++] = string(bytesLines);

                        // Note - Reset running variables
                        bytesLines = new bytes(80);
                        prevByteWasContinuation = false;
                        bytesOffset = 0;
                    }
                }
            }
        }
        
        string memory text = HTML_BEGINNING;
        for (uint i; i < lines; ++i) {
            text = string.concat(text, '<tspan x="50%" dy="20">', strLines[i], "</tspan>");
        }
        string memory json = Base64.encode(
            bytes(
                string.concat(
                    '{"name": "Bio #',
                    LibString.toString(_id),
                    '", "description": "',
                    bioText,
                    '", "image": "data:image/svg+xml;base64,',
                    Base64.encode(bytes(string.concat(SVG_HTML, text, "</text></svg>"))),
                    '"}'
                )
            )
        );
        return string(abi.encodePacked("data:application/json;base64,", json));
    }
```

# [L-04] Namespace: Consider introducing enums for font classes, to navigate the code more easily, remove dead comments and avoid future bugs

By naming variables better and making the code more "self-explainable", one not only makes it easier to read, but also minimizes future maintainence efforts to update comments which were meant to explain the code ("dead comments").

```
enum FontClass { EMOJI, BASIC, SCRIPT, SCRIPT_BOLD, OLDE, OLDE_BOLD, SQUIGGLE, ZALGO, BLOCKS, BLOCKS_INVERTED };

if (_fontClass == 0) {
// Emojis
uint256 byteOffset;
->
if (_fontClass == FontClass.EMOJI) {
uint256 byteOffset;

} else if (_fontClass == 7) {
    // Zalgo
    uint8 asciiStartingIndex = 97;
->
} else if (_fontClass == FontClass.ZALGO) {
    uint8 asciiStartingIndex = 97;
```

# [L-05] Namespace: Accessing out-of-bounds array in ```getTile``` should revert with specific error message

Trying to fuse a tile with ```_tileOffset >= TILES_PER_TRAY``` would cause a Solidity VM Panic code, but it should be gracefully handled and a suggestive error should be thrown instead.

## Recommendation
```
    function getTile(uint256 _trayId, uint8 _tileOffset) external view returns (TileData memory tileData) {
        if (!_exists(_trayId)) revert TrayNotMinted(_trayId);
        tileData = tiles[_trayId][_tileOffset];
    }
    ->
    function getTile(uint256 _trayId, uint8 _tileOffset) external view returns (TileData memory tileData) {
        if (!_exists(_trayId)) revert TrayNotMinted(_trayId);
        if (_tileOffset >= TILES_PER_TRAY) revert TileOffsetShouldBeSmallerThanLimit(_trayId, _tileOffset);
        tileData = tiles[_trayId][_tileOffset];
    }
```

# [L-06] Namespace: Adoption might be impacted by how fast fusing prices increase, based on number of fused tiles

[The cost of fusing tiles into your Namespace NFT](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L113) doubles, for each character less that you introduce => exclusivity comes from having a smaller name, not a longer one. 

The doubling seems like a very steep price increase:
* 13 characters is 1 NOTE
* 12 characters is 2 NOTE
...
* 10 characters is already 8 NOTE which is ~ 8$
...
* 3 character is 1024 NOTE
...
* 1 character is 4096 NOTE

## Recommendation:
Come up with a different formula for computing fusing prices, which doesn't double every step.

# [L-07] Namespace: The initial seed for minting trays is set by deployer
Ideally deployer should have less control ofer the ```_initHash```, I don't see why they would have to set it themselves and not take it from something like ```block.timestamp```, which wouldn't be that much in the deployer's control.

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L105

## Recommendation:
Initialize ```lastHash``` with ```block.timestamp```, converted to ```bytes32```.

# [N-01] For modern and more readable code; update import usages

By importing whole Solidity files, one can end up importing more than needed, which increases gas costs and also doesn't easily highlight the contracts/interfaces/libraries one actually needs.

All source files contain such imports, and need changing.

## Recommendation:
```solidity
canto-pfp-protocol/src/ProfilePicture.sol:5:
    import "src/interfaces/Turnstile.sol";
    ->
    import {Turnstile} from "src/interfaces/Turnstile.sol";
```

# [N-02] ProfilePicture: Important state variables like CIDNFT address should be public

In Canto Identity, subprotocols are tightly coupled with the CIDNFT address used, which means one should be able to query the CIDNFT address easily, especially since there is no reason to "hide" it.

## Recommendation:
```
canto-pfp-protocol/src/ProfilePicture.sol:14:
    ICidNFT private immutable cidNFT;
    ->
    ICidNFT public immutable cidNFT;
```

# [N-03] Solidity Code Style conventions

## [Function arguments should use mixedCase](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#function-argument-names)

All source files contain function arguments starting with ```_``` instead of using mixedCase, and need changing.

---

## [Functions should be grouped and ordered according to their visibility](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions)

ProfilePicture:
* [ProfilePicture.mint](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L79) should be at the top since it's external

Bio:
* [Bio.mint](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L121) should be at the top since it's external

Namespace:
* [_drawing](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L245) should be at the bottom since it's private
* [Tray.tokenURI](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L119) should be after all external functions
* [Namespace.tokenURI](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L90) should be after all external functions

---

## Common practice is that imports are spaced out and grouped according to the SolidityLang ordering

[SolidityLang Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-layout) mentions the order of components in a file, and common convention is that imports are grouped and ordered the same as if they were components: Interfaces, Libraries, Contracts, for ease of readability.

All source files contain such imports, and need changing.

```
    import {ERC721} from "solmate/tokens/ERC721.sol";
    import {Owned} from "solmate/auth/Owned.sol";
    import {Base64} from "solady/utils/Base64.sol";
    import "./Tray.sol";
    import "./Utils.sol";
    import "../interface/Turnstile.sol";
    ->
    import "../interface/Turnstile.sol";

    import {Base64} from "solady/utils/Base64.sol";
    import "./Utils.sol";

    import {ERC721} from "solmate/tokens/ERC721.sol";
    import {Owned} from "solmate/auth/Owned.sol";
    import "./Tray.sol";
```


# [N-04] Slither should be configured to run only on source files, while currently it runs on everything

## Recommendation:
In each subprotocol folder, create a ```slither.config.json``` file with contents, filtering non-source code:

```
{
    "filter_paths": "(lib/|test/|node_modules/|src/test/)"
}
```

# [N-05] Namespace: ```characterToUnicodeBytes``` for fontClass == 0 should use numBytes for calculation for improved code readability

## Recommendation:
```
numBytes = 3;
byteOffset = _characterIndex * 3;
supportsSkinToneModifier = _characterIndex >= EMOJIS_LE_THREE_BYTES - EMOJIS_MOD_SKIN_TONE_THREE_BYTES;
->
numBytes = 3;
byteOffset = _characterIndex * numBytes;
supportsSkinToneModifier = _characterIndex >= EMOJIS_LE_THREE_BYTES - EMOJIS_MOD_SKIN_TONE_THREE_BYTES;
```

Change the above for all if statements (3, 4, 6, 7, 8, 14 bytes cases).

# [N-06] Namespace: Use of ```bytes.concat()``` instead of ```abi.encodePacked()```

When needing to append bytes, since Solidity compiler version 0.8.4, ```bytes.concat()``` can and should be used instead of ```abi.encodePacked(,)```.

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L104
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L110
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L115
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L149
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L158
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L167

# [N-07] Typos and wrong comments

```
asciiStartingIndex = 22; // Starting index for (lowercase) characters - 25
->
asciiStartingIndex = 22; // Starting index for digits, - 25
```

# [N-08] NatSpec comments should be increased in contracts

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in [the Solidity official documentation](https://docs.soliditylang.org/en/v0.8.15/natspec-format.html).


```Namespace```, ```Tray```, ```Utils``` and ```Bio``` contracts contain multiple complex functions which should be explained more, using the ```@dev``` NatSpec annotation. The logic around character to bytes encoding implies a lot of prior Unicode logic which is not easily made available to the reading eyes, through comments.

# [N-09] Tests don't test core functionality, although coverage is at 100%
The test suite was envisioned around 100% coverage of all statements and branches, but doesn't necessarily cover some important features of the code, vital to the project's functionality:

Examples:
* should have tests for ```characterToUnicodeBytes```, at least one test for each font class
* should have tests for ```characterToUnicodeBytes```, at least one for each byte number of EMOJIs
* should have tests for ```fuse```, checking that the resulting name is actually what is expected, at least one test for each font class

# [N-10] Missing Events for constructors + some Events are missing important data

Smart contracts' constructors should emit all the arguments used for construction, so they could be easily read off-chain, on block explorers.

```PrelaunchEnded``` event should emit final number of prelaunch NFTs emitted.

# [N-11] Namespace: Critical address changes should be two-step procedures

Changing the ```$NOTE``` address is critical for the well-being of the ecosystem, while changing the ```revenueAddress``` is relevant for the correct receival of funds. Transforming these into two-step processes would make them less error-prone.

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L210
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L218

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L196
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L204

## Recommendation:
Implement a similar technique to OpenZeppelin's [Ownable2Step](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol), where ```owner``` could request an address change, and then confirm it using a different function.

# [N-12] Uppercase immutable variables

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L14

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L37
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L50
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L17

# [N-13] Use underscores for number literals, for improved code readability

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L64

## Recommendation:
```
unchecked {
    iteratedState = _currState * 15485863;
    iteratedState = (iteratedState * iteratedState * iteratedState) % 2038074743;
}
->
unchecked {
    iteratedState = _currState * 15_485_863;
    iteratedState = (iteratedState * iteratedState * iteratedState) % 2_038_074_743;
}
```

# [S-01] Suggestion: To make Slither work on the canto-namespace-protocol folder, you need a different Slither version, like 0.9.1

https://github.com/crytic/slither/issues/1610

## Recommendation:
```
pip3 install slither-analyzer==0.9.1
slither --version
0.9.1
slither . ### no longer throws an error
```