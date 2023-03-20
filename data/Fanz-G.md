# Report

## Gas Optimization

|                 | Issue                                                | Instances |
| --------------- | :--------------------------------------------------- | :-------: |
| [GAS-1](#GAS-1) | Set variables to empty value instead of use delete   |     2     |
| [GAS-2](#GAS-2) | Use nested if and, avoid multiple check combinations |     3     |
| [GAS-3](#GAS-3) | Sort Solidity operations using short-circuit mode    |     2     |
| [GAS-4](#GAS-4) | ProfilePicture could refactor for gas efficient      |     1     |
| [GAS-5](#GAS-5) | Bio could refactor for gas efficient                 |     1     |
| [GAS-6](#GAS-6) | packing storage slots                                |     1     |
| [GAS-7](#GAS-7) | Variables/Constants could be set to private          |     3     |

### <a name="GAS-1"></a>[GAS-1] set Variables to empty value instead of use delete

### <a name="GAS-2"></a>[GAS-2] Cache array length outside of loop

File: https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L189

```solidity
    delete tokenToName[_id];
    delete nameToToken[associatedName];

```

### <a name="GAS-2"></a>[GAS-2] Use nested if and, avoid multiple check combinations

File: https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L186

```solidity
    if (nftOwner != msg.sender && getApproved[_id] != msg.sender && !isApprovedForAll[nftOwner][msg.sender])
        revert CallerNotAllowedToBurn();

```

File: https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L175

```solidity
    if (
            namespaceNFT != msg.sender &&
            trayOwner != msg.sender &&
            getApproved(_id) != msg.sender &&
            !isApprovedForAll(trayOwner, msg.sender)
        ) revert CallerNotAllowedToBurn();

```

File: https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L60

```solidity
    if ((i > 0 && (i + 1) % 40 == 0) || prevByteWasContinuation || i == lengthInBytes - 1)

```

### <a name="GAS-3"></a>[GAS-3] Sort Solidity operations using short-circuit mode

Short-circuiting is a solidity contract development model that uses OR/AND logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low, if the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

```solidity
    //f(x) is a low gas cost operation
    //g(y) is a high gas cost operation
    //Sort operations with different gas costs as follows
    f(x) || g(y)
    f(x) && g(y)

```

File: https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L60

```solidity
    if ((i > 0 && (i + 1) % 40 == 0) || prevByteWasContinuation || i == lengthInBytes - 1)

```

File: https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L112

```solidity
    if (numCharacters > 13 || numCharacters == 0) revert InvalidNumberOfCharacters(numCharacters);

```

### <a name="GAS-4"></a>[GAS-4] ProfilePicture could refactor for gas efficient

-   Deployment Gas Saved: 7807
-   forge snapshot --diff: 827

File: https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol

```solidity
diff --git a/canto-pfp-protocol/src/ProfilePicture.sol b/canto-pfp-protocol/src/ProfilePicture.sol
index 9e99a08..014f989 100644
--- a/canto-pfp-protocol/src/ProfilePicture.sol
+++ b/canto-pfp-protocol/src/ProfilePicture.sol
@@ -44,8 +44,7 @@ contract ProfilePicture is ERC721 {
     /*//////////////////////////////////////////////////////////////
                                  ERRORS
     //////////////////////////////////////////////////////////////*/
-    error TokenNotMinted(uint256 tokenID);
-    error PFPNoLongerOwnedByOriginalOwner(uint256 tokenID);
+    error TokenNotExist(uint256 tokenID);
     error PFPNotOwnedByCaller(address caller, address nftContract, uint256 nftID);

     /// @notice Initiates CSR on mainnet
@@ -66,7 +65,6 @@ contract ProfilePicture is ERC721 {
     /// @dev Reverts if PFP is no longer owned by owner of associated CID NFT
     function tokenURI(uint256 _id) public view override returns (string memory) {
         (address nftContract, uint256 nftID) = getPFP(_id);
-        if (nftContract == address(0)) revert PFPNoLongerOwnedByOriginalOwner(_id);
         return ERC721(nftContract).tokenURI(nftID);
     }

@@ -90,15 +88,18 @@ contract ProfilePicture is ERC721 {
     /// @param _pfpID Profile picture NFT ID to query
     /// @return nftContract The referenced NFT contract (address(0) if no longer owned), nftID The referenced NFT ID
     function getPFP(uint256 _pfpID) public view returns (address nftContract, uint256 nftID) {
-        if (_ownerOf[_pfpID] == address(0)) revert TokenNotMinted(_pfpID);
+        if (_ownerOf[_pfpID] == address(0)) revert TokenNotExist(_pfpID);
+        uint256 cidNFTID = cidNFT.getPrimaryCIDNFT(subprotocolName, _pfpID);
+
+        if (cidNFTID == 0) revert TokenNotExist(_pfpID);
+
         ProfilePictureData storage pictureData = pfp[_pfpID];
+        if (pictureData.nftContract == address(0)) revert TokenNotExist(_pfpID);
+
         nftContract = pictureData.nftContract;
         nftID = pictureData.nftID;
-        uint256 cidNFTID = cidNFT.getPrimaryCIDNFT(subprotocolName, _pfpID);
+
         IAddressRegistry addressRegistry = cidNFT.addressRegistry();
-        if (cidNFTID == 0 || addressRegistry.getAddress(cidNFTID) != ERC721(nftContract).ownerOf(nftID)) {
-            nftContract = address(0);
-            nftID = 0; // Strictly not needed because nftContract has to be always checked, but reset nevertheless to 0
-        }
+        if (addressRegistry.getAddress(cidNFTID) != ERC721(nftContract).ownerOf(nftID)) revert TokenNotExist(_pfpID);
     }
 }

```

File: https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/test/ProfilePicture.t.sol

```solidity
diff --git a/canto-pfp-protocol/src/test/ProfilePicture.t.sol b/canto-pfp-protocol/src/test/ProfilePicture.t.sol
index ca047ae..cd851c7 100644
--- a/canto-pfp-protocol/src/test/ProfilePicture.t.sol
+++ b/canto-pfp-protocol/src/test/ProfilePicture.t.sol
@@ -82,7 +82,7 @@ contract ProfilePictureTest is DSTest {
     }

     function testGetPFPWithNotMintedTokenID() public {
-        vm.expectRevert(abi.encodeWithSelector(ProfilePicture.TokenNotMinted.selector, 1000));
+        vm.expectRevert(abi.encodeWithSelector(ProfilePicture.TokenNotExist.selector, 1000));
         pfp.getPFP(1000);
     }

@@ -101,7 +101,7 @@ contract ProfilePictureTest is DSTest {
         mockCidNFT.mockReturnPrimaryCIDNFT(0);

         // should revert since getPrimaryCIDNFT() is 0
-        vm.expectRevert(abi.encodeWithSelector(ProfilePicture.PFPNoLongerOwnedByOriginalOwner.selector, tokenId));
+        vm.expectRevert(abi.encodeWithSelector(ProfilePicture.TokenNotExist.selector, tokenId));
         pfp.tokenURI(tokenId);

         // mock cidNFTID to non-zero
@@ -110,7 +110,7 @@ contract ProfilePictureTest is DSTest {
         mockCidNFT.addressRegistry().mockReturnAddress(user2);

         // should revert since user2 is not the owner of nftId
-        vm.expectRevert(abi.encodeWithSelector(ProfilePicture.PFPNoLongerOwnedByOriginalOwner.selector, tokenId));
+        vm.expectRevert(abi.encodeWithSelector(ProfilePicture.TokenNotExist.selector, tokenId));
         pfp.tokenURI(tokenId);
     }

@@ -124,11 +124,9 @@ contract ProfilePictureTest is DSTest {

         // mock return 0
         mockCidNFT.mockReturnPrimaryCIDNFT(0);
+        vm.expectRevert(abi.encodeWithSelector(ProfilePicture.TokenNotExist.selector, 1));

-        (address nftContract, uint256 nftID) = pfp.getPFP(1);
-
-        assertEq(nftContract, address(0));
-        assertEq(nftID, 0);
+        pfp.getPFP(1);
     }

     function testNFTTransferredAfterwards() public {
@@ -144,10 +142,9 @@ contract ProfilePictureTest is DSTest {
         mockCidNFT.mockReturnPrimaryCIDNFT(1);
         mockCidNFT.addressRegistry().mockReturnAddress(user1);
         // since user1 no longer owns nft, nftContract should be address(0)
-        (address nftContract, uint256 nftID) = pfp.getPFP(1);

-        assertEq(nftContract, address(0));
-        assertEq(nftID, 0);
+        vm.expectRevert(abi.encodeWithSelector(ProfilePicture.TokenNotExist.selector, 1));
+        pfp.getPFP(1);
     }

     function testTokenURINFTOwnedByOwnerOfCIDNFT() public {

```

### <a name="GAS-5"></a>[GAS-5] Bio could refactor for gas efficient

-   forge snapshot --diff : 9395
-   Deployment Cost: 26627
-   mint avg: 3171
-   tokenURI avg: 9719

Before
| src/Bio.sol:Bio contract | | | | | |
|--------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost | Deployment Size | | | | |
| 1511035 | 8291 | | | | |
| Function Name | min | avg | median | max | # calls |
| bio | 1104 | 1104 | 1104 | 1104 | 1 |
| mint | 705 | 92459 | 116260 | 160632 | 7 |
| numMinted | 548 | 881 | 548 | 2548 | 6 |
| ownerOf | 522 | 522 | 522 | 522 | 1 |
| tokenURI | 80132 | 108686 | 113314 | 127984 | 4 |

After
| src/Bio.sol:Bio contract | | | | | |
|--------------------------|-----------------|-------|--------|--------|---------|
| Deployment Cost | Deployment Size | | | | |
| 1484408 | 8158 | | | | |
| Function Name | min | avg | median | max | # calls |
| bio | 1104 | 1104 | 1104 | 1104 | 1 |
| mint | 705 | 89288 | 116260 | 138458 | 7 |
| numMinted | 548 | 881 | 548 | 2548 | 6 |
| ownerOf | 522 | 522 | 522 | 522 | 1 |
| tokenURI | 78402 | 98967 | 97690 | 122087 | 4 |

```solidity
diff --git a/canto-bio-protocol/src/Bio.sol b/canto-bio-protocol/src/Bio.sol
index 635de76..c4ece94 100644
--- a/canto-bio-protocol/src/Bio.sol
+++ b/canto-bio-protocol/src/Bio.sol
@@ -53,6 +53,7 @@ contract Bio is ERC721 {
         // Because we do not split on zero-width joiners, line in bytes can technically be much longer. Will be shortened to the needed length afterwards
         bytes memory bytesLines = new bytes(80);
         uint256 bytesOffset;
+        string memory text;
         for (uint256 i; i < lengthInBytes; ++i) {
             bytes1 character = bioTextBytes[i];
             bytesLines[bytesOffset] = character;
@@ -93,26 +94,37 @@ contract Bio is ERC721 {
                 }
             }
         }
-        string memory svg =
-            '<svg xmlns="http://www.w3.org/2000/svg" preserveAspectRatio="xMinYMin meet" viewBox="0 0 400 100"><style>text { font-family: sans-serif; font-size: 12px; }</style>';
-        string memory text = '<text x="50%" y="50%" dominant-baseline="middle" text-anchor="middle">';
-        for (uint256 i; i < lines; ++i) {
+
+        for (uint256 i = 0; i < lines; ++i) {
             text = string.concat(text, '<tspan x="50%" dy="20">', strLines[i], "</tspan>");
         }
-        string memory json = Base64.encode(
-            bytes(
-                string.concat(
-                    '{"name": "Bio #',
-                    LibString.toString(_id),
-                    '", "description": "',
-                    bioText,
-                    '", "image": "data:image/svg+xml;base64,',
-                    Base64.encode(bytes(string.concat(svg, text, "</text></svg>"))),
-                    '"}'
+
+        return string(
+            abi.encodePacked(
+                "data:application/json;base64,",
+                Base64.encode(
+                    bytes(
+                        abi.encodePacked(
+                            '{"name": "Bio #',
+                            LibString.toString(_id),
+                            '", "description": "',
+                            bioText,
+                            '", "image": "data:image/svg+xml;base64,',
+                            Base64.encode(
+                                abi.encodePacked(
+                                    '<svg xmlns="http://www.w3.org/2000/svg" preserveAspectRatio="xMinYMin meet" viewBox="0 0 400 100"><style>text { font-family: sans-serif; font-size: 12px; }</style>',
+                                    '<text x="50%" y="50%" dominant-baseline="middle" text-anchor="middle">',
+                                    text,
+                                    "</text>",
+                                    "</svg>"
+                                )
+                            ),
+                            '"}'
+                        )
+                    )
                 )
             )
         );
-        return string(abi.encodePacked("data:application/json;base64,", json));
     }

     /// @notice Mint a new Bio NFT

```

### <a name="GAS-6"></a>[GAS-6] packing storage slots

File: https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L24

```solidity
     uint256 private constant ZALGO_NUM_ABOVE = 46;

    /// @notice UTF-8 encoding of possible characters that can be below a letter for Zalgo. All characters have a length of 2 bytes
    bytes private constant ZALGO_BELOW_LETTER =
        hex"CC96CC97CC98CC99CC9CCC9DCC9ECC9FCCA0CCA1CCA2CCA3CCA4CCA5CCA6CCA7CCA8CCA9CCAACCABCCACCCADCCAECCAFCCB0CCB1CCB2CCB3CCB9CCBACCBBCCBCCD85CD87CD88CD89CD8DCD8ECD93CD94CD95CD96CD99CD9ACD9CCD9FCDA2";

    /// @notice Number of characters that can be below a letter for Zalgo
    uint256 private constant ZALGO_NUM_BELOW = 47;

    /// @notice UTF-8 encoding of possible characters that can be over a letter (i.e., in the middle) for Zalgo. All characters have a length of 2 bytes
    bytes private constant ZALGO_OVER_LETTER = hex"CCB4CCB5CCB6CCB7CCB8";

    /// @notice Number of characters that can be over a letter for Zalgo
    uint256 private constant ZALGO_NUM_OVER = 5;

```

Recommend change to:

```solidity
    /// @notice UTF-8 encoding of possible characters that can be below a letter for Zalgo. All characters have a length of 2 bytes
    bytes private constant ZALGO_BELOW_LETTER =
        hex"CC96CC97CC98CC99CC9CCC9DCC9ECC9FCCA0CCA1CCA2CCA3CCA4CCA5CCA6CCA7CCA8CCA9CCAACCABCCACCCADCCAECCAFCCB0CCB1CCB2CCB3CCB9CCBACCBBCCBCCD85CD87CD88CD89CD8DCD8ECD93CD94CD95CD96CD99CD9ACD9CCD9FCDA2";

    /// @notice Number of characters that can be above a letter for Zalgo
    uint64 private constant ZALGO_NUM_ABOVE = 46;

    /// @notice Number of characters that can be below a letter for Zalgo
    uint64 private constant ZALGO_NUM_BELOW = 47;

    /// @notice UTF-8 encoding of possible characters that can be over a letter (i.e., in the middle) for Zalgo. All characters have a length of 2 bytes
    bytes12 private constant ZALGO_OVER_LETTER = hex"CCB4CCB5CCB6CCB7CCB8";

    /// @notice Number of characters that can be over a letter for Zalgo
    uint64 private constant ZALGO_NUM_OVER = 5;

```

Result:

src/Namespace.sol:Namespace
Deployment Cost: 3409

src/test/mock/MockTray.sol:MockTray
Deployment Cost: 3400

### <a name="GAS-7"></a>[GAS-7] Variables/Constants may not need to public set to private

File: https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L20

```
    /// @notice Reference to the $NOTE TOKEN
    ERC20 public note;
```

File: https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L47

```
    /// @notice Reference to the $NOTE TOKEN
    ERC20 public note;
```

File: https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L37

```
    /// @notice Price of one tray in $NOTE
    uint256 public immutable trayPrice;
```
