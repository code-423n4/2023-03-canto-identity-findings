# Gas Optimizations Report for Canto Identity Subprotocols contest
## Overview
During the audit, 1 gas issue were found.  
Total savings ~560.

№ | Title | Instance Count | Saved
--- | --- | --- | ---
G-1 | Use unchecked blocks for subtractions where underflow is impossible | 16 | 560

## Gas Optimizations Findings(1)
### G-1. Use unchecked blocks for subtractions where underflow is impossible
##### Description
In Solidity 0.8+, there’s a default overflow and underflow check on unsigned integers. When an overflow or underflow isn’t possible (after require or if-statement), some gas can be saved by using [unchecked blocks](https://docs.soliditylang.org/en/v0.8.17/control-structures.html#checked-or-unchecked-arithmetic).
##### Instances
- [```if ((i > 0 && (i + 1) % 40 == 0) || prevByteWasContinuation || i == lengthInBytes - 1) {```](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L60) (lengthInBytes - 1) - checked in line 49.
- [```if (i != lengthInBytes - 1) {```](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L62) checked in line 49.
- [```bioTextBytes[i - 2] == 0xE2 &&```](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L80) checked in line 79.
- [```bioTextBytes[i - 1] == 0x80 &&```](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L81) checked in line 79.
- [```uint256 fusingCosts = 2**(13 - numCharacters) * 1e18;```](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L113) checked in line 112.
- [```prelaunchMinted = _nextTokenId() - 1;```](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L227) safe because startTokenId = 1.
- [```tileData.fontClass = 3 + uint8((res - 80) / 8);```](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L259) safe because here res >= 80.
- [```tileData.fontClass = 5 + uint8((res - 96) / 4);```](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L261) safe because here res >= 96.
- [```tileData.fontClass = 7 + uint8((res - 104) / 2);```](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L263) safe because here res >= 104.
- [```supportsSkinToneModifier = _characterIndex >= EMOJIS_LE_THREE_BYTES - EMOJIS_MOD_SKIN_TONE_THREE_BYTES;```](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L86) 
- [```byteOffset = EMOJIS_BYTE_OFFSET_FOUR_BYTES + (_characterIndex - EMOJIS_LE_THREE_BYTES) * 4;```](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L89) checked in line 83.
- [```supportsSkinToneModifier = _characterIndex >= EMOJIS_LE_FOUR_BYTES - EMOJIS_MOD_SKIN_TONE_FOUR_BYTES;```](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L90) 
- [```byteOffset = EMOJIS_BYTE_OFFSET_SIX_BYTES + (_characterIndex - EMOJIS_LE_FOUR_BYTES) * 6;```](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L93) checked in line 87.
- [```byteOffset = EMOJIS_BYTE_OFFSET_SEVEN_BYTES + (_characterIndex - EMOJIS_LE_SIX_BYTES) * 7;```](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L96) checked in line 91.
- [```byteOffset = EMOJIS_BYTE_OFFSET_EIGHT_BYTES + (_characterIndex - EMOJIS_LE_SEVEN_BYTES) * 8;```](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L99) checked in line 94.
- [```byteOffset = EMOJIS_BYTE_OFFSET_FOURTEEN_BYTES + (_characterIndex - EMOJIS_LE_EIGHT_BYTES) * 14;```](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L102) checked in line 97.

##### Saved
This saves ~35.  
So, ~35*16 = 560