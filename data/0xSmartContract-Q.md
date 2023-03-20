## Summary
### Issues List
| Number |Issues Details|Context|
|:--:|:-------|:--:|
|[L-01]|Protect your NFT from copying in fork| 1 |
|[L-02]|No Check if OnErc721Received is implemented | 1 |
|[L-03]|Project Upgrade and Stop Scenario should be  |  |
|[L-04]|Add to Event-Emit for constructors|4 |
|[L-05]|Critical Address Changes Should Use Check Pattern| 2 |
|[L-06]|Solmate's SafeTransferLib doesn't check whether the ERC20 contract exists |2 |
|[N-07]|Add EIP-2981 NFT Royalty Standart Support|  |
|[N-08]|Use SMTChecker|  |
|[N-09]|Constants on the left are better| 3 |
|[N-10] |Function Calls in Loop Could Lead to Denial of Service due to Array length not being checked|2|
|[N-11] |Take advantage of Custom Error's return value property |3|
|[N-12] |`Function writing` that does not comply with the `Solidity Style Guide`| All Contracts |
|[N-13] |Tokens accidentally sent to the contract cannot be recovered| 1|
|[N-14] |Use a more recent version of Solidity|All Contracts |

Total 14 issues


### [L-01] Protect your NFT from copying in fork

Occasionally, forks can happen on different blockchains.

There may be forked versions of Blockchains, which could cause confusion and lead to scams as duplicated NFT assets enter the market, then it could result in duplication of non-fungible tokens (NFTs) and there's likely to be some level of confusion around which assets are 'official' or 'authentic.’

A forked Blockchains, chain will thus have duplicated deeds that point to the same tokenURI

About Replay Attack:
https://twitter.com/elerium115/status/1558471934924431363?s=20&t=RRheaYJwo-GmSnePwofgag

The most important part here is NFT's tokenURI detail. If the update I suggested below is not done, Duplicate NFTs will appear as a result, potentially leading to confusion and scams.

### Tools Used
Manual Code Review


### Recommended Mitigation Steps

```diff
src/CidNFT.sol:
  135      /// @return tokenURI The URI of the queried token (path to a JSON file)
  136:     function tokenURI(uint256 _id) public view override returns (string memory) {
+ 	 if (block.chainid != 1) revert wrongChain(); // Ethereum Chain ID : 1  - Polygon Chain ID : 137
  137:         if (ownerOf[_id] == address(0))
  138:             // According to ERC721, this revert for non-existing tokens is required
  139:             revert TokenNotMinted(_id);
  140:         return string(abi.encodePacked(baseURI, _id, ".json"));
  141:     }

4 results - 4 files

canto-bio-protocol/src/Bio.sol:
  42      /// @param _id ID to query for
  43:     function tokenURI(uint256 _id) public view override returns (string memory) {
  44          if (_ownerOf[_id] == address(0)) revert TokenNotMinted(_id);
+ 	 if (block.chainid != 7700) revert wrongChain(); // Canto Chain 7700


canto-namespace-protocol/src/Namespace.sol:
  89      /// @param _id ID to query for
  90:     function tokenURI(uint256 _id) public view override returns (string memory) {
  91          if (_ownerOf[_id] == address(0)) revert TokenNotMinted(_id);
+ 	 if (block.chainid != 7700) revert wrongChain(); // Canto Chain 7700


canto-namespace-protocol/src/Tray.sol:
  118      /// @param _id ID to query for
  119:     function tokenURI(uint256 _id) public view override returns (string memory) {
  120          if (!_exists(_id)) revert TrayNotMinted(_id);
+ 	 if (block.chainid != 7700) revert wrongChain(); // Canto Chain 7700


canto-pfp-protocol/src/ProfilePicture.sol:
  69      /// @dev Reverts if PFP is no longer owned by owner of associated CID NFT
  70:     function tokenURI(uint256 _id) public view override returns (string memory) {
  71          (address nftContract, uint256 nftID) = getPFP(_id);
+ 	 if (block.chainid != 7700) revert wrongChain(); // Canto Chain 7700



```


### [L-02]  No Check if OnErc721Received is implemented

` Bio.mint()` and `ProfilePicture.mint()`will mint a NFT . When minting a NFT, the function does not check if a receiving contract implements onERC721Received().
`msg.sender` can be contract address.

The intention behind this function is to check if the address receiving the NFT, if it is a contract, implements onERC721Received(). Thus, there is no check whether the receiving address supports ERC-721 tokens and position could be not transferrable in some cases.

Following shows that _mint is used instead of _safeMint

```solidity

canto-bio-protocol/src/Bio.sol:
  120      /// @param _bio The text to add
  121:     function mint(string calldata _bio) external {
  122:         // We check the length in bytes, so will be higher for UTF-8 characters. But sufficient for this check
  123:         if (bytes(_bio).length == 0 || bytes(_bio).length > 200) revert InvalidBioLength(bytes(_bio).length);
  124:         uint256 tokenId = ++numMinted;
  125:         bio[tokenId] = _bio;
  126:         _mint(msg.sender, tokenId);
  127:         emit BioAdded(msg.sender, tokenId, _bio);
  128:     }
  129  }


canto-pfp-protocol/src/ProfilePicture.sol:
  78      /// @param _nftID The nft ID to reference
  79:     function mint(address _nftContract, uint256 _nftID) external {
  80:         uint256 tokenId = ++numMinted;
  81:         if (ERC721(_nftContract).ownerOf(_nftID) != msg.sender)
  82:             revert PFPNotOwnedByCaller(msg.sender, _nftContract, _nftID);
  83:         ProfilePictureData storage pictureData = pfp[tokenId];
  84:         pictureData.nftContract = _nftContract;
  85:         pictureData.nftID = _nftID;
  86:         _mint(msg.sender, tokenId);
  87:         emit PfpAdded(msg.sender, tokenId, _nftContract, _nftID);
  88:     }

```

Recommendation
Consider using _safeMint instead of _mint



### [L-03] Project Upgrade and Stop Scenario should be

At the start of the project, the system may need to be stopped or upgraded, I suggest you have a script beforehand and add it to the documentation.
This can also be called an " EMERGENCY STOP (CIRCUIT BREAKER) PATTERN ".

https://github.com/maxwoe/solidity_patterns/blob/master/security/EmergencyStop.sol

### [L-04] Add to Event-Emit for constructors

Monitoring the amount of Ether coming to the contract outside the chain is an important initialize data

```solidity
4 results

canto-bio-protocol/src/Bio.sol:
  31      /// @notice Initiates CSR on mainnet
  32:     constructor() ERC721("Biography", "Bio") {
  33:         if (block.chainid == 7700) {
  34:             // Register CSR on Canto mainnnet
  35:             Turnstile turnstile = Turnstile(0xEcf044C5B4b867CFda001101c617eCd347095B44);
  36:             turnstile.register(tx.origin);
  37:         }
  38:     }

canto-namespace-protocol/src/Namespace.sol:
  72      /// @param _revenueAddress Adress to send the revenue to
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



canto-namespace-protocol/src/Tray.sol:
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

canto-pfp-protocol/src/ProfilePicture.sol:
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


### [L-05] Critical Address Changes Should Use Check Pattern

From time to time, 0 addresses or wrong token addresses can be identified by mistake

```solidity

canto-namespace-protocol/src/Namespace.sol:
  195      /// @param _newNoteAddress New address to use
  196:     function changeNoteAddress(address _newNoteAddress) external onlyOwner {
  197:         address currentNoteAddress = address(note);
  198:         note = ERC20(_newNoteAddress);
  199:         emit NoteAddressUpdate(currentNoteAddress, _newNoteAddress);
  200:     }
  201  

canto-namespace-protocol/src/Tray.sol:
  209      /// @param _newNoteAddress New address to use
  210:     function changeNoteAddress(address _newNoteAddress) external onlyOwner {
  211:         address currentNoteAddress = address(note);
  212:         note = ERC20(_newNoteAddress);
  213:         emit NoteAddressUpdate(currentNoteAddress, _newNoteAddress);
  214:     }


```



### [L-06] Solmate's SafeTransferLib doesn't check whether the ERC20 contract exists

Solmate's SafeTransferLib, which is often used to interact with non-compliant/unsafe ERC20 tokens, does not check whether the ERC20 contract exists. The following code will not revert in case the token doesn't exist (yet).

This is stated in the Solmate library:
https://github.com/transmissions11/solmate/blob/main/src/utils/SafeTransferLib.sol#L9



```solidity
2 results

canto-namespace-protocol/src/Namespace.sol:
  114:         SafeTransferLib.safeTransferFrom(note, msg.sender, revenueAddress, fusingCosts);

canto-namespace-protocol/src/Tray.sol:

  157:          SafeTransferLib.safeTransferFrom(note, msg.sender, revenueAddress, _amount * trayPrice);


```

https://twitter.com/zarfsec/status/1595070426782535681?t=Hu-p5n6afaQS6RvdfClwAg&s=19

### Recommended Mitigation Steps

Add a contract exist control in functions;
```js
pragma solidity >=0.8.0;

function isContract(address _addr) private returns (bool isContract) {
    isContract = _addr.code.length > 0;
}

```
### [N-07] Add EIP-2981 NFT Royalty Standart Support

Consider adding EIP-2981 NFT Royalty Standard to the project

https://eips.ethereum.org/EIPS/eip-2981


Royalty (Copyright – EIP 2981):

Fixed % royalties: For example, 6% of all sales go back to artists
Declining royalties: There may be a continuous decline in sales based on time or any other variable.
Dynamic royalties : Varies over time or sales amount
Upgradeable royalties: Allows a legal entity or NFT owner to change any copyright
Incremental royalties: No royalties, for example when sold for less than $100
Managed royalties : Funds are owned by a DAO, imagine the recipient is a DAO treasury
Royalties to different people : Collectors and artists can even use royalties, not specific to a particular personality


### [N-08] Use SMTChecker

The *highest* tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification.

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19

### [N-09] Constants on the left are better

If you use the constant first you support structures that veil programming errors. And one should only produce code either to add functionality, fix an programming error or trying to support structures to avoid programming errors (like design patterns). 

https://www.moserware.com/2008/01/constants-on-left-are-better-but-this.html

```solidity

3 results - 2 files

canto-bio-protocol/src/Bio.sol:
  123:         if (bytes(_bio).length == 0 || bytes(_bio).length > 200) revert InvalidBioLength(bytes(_bio).length);
  160:         if (bytes(_bio).length == 0 || bytes(_bio).length > 200) revert InvalidBioLength(bytes(_bio).length);

canto-pfp-protocol/src/ProfilePicture.sol:
  101:         if (cidNFTID == 0 || addressRegistry.getAddress(cidNFTID) != ERC721(nftContract).ownerOf(nftID)) {
```



### [N-10] Function Calls in Loop Could Lead to Denial of Service due to Array length not being checked


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

**Recommendation:**
Function calls made in unbounded loop are error-prone with potential resource exhaustion as it can trap the contract due to gas limitations or failed transactions. Consider bounding the loop length if the array is expected to be growing and/or handling a huge list of elements to avoid unnecessary gas wastage and denial of service.


### [N-11] Take advantage of Custom Error's return value property

An important feature of Custom Error is that values such as address, tokenID, msg.value can 
be written inside the `()` sign, this kind of approach provides a serious advantage in 
debugging and examining the revert details of dapps such as tenderly.

For Example;

```solidity
3 results - 1 file

canto-namespace-protocol/src/Namespace.sol:
  137:                 revert CannotFuseCharacterWithSkinTone();
  161:                 ) revert CallerNotAllowedToFuse();
  187:             revert CallerNotAllowedToBurn();

```


### [N-12] `Function writing` that does not comply with the `Solidity Style Guide`

**Context:**
All Contracts

**Description:**
Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier.But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered:

 constructor
receive function (if exists)
fallback function (if exists)
external
public
internal
private
within a grouping, place the view and pure functions last

### [N-13] Tokens accidentally sent to the contract cannot be recovered


**Contex**
canto-bio-protocol/src/Bio.sol

It can't be recovered if the tokens accidentally arrive at the contract address, which has happened to many popular projects, so I recommend adding a recovery code to your critical contracts.

### Recommended Mitigation Steps

Add this code:

```solidity
 /**
  * @notice Sends ERC20 tokens trapped in contract to external address
  * @dev Onlyowner is allowed to make this function call
  * @param account is the receiving address
  * @param externalToken is the token being sent
  * @param amount is the quantity being sent
  * @return boolean value indicating whether the operation succeeded.
  *
 */
  function rescueERC20(address account, address externalToken, uint256 amount) public onlyOwner returns (bool) {
    IERC20(externalToken).transfer(account, amount);
    return true;
  }
}

```


### [N-14] Use a more recent version of Solidity

**Context:**
All contracts

**Description:**
For security, it is best practice to use the latest Solidity version.
For the security fix list in the versions;
https://github.com/ethereum/solidity/blob/develop/Changelog.md


**Recommendation:**
Old version of Solidity is used `(0.8.0)`, newer version can be used `(0.8.17)` 
