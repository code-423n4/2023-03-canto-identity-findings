# No Two step transfer ownership on solmate owned

The protocol use solmate's owned contract `import {Owned} from "solmate/auth/Owned.sol";`. This contract doesn't have a two step transfer pattern.

Recommend considering implementing a two step process where the owner or admin nominates an account and the nominated account needs to call an acceptOwnership() function for the transfer of ownership to fully succeed. This ensures the nominated EOA account is a valid and active account.

# no check for zero address

Below are function which doesn't check if the input address is zero address or not. It's best to check the input in code to minimize risk.

```js
File: Namespace.sol
194:     /// @notice Change the address of the $NOTE token
195:     /// @param _newNoteAddress New address to use
196:     function changeNoteAddress(address _newNoteAddress) external onlyOwner {
197:         address currentNoteAddress = address(note);
198:         note = ERC20(_newNoteAddress);
199:         emit NoteAddressUpdate(currentNoteAddress, _newNoteAddress);
200:     }
201:
202:     /// @notice Change the revenue address
203:     /// @param _newRevenueAddress New address to use
204:     function changeRevenueAddress(address _newRevenueAddress) external onlyOwner {
205:         address currentRevenueAddress = revenueAddress;
206:         revenueAddress = _newRevenueAddress;
207:         emit RevenueAddressUpdated(currentRevenueAddress, _newRevenueAddress);
208:     }
```

# Order the block to top before any assignment

It's a common practice to put the 'block' of check for condition statement in top of function implementation before any other variable assignment.

In `ProfilePicture.sol` the 'block' to check the owner of nftId is executed after a local variable `tokenId`, while it doesn't really related to the 'block' condition. So, moving the 'block' to top and the `tokenId` assignment bellow it would be the best way to do that.

```js
File: ProfilePicture.sol
80:         uint256 tokenId = ++numMinted;
81:         if (ERC721(_nftContract).ownerOf(_nftID) != msg.sender)
82:             revert PFPNotOwnedByCaller(msg.sender, _nftContract, _nftID);
```

# Unspecific compiler version pragma

```solidity
pragma solidity >=0.8.0;
```

Avoid floating pragmas for non-library contracts.

While floating pragmas make sense for libraries to allow them to be included with multiple different versions of applications, it may be a security risk for application implementations.

A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

It is recommended to pin to a concrete compiler version.

# Avoid magic number

Avoid any `magic` number showing inside the code, yet, it's better to define as constant variable if necessary.

```js
File: Bio.sol
49:         uint lines = (lengthInBytes - 1) / 40 + 1;
...
54:         bytes memory bytesLines = new bytes(80);

File: Tray.sol
248:         if (res < 32) {
249:             // Class is 0 in that case
250:             tileData.characterIndex = uint16(charRandValue % NUM_CHARS_EMOJIS);
251:         } else {
252:             tileData.characterIndex = uint16(charRandValue % NUM_CHARS_LETTERS);
253:             if (res < 64) {
254:                 tileData.fontClass = 1;
255:                 tileData.characterIndex = uint16(charRandValue % NUM_CHARS_LETTERS_NUMBERS);
256:             } else if (res < 80) {
257:                 tileData.fontClass = 2;
258:             } else if (res < 96) {
259:                 tileData.fontClass = 3 + uint8((res - 80) / 8);
260:             } else if (res < 104) {
261:                 tileData.fontClass = 5 + uint8((res - 96) / 4);
262:             } else if (res < 108) {
263:                 tileData.fontClass = 7 + uint8((res - 104) / 2);
264:                 if (tileData.fontClass == 7) {
265:                     // Set seed for Zalgo to ensure same characters will be always generated for this tile
266:                     uint256 zalgoSeed = Utils.iteratePRNG(_seed);
267:                     tileData.characterModifier = uint8(zalgoSeed % 256);
268:                 }
269:             } else {
270:                 tileData.fontClass = 9;
271:             }
272:         }

```
