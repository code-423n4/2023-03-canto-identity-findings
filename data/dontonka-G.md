```diff
@@ -120,10 +122,16 @@ contract Bio is ERC721 {
     /// @param _bio The text to add
     function mint(string calldata _bio) external {
         // We check the length in bytes, so will be higher for UTF-8 characters. But sufficient for this check
-        if (bytes(_bio).length == 0 || bytes(_bio).length > 200) revert InvalidBioLength(bytes(_bio).length);
+        bytes memory bioBytes = bytes(_bio); //@audit (GAS) will probably save gas converting to bytes only once
+        if (bioBytes.length == 0 || bioBytes.length > 200) revert InvalidBioLength(bioBytes.length);
```