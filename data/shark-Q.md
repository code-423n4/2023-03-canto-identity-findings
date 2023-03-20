## 1. Use a more recent version of solidity

It is considered best practice to use the latest version of solidity. More recent versions of solidity have compiler optimizations, bugfixes, among other things. This could help in reading and writing safe and clean code.

A list of known compiler bugs can be found [here](https://etherscan.io/solcbuginfo).

Here are some instances of this issue:

File: `Bio.sol` [Line 2](https://github.com/code-423n4/2023-03-canto-identity/blob/HEAD/canto-bio-protocol/src/Bio.sol#L2)

```solidity
pragma solidity >=0.8.0;
```

File: `Namespace.sol` [Line 2](https://github.com/code-423n4/2023-03-canto-identity/blob/HEAD/canto-namespace-protocol/src/Namespace.sol#L2)

```solidity
pragma solidity >=0.8.0;
```

File: `Tray.sol` [Line 2](https://github.com/code-423n4/2023-03-canto-identity/blob/HEAD/canto-namespace-protocol/src/Tray.sol#L2)

```solidity
pragma solidity >=0.8.0;
```

File: `ProfilePicture.sol` [Line 2](https://github.com/code-423n4/2023-03-canto-identity/blob/HEAD/canto-pfp-protocol/src/ProfilePicture.sol#L2)

```solidity
pragma solidity >=0.8.0;
```

## 2. Consider using `constant` variables instead of magic numbers

For example:

https://github.com/code-423n4/2023-03-canto-identity/blob/HEAD/canto-bio-protocol/src/Bio.sol#L123

```solidity
        if (bytes(_bio).length == 0 || bytes(_bio).length > 200) revert InvalidBioLength(bytes(_bio).length);
```

Consider defining a constant for the above instance. e.g: `uint256 public constant MAX_BIO_LENGTH = 200;`

Other instances:

- File: `Namespace.sol` [Line 112-113](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L112-L113)

## 3. Use `delete` to clear variables instead of zero assignment, i.e. (0, address(0), false)

A better way to indicate that you are clearing a variable is to use the [`delete`](https://docs.soliditylang.org/en/v0.8.17/types.html#delete) keyword.

File: [`Bio.sol`](https://github.com/code-423n4/2023-03-canto-identity/blob/HEAD/canto-bio-protocol/src/Bio.sol#L92)

```solidity
File: canto-bio-protocol/src/Bio.sol

92:    prevByteWasContinuation = false;
93:    bytesOffset = 0;
```

File: [`ProfilePicture.sol`](https://github.com/code-423n4/2023-03-canto-identity/blob/HEAD/canto-pfp-protocol/src/ProfilePicture.sol#L102)

```solidity
File: canto-pfp-protocol/src/ProfilePicture.sol

102:    nftContract = address(0);
103:    nftID = 0; // Strictly not needed because nftContract has to be always checked, but reset nevertheless to 0
```

File: `Namespace.sol` [Line 129](https://github.com/code-423n4/2023-03-canto-identity/blob/HEAD/canto-namespace-protocol/src/Namespace.sol#L129)

```solidity
File: canto-namespace-protocol/src/Namespace.sol

129:    isLastTrayEntry = false;
```

## 4. Contracts are not fully documented with NatSpec

It is recommended that contracts be thoroughly documented using [NatSpec](https://docs.soliditylang.org/en/develop/natspec-format.html). Using NatSpec will improve readability for all readers.

Here are some examples:

missing `@return`:

File: `Bio.sol` [Line 40-43](https://github.com/code-423n4/2023-03-canto-identity/blob/HEAD/canto-bio-protocol/src/Bio.sol#L40-L43)

```solidity
    /// @notice Get the token URI for the specified _id
    /// @dev Generates an on-chain SVG with a new line after 40 bytes. Line splitting generally supports UTF-8 multibyte characters and emojis, but is not tested for arbitrary UTF-8 characters
    /// @param _id ID to query for
    function tokenURI(uint256 _id) public view override returns (string memory) { /// @audit missing @return
```

missing NatSpec entirely:

File: [`Tray.sol`](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L231)

```solidity
File: canto-namespace-protocol/src/Tray.sol

        /// @audit missing NatSpec entirely
231:    function _beforeTokenTransfers(

        /// @audit missing NatSpec entirely
245:    function _drawing(uint256 _seed) private pure returns (TileData memory tileData) {
```

## 5. Typos

File: `Namespace.sol` [Line 152](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L152)

```solidity
            /// @audit occurence -> occurrence                                                                      vvvvvvvvv
            // We keep track of the unique trays NFTs (for burning them) and only check the owner once for the last occurence of the tray
```

## 6. Use named imports

Using named imports can improve code readability.

```diff
-import "../interface/Turnstile.sol";
-import "../interface/ICidNFT.sol";
+import {Turnstile} from "../interface/Turnstile.sol";
+import {ICidNFT} from "../interface/ICidNFT.sol";
```

## 7. Check effect interactions pattern not used

In the `fuse()` function of the `Namespace` contract, the [Checks Effects Interactions Pattern](https://fravoll.github.io/solidity-patterns/checks_effects_interactions.html) is not followed when performing transactions. It is recommended to follow the Check Effects Interactions to prevent reentrancy bugs.

For example, the following `safeTransferFrom` should be performed once all state variables have been updated accordingly:

File: `Namespace.sol` [Line 114](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L114)

```
        SafeTransferLib.safeTransferFrom(note, msg.sender, revenueAddress, fusingCosts);
```

## 8. Custom errors can be used with parameters

Custom errors can use parameters to show what actually went wrong.

For example, the following errors could use parameters:

https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L87-L88

```solidity
File: Tray.sol

87:    error OnlyOwnerCanMintPreLaunch();
88:    error MintExceedsPreLaunchAmount();
```

## 9. Use named parameters for mapping type declarations

Consider using named parameters in mappings to improve readability. Note: This feature is present since Solidity 0.8.18

For example:

https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L32

```diff
-    mapping(uint256 => ProfilePictureData) private pfp;
+    mapping(uint256 pfpID => ProfilePictureData) private pfp;
```

https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L18

```diff
-    mapping(uint256 => string) public bio;
+    mapping(uint256 id => string bioText) public bio;
```

File: `Namespace.sol` [Line 42-49](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L42-L49)

```diff
-    mapping(string => uint256) public nameToToken;
+    mapping(string name => uint256 nftId) public nameToToken;

-    mapping(uint256 => string) public tokenToName;
+    mapping(uint256 nftId => string name) public tokenToName;

-    mapping(uint256 => Tray.TileData[]) private nftCharacters;
+    mapping(uint256 nftId => Tray.TileData[]) private nftCharacters;
```
