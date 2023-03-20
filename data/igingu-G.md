# [G-01] 6/7 storage reads can be removed by storing all tiles inside one uint256

Each Tile uses 32 bits on memory:
```
struct TileData {
    /// @notice Allowed values between 0 (emoji) and 9 (font5 rare)
    uint8 fontClass;
    /// @notice For Emojis (font class 0) between 0..NUM_CHARS_EMOJIS - 1, otherwise between 0..NUM_CHARS_LETTERS - 1
    uint16 characterIndex;
    /// @notice For generative fonts with randomness (Zalgo), we generate and fix this on minting. For some emojis, it can be set by the user to influence the skin color
    uint8 characterModifier;
}
```

There can be at most 7 tiles, which means we could store all 7 tiles inside just one uint256:
* tile 0: bits 0 -> 31
* tile 1: bits 32 -> 63
...
* tile 6: bits 192 -> 227

This would serve as a major gas optimization, since we would only read 1 uint256 slot from storage for each tray, instead of 7 uint256 slots. These 7 reads are done extensively across the Namespace protocol implementation, since they are part of ```Tray.mint```, ```Tray.buy```, ```Tray.tokenURI```, as well as ```Namespace.fuse```, basically the functions that will be called 99.5% of the time. 

## Recommendation:
Will write below how the code would look for ```getTile```.

```
mapping(uint256 => TileData[TILES_PER_TRAY]) private tiles;

function getTile(uint256 _trayId, uint8 _tileOffset) external view returns (TileData memory tileData) {
    if (!_exists(_trayId)) revert TrayNotMinted(_trayId);
    tileData = tiles[_trayId][_tileOffset];
}
-> 
uint256 private constant BITS_PER_TILE = 32;
uint256 private constant 32BIT_BITMASK = (1 << BITS_PER_TILE) - 1;
uint256 private constant 16BIT_BITMASK = (1 << 16) - 1;
uint256 private constant 8BIT_BITMASK = (1 << 8) - 1;

mapping(uint256 => uint256) private tiles;

function getTile(uint256 _trayId, uint8 _tileOffset) external view returns (TileData memory tileData) {
    if (!_exists(_trayId)) revert TrayNotMinted(_trayId);

    uint256 tileAtOffset = (tiles[_trayId] >> (_tileOffset * BITS_PER_TILE)) & 32BIT_BITMASK;
    tileData.fontClass = tileAtOffset & 8BIT_BITMASK;
    tileAtOffset = tileAtOffset >> 8;

    tileData.characterIndex = tileAtOffset & 16BIT_BITMASK;
    tileAtOffset = tileAtOffset >> 16;

    tileData.characterModifier = tileAtOffset & 8BIT_BITMASK;
}
```



# [G-02] ProfilePicture.getPFP could be reordered for cleaner code, and to save gas

## Recommendation:
```
canto-pfp-protocol/src/ProfilePicture.sol:95-103
    nftContract = pictureData.nftContract;
    nftID = pictureData.nftID;
    uint256 cidNFTID = cidNFT.getPrimaryCIDNFT(subprotocolName, _pfpID);
    IAddressRegistry addressRegistry = cidNFT.addressRegistry();
    if (cidNFTID == 0 || addressRegistry.getAddress(cidNFTID) != ERC721(nftContract).ownerOf(nftID)) {
        nftContract = address(0);
        nftID = 0; // Strictly not needed because nftContract has to be always checked, but reset nevertheless to 0
    }
    -> 
    uint256 cidNFTID = cidNFT.getPrimaryCIDNFT(subprotocolName, _pfpID);
    IAddressRegistry addressRegistry = cidNFT.addressRegistry();
    if (cidNFTID > 0 || addressRegistry.getAddress(cidNFTID) == ERC721(nftContract).ownerOf(pictureDate.nftID)) {
        nftContract = pictureData.nftContract;
        nftID = pictureData.nftID;
    }
```

# [G-03] Can use storage references to save gas

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L45

## Recommendation:
```
canto-bio-protocol/src/Bio.sol:45
    string memory bioText = bio[_id];
    -> 
    string storage bioText = bio[_id];
```

# [G-04] For font classes 2, 3, 4, 5, 8, 9, startingSequence contains 4 bytes but only the last 2 are ever used

When determining the startingSequence, all classes have the first two bytes the same: ["F09D"](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L182), hence they are redundant, which is also shown in the ```_getUtfSequence``` function, which only uses bytes 3 and 4 ([positions 2 and 3](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L278)). 

## Recommendation:

Only return the last 2 bytes for each startingSequence. Update the ```_getUtfSequence``` function to work with positions 0 and 1 instead, and return "F09D" + ```_startingSequence``` instead.

# [G-05] Use calldata instead of memory for function arguments that do not get mutated

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L267

Mark data types as calldata instead of memory where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as calldata. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies memory storage.

# [G-06] Check that tray with ```_trayId``` exists can be extracted to modifier to save gas and improve readability

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L120
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L196
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L204

## Recommendation:
```
modifier trayMinted(uint256 _trayId) {
    if (!_exists(_trayId)) revert TrayNotMinted(_trayId);
    _;
}

function getTiles(uint256 _trayId) external view trayMinted(_trayId) returns (TileData[TILES_PER_TRAY] memory tileData) {
    if (!_exists(_trayId)) revert TrayNotMinted(_trayId);
    tileData = tiles[_trayId];
}
```

# [G-07] ```iteratePRNG``` could be simpler to save gas, while giving the same amount of pseudo-randomness

Unless there is a mathematical explanation about the ```iteratePRNG``` algorithm, gas could be saved by doing less multiplications, which would give the same amount of pseudorandomness.

## Recommendation: 
```
    function iteratePRNG(uint256 _currState) internal pure returns (uint256 iteratedState) {
        unchecked {
            iteratedState = (_currState ** 2) % 2038074743;
        }
    }
```

# [G-08] State variables that don't change should be marked as ```immutable``` to save gas

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L35

# [G-09] Use `assembly` to write address storage values

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L107
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L109
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L220

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L107
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L109
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L220

## Recommendation:
```solidity
canto-pfp-protocol/ProfilePicture/src/ProfilePicture.sol:84:
    pictureData.nftContract = _nftContract;
    ->
    assembly {
        sstore(pictureData.nftContract.slot, _nftContract)
    }
```

# [G-10] To prevent having to redeploy contracts and waste gas, require immutable storage members to have been provided values in constructors

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L78
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L50
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L37


## Recommendation
```
lastHash = _initHash;
trayPrice = _trayPrice;
revenueAddress = _revenueAddress;
note = ERC20(_note);
namespaceNFT = _namespaceNFT;
->
if (_trayPrice == 0) revert TrayPriceCantBeZero;
if (_namespaceNFT == address(0)) revert NamespaceNFTCantBeZeroAddress;
lastHash = _initHash;
trayPrice = _trayPrice;
revenueAddress = _revenueAddress;
note = ERC20(_note);
namespaceNFT = _namespaceNFT;
```

# [G-11] Functions guaranteed to revert if called by users can be marked `payable` to save gas
When functions are not `payable`, Solidity compiler introduces 10 more opcodes, to assert that `msg.value` is 0 =>
CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2), which cost
about 21 gas per call to the function, in addition to extra deployment cost.

```onlyOwner``` functions could be payable, as owners are less prone to send ETH to the contract by mistake.

All constructors can also be made `payable`, to avoid additional gas cost on deployment.

The above is a saving with no security risk, other than owners accidentally sending ether to the smart contracts.

# [G-12] Forgotten ```delete``` in ```Namespace.burn``` would recover gas

Burning a Namespace NFT [deletes the tokenToName and nameToToken mapping entries](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L190), but doesn't delete ```nftCharacters[_id]```, which holds the ```TileData``` information for that specific Namespace NFT.

## Recommendation:
```
delete tokenToName[_id];
delete nameToToken[associatedName];
->
delete nftCharacters[_id];
delete tokenToName[_id];
delete nameToToken[associatedName];
```

# [G-13] Whole ```Namespace.fuse``` function can be re-written to prioritise revert checks

Functions that revert should prioritise the revert code first and then the effects. Reverting as early as possible saves the calling user as much gas as possible, while adding no security risks or complexity. 

## Recommendation:
Rewrite ```Namespace.fuse``` like so:
```
function fuse(CharacterData[] calldata _characterList) external {
    uint256 numCharacters = _characterList.length;
    if (numCharacters > 13 || numCharacters == 0) revert InvalidNumberOfCharacters(numCharacters);

    uint256 fusingCosts = 2**(13 - numCharacters) * 1e18;
    SafeTransferLib.safeTransferFrom(note, msg.sender, revenueAddress, fusingCosts);

    uint256 namespaceIDToMint = ++nextNamespaceIDToMint;
    Tray.TileData[] storage nftToMintCharacters = nftCharacters[namespaceIDToMint];

    bytes memory bName = new bytes(numCharacters * 33); // Used to convert into a string. Can be 33 times longer than the string at most (longest zalgo characters is 33 bytes)
    uint256 numBytes;

    // Extract unique trays for burning them later on
    uint256 numUniqueTrays;
    uint256[] memory uniqueTrays = new uint256[](_characterList.length);

    for (uint256 i; i < numCharacters; ++i) {
        bool isLastTrayEntry = true;
        uint256 trayID = _characterList[i].trayID;
        uint8 tileOffset = _characterList[i].tileOffset;
        // Check for duplicate characters in the provided list. 1/2 * n^2 loop iterations, but n is bounded to 13 and we do not perform any storage operations
        for (uint256 j = i + 1; j < numCharacters; ++j) {
            if (_characterList[j].trayID == trayID) {
                if (_characterList[j].tileOffset == tileOffset) revert FusingDuplicateCharactersNotAllowed();
                isLastTrayEntry = false;
            }
        }

        if (isLastTrayEntry) {
            // Verify address is allowed to fuse
            address trayOwner = tray.ownerOf(trayID);
            if (
                trayOwner != msg.sender &&
                tray.getApproved(trayID) != msg.sender &&
                !tray.isApprovedForAll(trayOwner, msg.sender)
            ) revert CallerNotAllowedToFuse();

            uniqueTrays[numUniqueTrays++] = trayID;
        }

        Tray.TileData memory tileData = tray.getTile(trayID, tileOffset); // Will revert if tileOffset is too high
        if (tileData.fontClass != 0 && _characterList[i].skinToneModifier != 0) {
            revert CannotFuseCharacterWithSkinTone();
        }
        
        uint8 characterModifier = tileData.characterModifier;
        if (tileData.fontClass == 0) {
            // Emoji
            characterModifier = _characterList[i].skinToneModifier;
        }

        bytes memory charAsBytes = Utils.characterToUnicodeBytes(0, tileData.characterIndex, characterModifier);
        tileData.characterModifier = characterModifier;
        uint256 numBytesChar = charAsBytes.length;
        charAsBytes = bytes.concat(charAsBytes, )
        for (uint256 j; j < numBytesChar; ++j) {
            bName[numBytes + j] = charAsBytes[j];
        }
        numBytes += numBytesChar;
        nftToMintCharacters.push(tileData);
        // We keep track of the unique trays NFTs (for burning them) and only check the owner once for the last occurence of the tray
    }
    // Set array to the real length (in bytes) to avoid zero bytes in the end when doing the string conversion
    assembly {
        mstore(bName, numBytes)
    }
    string memory nameToRegister = string(bName);
    uint256 currentRegisteredID = nameToToken[nameToRegister];
    if (currentRegisteredID != 0) revert NameAlreadyRegistered(currentRegisteredID);
    nameToToken[nameToRegister] = namespaceIDToMint;
    tokenToName[namespaceIDToMint] = nameToRegister;

    for (uint256 i; i < numUniqueTrays; ++i) {
        tray.burn(uniqueTrays[i]);
    }
    _mint(msg.sender, namespaceIDToMint);
    // Although _mint already emits an event, we additionally emit one because of the name
    emit NamespaceFused(msg.sender, namespaceIDToMint, nameToRegister);
}
```