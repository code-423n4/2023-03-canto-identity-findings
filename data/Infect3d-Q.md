# Low Risk and Non-Critical Issues

## Missing zero address check when assigning value to immutable variable

### ProfilePicture::constructor
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L58
cidNFTID is an immutable variable, if not set correclty the contract is unusable

### Tray::constructor
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L109
namespaceNFT is an immutable variable, if not set correclty the contract is unusable