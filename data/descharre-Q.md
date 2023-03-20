# Summary
## Low Risk
|ID     | Finding| Instances |
|:----: | :---           |   :----:         |
|L-01       |Missing 0 address checks in constructors|3|
|L-02       |Buying over 180 trays exceeds block gas limit|1|
|L-03       |Transaction shouldn't revert when emoji does not support skin modifier|1|
|L-04       |TokenURI of ProfilePicture will revert when it's connected to the CIDNFT|1|



## Non critical
|ID     | Finding| Instances |
|:----: | :---           |   :----:         |
|N-01       |Unspecific compiler version pragma  | 1 |
|N-02       |Attach SafeTransferLib to ERC20  | 2 |
|N-03       |Unnecesarry variable assignments | 3 |
|N-04       |mapping `nftCharacters` does not get deleted | 1 |
|N-05       |Be consistent with how you generate metadata | 2 |


# Details
## Low Risk
## [L-01] Missing 0 address checks in constructors
Allowing zero-addresses will lead to contract reverts and force redeployments if there are no setters for such address variables.
- [ProfilePicture.sol#L58-L59](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L58-L59)
- [Tray.sol#L109](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L109)
- [Namespace.sol#L78](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L78)
## [L-02] Buying over 180 trays exceeds block gas limit
[`buy()`](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L150-L168) is a very expensive function. Around 180 trays can be bought at once before exceeding the block gas limit of 30 million. This also means the owner can't mint all 1000 of them during prelaunch in one transaction. One mitigation is to have a limit on `_amount` and add a require statement.
## [L-03] Transaction shouldn't revert when emoji does not support skin modifier
When an emoji does not support a skintonemodifier but one is given, the transaction will revert. There is no reason to revert, it's better to just return the emoji with skintonemodifier 0 rather than reverting.

[Utils.sol#L112-L128](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L112-L128)
```diff
        if (_characterModifier != 0) {
-           if (!supportsSkinToneModifier) revert EmojiDoesNotSupportSkinToneModifier(_characterIndex);
+           if (!supportsSkinToneModifier) return character;
            if (_characterModifier == 1) {
                character = abi.encodePacked(character, hex"F09F8FBB");
            } else if (_characterModifier == 2) {
                character = abi.encodePacked(character, hex"F09F8FBC");
            } else if (_characterModifier == 3) {
                character = abi.encodePacked(character, hex"F09F8FBD");
            } else if (_characterModifier == 4) {
                character = abi.encodePacked(character, hex"F09F8FBE");
            } else if (_characterModifier == 5) {
                character = abi.encodePacked(character, hex"F09F8FBF");
            } else {
                revert InvalidSkinToneModifierProvided(_characterModifier);
            }
        }
        return character;
```
## [L-04] TokenURI of ProfilePicture will revert when it's not connected to the CIDNFT
When a user mints a ProfilePicture but doesn't immediately add it to his CIDNFT, the function [`tokenURI()`](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L70-L74) will revert with the errorCode `PFPNoLongerOwnedByOriginalOwner`. Even though the owner probably still has the NFT. This is because the function [`getPFP()`](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L94-L105) will have a cidNFTID of 0. This might lead to users reminting their NFT because they think they have the wrong picture.

A better way do this would be to update a mapping everytime the pfp nft gets minted and check if that matches the owner of the nft.
```diff
L34:
+    mapping(uint256 => address) private pfpMinter;

L79:
    function mint(address _nftContract, uint256 _nftID) external {
        uint256 tokenId = ++numMinted;
        if (ERC721(_nftContract).ownerOf(_nftID) != msg.sender)
            revert PFPNotOwnedByCaller(msg.sender, _nftContract, _nftID);
        ProfilePictureData storage pictureData = pfp[tokenId];
        pictureData.nftContract = _nftContract;
        pictureData.nftID = _nftID;
+       pfpMinter[tokenId] = msg.sender;
        _mint(msg.sender, tokenId);
        emit PfpAdded(msg.sender, tokenId, _nftContract, _nftID);
    }

L94:
    function getPFP(uint256 _pfpID) public view returns (address nftContract, uint256 nftID) {
        if (_ownerOf[_pfpID] == address(0)) revert TokenNotMinted(_pfpID);
        ProfilePictureData storage pictureData = pfp[_pfpID];
        nftContract = pictureData.nftContract;
        nftID = pictureData.nftID;
        uint256 cidNFTID = cidNFT.getPrimaryCIDNFT(subprotocolName, _pfpID);
        IAddressRegistry addressRegistry = cidNFT.addressRegistry();
-       if (cidNFTID == 0 || addressRegistry.getAddress(cidNFTID) != ERC721(nftContract).ownerOf(nftID)) {
+       if (pfpMinter[_pfpID] != ERC721(nftContract).ownerOf(nftID)) {
            nftContract = address(0);
            nftID = 0; // Strictly not needed because nftContract has to be always checked, but reset nevertheless to 0
        }
    }
```
## Non critical
## [N-01] Unspecific compiler version pragma
Avoid floating pragmas for non-library contracts. Contracts should be deployed with the same compiler version and flags that they have been tested the most with. Locking the pragma helps ensure that contracts do not accidentally get deployed using, for example, the latest compiler which may have higher risks of undiscovered bugs. Contracts may also be deployed by others and the pragma indicates the compiler version intended by the original authors.
## [N-02] Attach SafeTransferLib to ERC20
You can attach library function to any type with the for statement. This is used in most protocols and is the most readable.

[Tray.sol#L157](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L157)
```diff
L14:
+   using SafeTransferLib for ERC20;

L157:
-   SafeTransferLib.safeTransferFrom(note, msg.sender, revenueAddress, _amount * trayPrice);
+   note.safeTransferFrom(msg.sender, revenueAddress, _amount * trayPrice);
```
The same can be done in [Namespace](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L114).
## [N-03] Unnecesarry variable assignments
It's useless to write a memory variable to another memory variable. Even when it's a struct, it doesn't save any more gas. The extra assignments can be removed

[Namespace.sol#L133-L145](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L133-L145)
```diff
        Tray.TileData memory tileData = tray.getTile(trayID, tileOffset); // Will revert if tileOffset is too high
-       uint8 characterModifier = tileData.characterModifier;

        if (tileData.fontClass != 0 && _characterList[i].skinToneModifier != 0) {
            revert CannotFuseCharacterWithSkinTone();
        }
            
        if (tileData.fontClass == 0) {
            // Emoji
-           characterModifier = _characterList[i].skinToneModifier;
+           tileData.characterModifier = _characterList[i].skinToneModifier;
        }
-       bytes memory charAsBytes = Utils.characterToUnicodeBytes(0, tileData.characterIndex, characterModifier);
+       bytes memory charAsBytes = Utils.characterToUnicodeBytes(0, tileData.characterIndex, tileData.characterModifier);
-       tileData.characterModifier = characterModifier;
```

## [N-04] mapping `nftCharacters` does not get deleted
When you [`burn()`](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L184-L192) a namespace, the mappings `tokenToName` and [`nameToToken`] gets deleted. But the mapping [`nftCharacters`](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L49) is also associated to that nameSpace so it should also be deleted.

## [N-05] Be consistent with how you generate metadata
The function tokenURI() in [Namespace](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L90-L106) and [Tray](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L119-L146) both use abi.encodePacked to generate the json metadata. The function tokenURI() in [Bio](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L43-L117) uses string.concat. It's best to make this consistent accross the whole protocol.
