```diff
diff --git a/canto-bio-protocol/src/Bio.sol b/canto-bio-protocol/src/Bio.sol
index 6d5757e..7ef6ed7 100644
--- a/canto-bio-protocol/src/Bio.sol
+++ b/canto-bio-protocol/src/Bio.sol
@@ -1,5 +1,5 @@
 // SPDX-License-Identifier: GPL-3.0-only
-pragma solidity >=0.8.0;
+pragma solidity >=0.8.12; // @audit (QA) 8.12 is required since using string.concat
@@ -120,10 +122,16 @@ contract Bio is ERC721 {
         uint256 tokenId = ++numMinted;
         bio[tokenId] = _bio;
-        _mint(msg.sender, tokenId);
+        _mint(msg.sender, tokenId); //@audit (QA) should _safeMint be used instead in case msg.sender is a contract which doesn't support ERC721? Otherwise if this is strictly for EOA should there be a require to prevent msg.sender to be a contract (require(msg.sender.code.length == 0))?
         emit BioAdded(msg.sender, tokenId, _bio);
     }
 }
```
```diff
diff --git a/canto-pfp-protocol/src/ProfilePicture.sol b/canto-pfp-protocol/src/ProfilePicture.sol
index 54d814a..a7bdb56 100644
--- a/canto-pfp-protocol/src/ProfilePicture.sol
+++ b/canto-pfp-protocol/src/ProfilePicture.sol
@@ -83,7 +83,7 @@ contract ProfilePicture is ERC721 {
         ProfilePictureData storage pictureData = pfp[tokenId];
         pictureData.nftContract = _nftContract;
         pictureData.nftID = _nftID;
-        _mint(msg.sender, tokenId);
+        _mint(msg.sender, tokenId); //@audit (QA) same question as for Bio contract.
         emit PfpAdded(msg.sender, tokenId, _nftContract, _nftID);
     }
```