# Report
## Gas Optimizations ##
### [G-1]: Place subtractions where the operands can't underflow in unchecked {} block
**Context:**

1. ```if ((i > 0 && (i + 1) % 40 == 0) || prevByteWasContinuation || i == lengthInBytes - 1) {``` [L60](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L60) (lengthInBytes - 1) 
1. ```if (i != lengthInBytes - 1) {``` [L62](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L62) 
1. ```bioTextBytes[i - 2] == 0xE2 &&``` [L80](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L80) 
1. ```bioTextBytes[i - 1] == 0x80 &&``` [L81](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L81) 
1. ```uint256 fusingCosts = 2**(13 - numCharacters) * 1e18;``` [L113](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L113) 
1. ```prelaunchMinted = _nextTokenId() - 1;``` [L227](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L227) 
1. ```tileData.fontClass = 3 + uint8((res - 80) / 8);``` [L259](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L259) 
1. ```tileData.fontClass = 5 + uint8((res - 96) / 4);``` [L261](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L261) 
1. ```tileData.fontClass = 7 + uint8((res - 104) / 2);``` [L263](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L263) 
1. ```supportsSkinToneModifier = _characterIndex >= EMOJIS_LE_THREE_BYTES - EMOJIS_MOD_SKIN_TONE_THREE_BYTES;``` [L86](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L86)
1. ```byteOffset = EMOJIS_BYTE_OFFSET_FOUR_BYTES + (_characterIndex - EMOJIS_LE_THREE_BYTES) * 4;``` [L89](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L89) 
1. ```supportsSkinToneModifier = _characterIndex >= EMOJIS_LE_FOUR_BYTES - EMOJIS_MOD_SKIN_TONE_FOUR_BYTES;``` [L90](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L90)
1. ```byteOffset = EMOJIS_BYTE_OFFSET_SIX_BYTES + (_characterIndex - EMOJIS_LE_FOUR_BYTES) * 6;``` [L93](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L93) 
1. ```byteOffset = EMOJIS_BYTE_OFFSET_SEVEN_BYTES + (_characterIndex - EMOJIS_LE_SIX_BYTES) * 7;``` [L96](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L96) 
1. ```byteOffset = EMOJIS_BYTE_OFFSET_EIGHT_BYTES + (_characterIndex - EMOJIS_LE_SEVEN_BYTES) * 8;``` [L99](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L99) 
1. ```byteOffset = EMOJIS_BYTE_OFFSET_FOURTEEN_BYTES + (_characterIndex - EMOJIS_LE_EIGHT_BYTES) * 14;``` [L102](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L102) 

**Description:**

Some gas can be saved by using an unchecked {} block if an underflow isn't possible because of a previous require() or if-statement.