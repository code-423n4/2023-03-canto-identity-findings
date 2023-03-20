```diff
@@ -120,10 +122,16 @@ contract Bio is ERC721 {
     /// @param _bio The text to add
     function mint(string calldata _bio) external {
         // We check the length in bytes, so will be higher for UTF-8 characters. But sufficient for this check
-        if (bytes(_bio).length == 0 || bytes(_bio).length > 200) revert InvalidBioLength(bytes(_bio).length);
+        bytes memory bioBytes = bytes(_bio); //@audit (GAS) will probably save gas converting to bytes only once
+        if (bioBytes.length == 0 || bioBytes.length > 200) revert InvalidBioLength(bioBytes.length);
```
```diff
diff --git a/canto-namespace-protocol/src/Namespace.sol b/canto-namespace-protocol/src/Namespace.sol
index 484d049..8971cc8 100644
--- a/canto-namespace-protocol/src/Namespace.sol
+++ b/canto-namespace-protocol/src/Namespace.sol
@@ -118,7 +118,7 @@ contract Namespace is ERC721, Owned {
         uint256 numBytes;
         // Extract unique trays for burning them later on
         uint256 numUniqueTrays;
-        uint256[] memory uniqueTrays = new uint256[](_characterList.length);
+        uint256[] memory uniqueTrays = new uint256[](_characterList.length); //@audit (GAS) could re-use numCharacters instead of _characterList.length
         for (uint256 i; i < numCharacters; ++i) {
             bool isLastTrayEntry = true;
             uint256 trayID = _characterList[i].trayID;
```