## Summary
### Gas Optimizations List
| Number | Optimization Details | Context |
|:--:|:-------| :-----:|
| [G-01] |Gas overflow during iteration (DoS)| 2 |
| [G-02] |State variables only set in the constructor should be declared immutable| 1 |
| [G-03] |Failure to check the zero address in the constructor causes the contract to be deployed again |3 |
| [G-04] |By updating ``emit``, extra emit usage can be reduced | 1 |
| [G-05] |Emit can be rearranged |4|
| [G-06] |Use ``assembly`` to write _address storage values_ | 2 |
| [G-07] |_Functions guaranteed to revert_ when called by normal users can be marked ``payable``| 6 |
| [G-08] | Multiple ``address``/ID mappings can be combined into a single ``mapping`` of an ``address``/ID to a ``struct``, where appropriate| 2 |
| [G-09] | Use a more recent version of solidity |5 |
| [G-10] |The result of function calls should be cached rather than re-calling the function | 1 |
| [G-11] |Avoid contract existence checks by using low level calls |11 |
| [G-12] |``bytes`` constants are more eficient than ``string`` constans| 1 |
| [G-13] |Change ``public`` function visibility to ``external`` | 3 |
| [G-14] |Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead|15 |
| [G-15] |Setting the _constructor_ to `payable` |4 |
| [G-16] |Use nested if and, avoid multiple check combinations |7 |
| [G-17] |Sort Solidity operations using short-circuit mode | 4 |
| [G-18] |Use ERC721A instead ERC721 |  |
| [G-19] |Optimize names to save gas |All Contract |
| [G-20] |Upgrade Solidity's optimizer | 3 |


### [G-01] Gas overflow during iteration (DoS)

Each iteration of the cycle requires a gas flow. A moment may come when more gas is required than it is allocated to record one block. In this case, all iterations of the loop will fail.

2 results - 2 files:

```diff
canto-namespace-protocol\src\Namespace.sol:

  109      /// @param _characterList The tiles to use for the fusing
  110:     function fuse(CharacterData[] calldata _characterList) external {
+              require(_characterList.length < max _characterListLengt, "max length");

  111:         uint256 numCharacters = _characterList.length;
  112:         if (numCharacters > 13 || numCharacters == 0) revert InvalidNumberOfCharacters(numCharacters);
  113:         uint256 fusingCosts = 2**(13 - numCharacters) * 1e18;
  114:         SafeTransferLib.safeTransferFrom(note, msg.sender, revenueAddress, fusingCosts);
  115:         uint256 namespaceIDToMint = ++nextNamespaceIDToMint;
  116:         Tray.TileData[] storage nftToMintCharacters = nftCharacters[namespaceIDToMint];
  117:         bytes memory bName = new bytes(numCharacters * 33); // Used to convert into a string. Can be 33 times longer than the string at most (longest zalgo characters is 33 bytes)
  118:         uint256 numBytes;
  119:         // Extract unique trays for burning them later on
  120:         uint256 numUniqueTrays;
  121:         uint256[] memory uniqueTrays = new uint256[](_characterList.length);
  122:         for (uint256 i; i < numCharacters; ++i) {
  123:             bool isLastTrayEntry = true;
  124:             uint256 trayID = _characterList[i].trayID;
  125:             uint8 tileOffset = _characterList[i].tileOffset;
  126:             // Check for duplicate characters in the provided list. 1/2 * n^2 loop iterations, but n is bounded to 13 and we do not perform any storage operations

```
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L110


```diff
canto-namespace-protocol\src\Utils.sol:

  222:     function generateSVG(Tray.TileData[] memory _tiles, bool _isTray) internal pure returns (string memory) {
+              require(_tiles.length < max _tilesLengt, "max length");
  223:         string memory innerData;
  224:         uint256 dx = 400 / (_tiles.length + 1);
  225:         for (uint256 i; i < _tiles.length; ++i) {
  226:             innerData = string.concat(
  227:                 innerData,
  228:                 '<text dominant-baseline="middle" text-anchor="middle" y="100" x="',
  229:                 LibString.toString(dx * (i + 1)),
  230:                 '">',
  231:                 string(
  232:                     characterToUnicodeBytes(_tiles[i].fontClass, _tiles[i].characterIndex, _tiles[i].characterModifier)
  233:                 ),
  234:                 "</text>"
  235:             );
  236:             if (_isTray) {
  237:                 innerData = string.concat(
  238:                     innerData,
  239:                     '<rect width="34" height="60" y="70" x="',
  240:                     LibString.toString(dx * (i + 1) - 17),
  241:                     '" stroke="black" stroke-width="1" fill="none"></rect>'
  242:                 );
  243:             }
  244:         }
  245:         return
  246:             string.concat(
  247:                 '<svg xmlns="http://www.w3.org/2000/svg" preserveAspectRatio="xMinYMin meet" viewBox="0 0 400 200"><style>text { font-family: sans-serif; font-size: 30px; }</style>',
  248:                 innerData,
  249:                 "</svg>"
  250:             );
  251:     }

```

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L222


Here are the data available in the covered contracts. The use of this situation in contracts that are not covered will also provide gas optimization.



### [G-02] State variables only set in the constructor should be declared immutable

Avoids a Gsset (20000 gas) in the constructor and replaces the first access in each transaction (Gcoldsload - 2100 gas) and each access thereafter (Gwarmacces - 100 gas) with a PUSH32 (3 gas).


```solidity
canto-pfp-protocol\src\ProfilePicture.sol:

  35:     string public subprotocolName;

```

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L35


### [G-03] Failure to check the zero address in the constructor causes the contract to be deployed again

Zero address control is not performed in the constructor in all 3 contracts within the scope of the audit. Bypassing this check could cause the contract to be deployed by mistakenly entering a zero address. In this case, the contract will need to be redeployed. This means extra gas consumption as contract deployment costs are high.


3 results - 3 files:

```solidity
canto-namespace-protocol\src\Namespace.sol:

  73:     constructor(
  74:         address _tray,
  75:         address _note,
  76:         address _revenueAddress
  77:     ) ERC721("Namespace", "NS") Owned(msg.sender) {
  78:         tray = Tray(_tray);
  79:         note = ERC20(_note);
  80:         revenueAddress = _revenueAddress;
  81:         if (block.chainid == 7700) {
  82:             // Register CSR on Canto mainnnet
  83:             Turnstile turnstile = Turnstile(0xEcf044C5B4b867CFda001101c617eCd347095B44);
  84:             turnstile.register(tx.origin);
  85:         }
  86:     }

```
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L73


```solidity
canto-namespace-protocol\src\Tray.sol:

   97      /// @param _namespaceNFT Address of the Namespace NFT
   98:     constructor(
   99:         bytes32 _initHash,
  100:         uint256 _trayPrice,
  101:         address _revenueAddress,
  102:         address _note,
  103:         address _namespaceNFT
  104:     ) ERC721A("Namespace Tray", "NSTRAY") Owned(msg.sender) {
  105:         lastHash = _initHash;
  106:         trayPrice = _trayPrice;
  107:         revenueAddress = _revenueAddress;
  108:         note = ERC20(_note);
  109:         namespaceNFT = _namespaceNFT;
  110:         if (block.chainid == 7700) {
  111:             // Register CSR on Canto mainnnet
  112:             Turnstile turnstile = Turnstile(0xEcf044C5B4b867CFda001101c617eCd347095B44);
  113:             turnstile.register(tx.origin);
  114:         }
  115:     }

```
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L98


```solidity
canto-pfp-protocol\src\ProfilePicture.sol:

  56      /// @param _subprotocolName Name with which the subprotocol is / will be registered in the registry. Registration will not be performed automatically
  57:     constructor(address _cidNFT, string memory _subprotocolName) ERC721("Profile Picture", "PFP") {
  58:         cidNFT = ICidNFT(_cidNFT);
  59:         subprotocolName = _subprotocolName;
  60:         if (block.chainid == 7700) {
  61:             // Register CSR on Canto mainnnet
  62:             Turnstile turnstile = Turnstile(0xEcf044C5B4b867CFda001101c617eCd347095B44);
  63:             turnstile.register(tx.origin);
  64:         }
  65:     }

```
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L57


### [G-04] By updating ``emit``, extra emit usage can be reduced

In the ``mint`` function in the ``Bio.sol`` contract, the protocol emits two emits (the _mint and bioAdded emits). By updating the emit of Solady's _mint function, the use of one extra emit can be reduced.

Reference: https://github.com/ethereum/go-ethereum/blob/master/params/protocol_params.go#L78-L82

```diff 
canto-bio-protocol/lib/solady/lib/solmate/src/tokens/ERC721.sol:

+ event Transfer(address from, address indexed to, uint256 indexed id, string indexed bio);

+ 157:     function _mint(address to, uint256 id) internal virtual {
+ 158:         require(to != address(0), "INVALID_RECIPIENT");
+ 159: 
+ 160:         require(_ownerOf[id] == address(0), "ALREADY_MINTED");
+ 161: 
+ 162:         // Counter overflow is incredibly unrealistic.
+ 163:         unchecked {
+ 164:             _balanceOf[to]++;
+ 165:         }
+ 166: 
+ 167:         _ownerOf[id] = to;
+ 168: 
+ 169:         emit Transfer(address(0), to, tokenId, _bio );
+ 170:     }


canto-bio-protocol/src/Bio.sol:

23:     event BioAdded(address indexed minter, uint256 indexed nftID, string indexed bio);
  120      /// @param _bio The text to add
  121:     function mint(string calldata _bio) external {
  122:         // We check the length in bytes, so will be higher for UTF-8 characters. But sufficient for this check
  123:         if (bytes(_bio).length == 0 || bytes(_bio).length > 200) revert InvalidBioLength(bytes(_bio).length);
  124:         uint256 tokenId = ++numMinted;
  125:         bio[tokenId] = _bio;
- 126:        _mint(msg.sender, tokenId,);
+ 126:        _mint(msg.sender, tokenId, _bio);

- 127:        emit BioAdded(msg.sender, tokenId, _bio);
  128:     }

```

### [G-05] Emit can be rearranged

The emit can be rearranged in the changeNoteAddress() and changeRevenueAddress() functions. With this arrangement, both the deploy time (~1.4 k * 4 = 5.6k gas per function) and every time the functions are triggered (~16 gas) gas savings will be achieved.

4 results - 2 files:
```solidity
canto-namespace-protocol\src\Tray.sol:

  209      /// @param _newNoteAddress New address to use
  210:     function changeNoteAddress(address _newNoteAddress) external onlyOwner {
  211:         address currentNoteAddress = address(note);
  212:         note = ERC20(_newNoteAddress);
  213:         emit NoteAddressUpdate(currentNoteAddress, _newNoteAddress);
  214:     }
  

  217      /// @param _newRevenueAddress New address to use
  218:     function changeRevenueAddress(address _newRevenueAddress) external onlyOwner {
  219:         address currentRevenueAddress = revenueAddress;
  220:         revenueAddress = _newRevenueAddress;
  221:         emit RevenueAddressUpdated(currentRevenueAddress, _newRevenueAddress);
  222:     }

```



```solidity
canto-namespace-protocol\src\Namespace.sol:

  195      /// @param _newNoteAddress New address to use
  196:     function changeNoteAddress(address _newNoteAddress) external onlyOwner {
  197:         address currentNoteAddress = address(note);
  198:         note = ERC20(_newNoteAddress);
  199:         emit NoteAddressUpdate(currentNoteAddress, _newNoteAddress);
  200:     }


  203      /// @param _newRevenueAddress New address to use
  204:     function changeRevenueAddress(address _newRevenueAddress) external onlyOwner {
  205:         address currentRevenueAddress = revenueAddress;
  206:         revenueAddress = _newRevenueAddress;
  207:         emit RevenueAddressUpdated(currentRevenueAddress, _newRevenueAddress);
  208:     }
  209  }

```

```diff
canto-namespace-protocol\src\Namespace.sol:

  195      /// @param _newNoteAddress New address to use
  196:     function changeNoteAddress(address _newNoteAddress) external onlyOwner {
- 197:         address currentNoteAddress = address(note);
- 198:         note = ERC20(_newNoteAddress);
- 199:         emit NoteAddressUpdate(currentNoteAddress, _newNoteAddress);
+ 199:         emit NoteAddressUpdate(note, _newNoteAddress);
+ 198:         note = ERC20(_newNoteAddress);
  200:     }

```

### [G-06] Use ``assembly`` to write _address storage values_ 

2 results - 4 files:
```solidity
canto-namespace-protocol\src\Namespace.sol:

  73:     constructor(
  80:         revenueAddress = _revenueAddress;


  204:     function changeRevenueAddress(address _newRevenueAddress) external onlyOwner {
  206:         revenueAddress = _newRevenueAddress;

```
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L80


```solidity
canto-namespace-protocol\src\Tray.sol:

   98:     constructor(
  107:         revenueAddress = _revenueAddress;


  218:     function changeRevenueAddress(address _newRevenueAddress) external onlyOwner {
  220:         revenueAddress = _newRevenueAddress;

```
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L107



**Recommendation Code:**
```diff
canto-namespace-protocol\src\Tray.sol:

   98:     constructor(
- 107:         revenueAddress = _revenueAddress;
+                  assembly {                      
+                      sstore(vault. revenueAddress, _revenueAddress)
+                  }                               
          
```


### [G-07] _Functions guaranteed to revert_ when called by normal users can be marked ``payable``

If a function modifier such as ``onlyOwner`` is used, the function will revert if a normal user tries to pay the function. Marking the function as ``payable`` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

6 results - 2 files:
```solidity
canto-namespace-protocol\src\Namespace.sol:

  196:     function changeNoteAddress(address _newNoteAddress) external onlyOwner {

  204:     function changeRevenueAddress(address _newRevenueAddress) external onlyOwner {

```
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L196


```solidity
canto-namespace-protocol\src\Tray.sol:

  210:     function changeNoteAddress(address _newNoteAddress) external onlyOwner {

  218:     function changeRevenueAddress(address _newRevenueAddress) external onlyOwner {

  225:     function endPrelaunchPhase() external onlyOwner {

```
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L210


### [G-08] Multiple ``address``/ID mappings can be combined into a single ``mapping`` of an ``address``/ID to a ``struct``, where appropriate

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations.


2 results - 1 file:
```solidity
canto-namespace-protocol\src\Namespace.sol:

  43:     mapping(string => uint256) public nameToToken;
  46:     mapping(uint256 => string) public tokenToName;

```
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L43-L46



### [G-09] Use a more recent version of solidity

Use a solidity version of at least 0.8.0 to get overflow protection without SafeMath
Use a solidity version of at least 0.8.2 to get simple compiler automatic inlining
Use a solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads
Use a solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than revert()/require() strings
Use a solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value
In 0.8.15 the conditions necessary for inlining are relaxed. Benchmarks show that the change significantly decreases the bytecode size (which impacts the deployment cost) while the effect on the runtime gas usage is smaller.

In 0.8.17 prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call; Simplify the starting offset of zero-length operations to zero. More efficient overflow checks for multiplication.

 5 results - 5 files:
```solidity
canto-bio-protocol/src/Bio.sol:

  2: pragma solidity >=0.8.0;

```
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L2


```solidity
canto-namespace-protocol/src/Namespace.sol:
  
  2: pragma solidity >=0.8.0;

```
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L2


```solidity
canto-namespace-protocol/src/Tray.sol:

  2: pragma solidity >=0.8.0;

```
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L2


```solidity
canto-namespace-protocol/src/Utils.sol:
  
  2: pragma solidity >=0.8.0;

```
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L2  


```solidity
canto-pfp-protocol/src/ProfilePicture.sol:
  
  2: pragma solidity >=0.8.0;
  
``` 
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L2

### [G-10] The result of function calls should be cached rather than re-calling the function


```diff
canto-namespace-protocol\src\Tray.sol:
 
  245:     function _drawing(uint256 _seed) private pure returns (TileData memory tileData) {

  247:         uint256 charRandValue = Utils.iteratePRNG(_seed); // Iterate PRNG to not have any biasedness / correlation between random numbers

  265:                     // Set seed for Zalgo to ensure same characters will be always generated for this tile
- 266:                     uint256 zalgoSeed = Utils.iteratePRNG(_seed);
+ 266:                     uint256 zalgoSeed = charRandValue;
  267:                     tileData.characterModifier = uint8(zalgoSeed % 256);
  268                  }

```


### [G-11] Avoid contract existence checks by using low level calls

Prior to 0.8.10 the compiler inserted extra code, including ``EXTCODESIZE`` (**100 gas**), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

12 results - 4 files:
```solidity
canto-bio-protocol\src\Bio.sol:

  // @audit register()
  36:    turnstile.register(tx.origin);

```
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L36


```solidity
canto-namespace-protocol\src\Namespace.sol:

  // @audit register() 
  84:    turnstile.register(tx.origin);

  // @audit getTile()
  133:   Tray.TileData memory tileData = tray.getTile(trayID, tileOffset); // Will revert if tileOffset is too high

  // @audit ownerOf()
  156:   address trayOwner = tray.ownerOf(trayID);

  // @audit getApproved()
  159:   tray.getApproved(trayID) != msg.sender &&
  
  // @audit isApprovedForAll()
  160:   !tray.isApprovedForAll(trayOwner, msg.sender)

```
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L84


```solidity
canto-namespace-protocol\src\Tray.sol:

  // @audit register() 
  113:   turnstile.register(tx.origin);

```
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L113


```solidity
canto-pfp-protocol\src\ProfilePicture.sol:

  // @audit register()  
  63:    turnstile.register(tx.origin);

  // @audit tokenURI()
  73:    return ERC721(nftContract).tokenURI(nftID);

  // @audit ownerOf()
  81:    if (ERC721(_nftContract).ownerOf(_nftID) != msg.sender)

  // @audit addressRegistry()
  100:   IAddressRegistry addressRegistry = cidNFT.addressRegistry();

  // @audit getAddress(), ownerOf() 
  101:   if (cidNFTID == 0 || addressRegistry.getAddress(cidNFTID) != ERC721(nftContract).ownerOf(nftID)) {

```
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L63


### [G-12] ``bytes`` constants are more eficient than ``string`` constans

If the data can fit in 32 bytes, the bytes32 data type can be used instead of bytes or strings, as it is less robust in terms of robustness.


```solidity
canto-pfp-protocol\src\ProfilePicture.sol:
  
  35:     string public subprotocolName;

```

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L35


### [G-13] Change ``public`` function visibility to ``external``

Since the following public functions are not called from within the contract, their visibility can be made external for gas optimization.

3 results - 3 files:
```solidity
canto-bio-protocol\src\Bio.sol:

  43:     function tokenURI(uint256 _id) public view override returns (string memory) {

```
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L43


```solidity
canto-namespace-protocol\src\Namespace.sol:

  90:     function tokenURI(uint256 _id) public view override returns (string memory) {

```
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L90


```solidity
canto-namespace-protocol\src\Tray.sol:

  119:     function tokenURI(uint256 _id) public view override returns (string memory) {

```
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L119



### [G-14] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contracts gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html 

Use a larger size then downcast where needed.

15 result - 2 files:
```solidity
canto-namespace-protocol\src\Tray.sol:

  // @audit uint8 fontClass
  259:     tileData.fontClass = 3 + uint8((res - 80) / 8);
  261:     tileData.fontClass = 5 + uint8((res - 96) / 4);
  263:     tileData.fontClass = 7 + uint8((res - 104) / 2);


  // @audit uint8 characterModifier
  267:     tileData.characterModifier = uint8(zalgoSeed % 256);

```
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L259


```solidity
canto-namespace-protocol\src\Utils.sol:

   // @audit uint16 _characterIndex
   85:     byteOffset = _characterIndex * 3;
   86:     supportsSkinToneModifier = _characterIndex >= EMOJIS_LE_THREE_BYTES - EMOJIS_MOD_SKIN_TONE_THREE_BYTES;
   89:     byteOffset = EMOJIS_BYTE_OFFSET_FOUR_BYTES + (_characterIndex - EMOJIS_LE_THREE_BYTES) * 4;
   93:     byteOffset = EMOJIS_BYTE_OFFSET_SIX_BYTES + (_characterIndex - EMOJIS_LE_FOUR_BYTES) * 6;
   96:     byteOffset = EMOJIS_BYTE_OFFSET_SEVEN_BYTES + (_characterIndex - EMOJIS_LE_SIX_BYTES) * 7;
   99:     byteOffset = EMOJIS_BYTE_OFFSET_EIGHT_BYTES + (_characterIndex - EMOJIS_LE_SEVEN_BYTES) * 8;
  102:     byteOffset = EMOJIS_BYTE_OFFSET_FOURTEEN_BYTES + (_characterIndex - EMOJIS_LE_EIGHT_BYTES) * 14;

  // @audit uint8 asciiStartingIndex
  135:     return abi.encodePacked(bytes1(asciiStartingIndex + uint8(_characterIndex)));

  // @audit uint8 asciiStartingIndex, _characterIndex
  145:     bytes memory character = abi.encodePacked(bytes1(asciiStartingIndex + uint8(_characterIndex)));
  273:     uint8 lastByteSum = lastByteVal + _characterIndex;
  277:     lastByteVal = 128 + (lastByteSum % 192);

```
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L85



### [G-15] Setting the _constructor_ to `payable`

You can cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor payable. Making the constructor payable eliminates the need for an initial check of ```msg.value == 0``` and saves ```13 gas``` on deployment with no security risks.

4 results - 4 files:
```solidty
canto-bio-protocol\src\Bio.sol:

  32:     constructor() ERC721("Biography", "Bio") {

```
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L32


```solidty
canto-namespace-protocol\src\Namespace.sol:

  73:     constructor(

```
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L73


```solidty
canto-namespace-protocol\src\Tray.sol:

  98:     constructor(

```
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L98


```solidty
canto-pfp-protocol\src\ProfilePicture.sol:

  57:     constructor(address _cidNFT, string memory _subprotocolName) ERC721("Profile Picture", "PFP") {

```
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L57



**Recommendation:**
Set the constructor to ```payable```


### [G-16] Use nested if and, avoid multiple check combinations

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

7 results - 3 files:
```solidity
canto-bio-protocol\src\Bio.sol:

  60:     if ((i > 0 && (i + 1) % 40 == 0) || prevByteWasContinuation || i == lengthInBytes - 1) {

  65:     if (nextCharacter & 0xC0 == 0x80) {

```
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L60


```solidity
canto-namespace-protocol\src\Namespace.sol:

  136:    if (tileData.fontClass != 0 && _characterList[i].skinToneModifier != 0) {


  157:    if (
  158:        trayOwner != msg.sender &&
  159:        tray.getApproved(trayID) != msg.sender &&
  160:        !tray.isApprovedForAll(trayOwner, msg.sender)
  161:    ) revert CallerNotAllowedToFuse();


  186:    if (nftOwner != msg.sender && getApproved[_id] != msg.sender && !isApprovedForAll[nftOwner][msg.sender])

```
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L136


```solidity
canto-namespace-protocol\src\Tray.sol:

  175:    if (
  176:        namespaceNFT != msg.sender &&
  177:        trayOwner != msg.sender &&
  178:        getApproved(_id) != msg.sender &&
  179:        !isApprovedForAll(trayOwner, msg.sender)
  180:    ) revert CallerNotAllowedToBurn();


  184:    if (numPrelaunchMinted != type(uint256).max && _id <= numPrelaunchMinted)

```
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L175-LL180


### [G-17] Sort Solidity operations using short-circuit mode

Short-circuiting is a solidity contract development model that uses ```OR/AND``` logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

```
//f(x) is a low gas cost operation 
//g(y) is a high gas cost operation 

//Sort operations with different gas costs as follows 
f(x) || g(y) 
f(x) && g(y)
```
4 results - 3 files:
```solidity
canto-bio-protocol\src\Bio.sol:

  60:    if ((i > 0 && (i + 1) % 40 == 0) || prevByteWasContinuation || i == lengthInBytes - 1) {

 123:    if (bytes(_bio).length == 0 || bytes(_bio).length > 200) revert InvalidBioLength(bytes(_bio).length);

```
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L60


```solidity
canto-namespace-protocol\src\Namespace.sol:

  112:   if (numCharacters > 13 || numCharacters == 0) revert InvalidNumberOfCharacters(numCharacters);

```
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L112


```solidity
canto-pfp-protocol\src\ProfilePicture.sol:

  101:   if (cidNFTID == 0 || addressRegistry.getAddress(cidNFTID) != ERC721(nftContract).ownerOf(nftID)) {

```
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L101

### [G-18] Use ERC721A instead ERC721

ERC721A standard, ERC721A is an improvement standard for ERC721 tokens. It was proposed by the Azuki team and used for developing their NFT collection. Compared with ERC721, ERC721A is a more gas-efficient standard to mint a lot of of NFTs simultaneously. It allows developers to mint multiple NFTs at the same gas price. This has been a great improvement due to Ethereum’s sky-rocketing gas fee.

Reference: https://nextrope.com/erc721-vs-erc721a-2/



### [G-19] Optimize names to save gas

Contracts most called functions could simply save gas by function ordering via ```Method ID```. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because ```22 gas``` are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions. 

**Context:** 
All Contracts


**Recommendation:** 
Find a lower ```method ID``` name for the most called functions for example Call() vs. Call1() is cheaper by ```22 gas```
For example, the function IDs in the ```Namespace.sol``` contract will be the most used; A lower method ID may be given.

**Proof of Consept:**
https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92

Namespace.sol function names can be named and sorted according to METHOD ID

```js
Sighash   |   Function Signature
========================
c87b56dd  =>  tokenURI(uint256)
11f839e2  =>  fuse(CharacterData[])
42966c68  =>  burn(uint256)
21496de6  =>  changeNoteAddress(address)
92f8b9a3  =>  changeRevenueAddress(address)

```


### [G-20] Upgrade Solidity's optimizer

Make sure Solidity’s optimizer is enabled. It reduces gas costs. If you want to gas optimize for contract deployment (costs less to deploy a contract) then set the Solidity optimizer at a low number. If you want to optimize for run-time gas costs (when functions are called on a contract) then set the optimizer to a high number.


3 results - 3 files:

```js
canto-bio-protocol\foundry.toml:

  2: optimizer = true
  3: optimizer_runs = 1_000

```


```js
canto-namespace-protocol\foundry.toml:

  2: optimizer = true
  3: optimizer_runs = 1_000

```


```js
canto-pfp-protocol\foundry.toml:

  2: optimizer = true
  3: optimizer_runs = 1_000

```


