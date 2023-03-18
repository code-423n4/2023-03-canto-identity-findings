**canto-pfp-protocol/src/ProfilePicture.sol**
- L62/63 - It is not necessary to create a variable in memory if it is only going to be used once, therefore the code could be simplified directly into:
Turnstile(0xEcf044C5B4b867CFda001101c617eCd347095B44).register(tx.origin);

- L100/101 - It is not necessary to create a variable in memory if it is only going to be used once, therefore the code could be simplified directly into:
cidNFT.addressRegistry().getAddress(cidNFTID)


**canto-bio-protocol/src/Bio.sol**
- L35/36/45/46/49/50/57/58 - It is not necessary to create a variable in memory if it is only going to be used once, therefore the code could be simplified directly into:
* Turnstile(0xEcf044C5B4b867CFda001101c617eCd347095B44).register(tx.origin);
* bioTextBytes = bytes(bio[_id]);
* strLines = new string[]((lengthInBytes - 1) / 40 + 1);
* bytesLines[bytesOffset] = bioTextBytes[i];


**canto-namespace-protocol/src/Namespace.sol**
- L111/121 - Instead of using _characterList.length to create a new array, you could directly use numCharacters, which was defined long before and was not modified. This would generate a lower gas expense, by going through the arrangement unnecessarily.

- L83/84/113/114/116/151 - It is not necessary to create a variable in memory if it is only going to be used once, therefore the code could be simplified directly into:
* Turnstile(0xEcf044C5B4b867CFda001101c617eCd347095B44).register(tx.origin);
* SafeTransferLib.safeTransferFrom(note, msg.sender, revenueAddress, 2**(13 - numCharacters) * 1e18);
* nftCharacters[namespaceIDToMint].push(tileData);


**canto-namespace-protocol/src/Tray.sol**
- L112/113/266/267 - It is not necessary to create a variable in memory if it is only going to be used once, therefore the code could be simplified directly into:
* Turnstile(0xEcf044C5B4b867CFda001101c617eCd347095B44).register(tx.origin);
* tileData.characterModifier = uint8(Utils.iteratePRNG(_seed) % 256);


**canto-namespace-protocol/src/Utils.sol**
- L224/225 - _tiles.length is used twice, this could generate less gas costs if an in-memory variable of length was created.
