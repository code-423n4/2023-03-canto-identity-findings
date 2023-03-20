### > Immutable address (_namespaceNFT) should 0-Check in Constructor
In order to reduce human error which can't be changed back, check that the address _namespaceNFT specified in the constructor is different than address(0). Not doing so might lead to non-functional contract when it is updated to 0-address accidentally.

**https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L109**

### > NatSpec is incomplete (Missing: '@return')

**https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L195**

**https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L203**

**https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L119**

### > Use a more recent version of solidity
**Use a solidity version of at least 0.8.4 to get `bytes.concat()` instead of `abi.encodePacked(<bytes>,<bytes>)`**
**Use a solidity version of at least 0.8.12 to get `string.concat()` to be used instead of `abi.encodePacked(<str>,<str>)`**
**Use a solidity version of at least 0.8.13 to get the ability to use `using for` with a list of free functions**

**https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L2**
**https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L2**
**https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L2**
**https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L2**
**https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L2**