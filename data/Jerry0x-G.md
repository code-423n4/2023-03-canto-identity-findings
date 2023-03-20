https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L57-L64

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
To
```
    struct TileData {
        /// @notice Allowed values between 0 (emoji) and 9 (font5 rare)
        uint8 fontClass;
        /// @notice For Emojis (font class 0) between 0..NUM_CHARS_EMOJIS - 1, otherwise between 0..NUM_CHARS_LETTERS - 1
        uint16 characterIndex;
        /// @notice For generative fonts with randomness (Zalgo), we generate and fix this on minting. For some emojis, it can be set by the user to influence the skin color
        uint232 characterModifier;
    }
```

Make the structure occupy exactly uint256.

Gas decreased from 231292 to 231208.