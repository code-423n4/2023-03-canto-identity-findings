## QA
---

### Function Visibility [1]

- Order of Functions: Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. Functions should be grouped according to their visibility and ordered: constructor, receive function (if exists), fallback function (if exists), public, external, internal, private. Within a grouping, place the view and pure functions last.

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol

```solidity
// external function coming before public one
79:    function mint(address _nftContract, uint256 _nftID) external {
```

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol

```solidity
// external views should come after all external non-views
195:    function getTile(uint256 _trayId, uint8 _tileOffset) external view returns (TileData memory tileData) {
203:    function getTiles(uint256 _trayId) external view returns (TileData[TILES_PER_TRAY] memory tileData) {

// internal function coming after a private one
276:    function _startTokenId() internal pure override returns (uint256) {
```

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-identity-protocol/src/CidNFT.sol

```solidity
// public functions coming after external ones
190:    function add(
274:    function remove(
439:    function transferFrom(
```

---

### natSpec missing [2]

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol

```solidity
40:    event PfpAdded(

50:    error TokenNotMinted(uint256 tokenID);

51:    error PFPNoLongerOwnedByOriginalOwner(uint256 tokenID);

52:    error PFPNotOwnedByCaller(address caller, address nftContract, uint256 nftID);

// @return missing
70:    function tokenURI(uint256 _id) public view override returns (string memory) {
```

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol

```solidity
12:    error EmojiDoesNotSupportSkinToneModifier(uint16 emojiIndex);

13:    error InvalidSkinToneModifierProvided(uint256 characterModifier);

// @return missing
73:    function characterToUnicodeBytes(
222:    function generateSVG(Tray.TileData[] memory _tiles, bool _isTray) internal pure returns (string memory) {
267:    function _getUtfSequence(bytes memory _startingSequence, uint8 _characterIndex)
```

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol

```solidity
78:    event RevenueAddressUpdated(address indexed oldRevenueAddress, address indexed newRevenueAddress);

79:    event NoteAddressUpdate(address indexed oldNoteAddress, address indexed newNoteAddress);

80:    event PrelaunchEnded();

85:    error CallerNotAllowedToBurn();

86:    error TrayNotMinted(uint256 tokenID);

87:    error OnlyOwnerCanMintPreLaunch();

88:    error MintExceedsPreLaunchAmount();

89:    error PrelaunchTrayCannotBeUsedAfterPrelaunch(uint256 startTokenId);

90:    error PrelaunchAlreadyEnded();

// @return missing
119:    function tokenURI(uint256 _id) public view override returns (string memory) {
195:    function getTile(uint256 _trayId, uint8 _tileOffset) external view returns (TileData memory tileData) {
203:    function getTiles(uint256 _trayId) external view returns (TileData[TILES_PER_TRAY] memory tileData) {

231:    function _beforeTokenTransfers(

245:    function _drawing(uint256 _seed) private pure returns (TileData memory tileData) {
```

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol

```solidity
54:    event NamespaceFused(address indexed fuser, uint256 indexed namespaceId, string indexed name);

55:    event RevenueAddressUpdated(address indexed oldRevenueAddress, address indexed newRevenueAddress);

56:    event NoteAddressUpdate(address indexed oldNoteAddress, address indexed newNoteAddress);

61:    error CallerNotAllowedToFuse();

62:    error CallerNotAllowedToBurn();

63:    error InvalidNumberOfCharacters(uint256 numCharacters);

64:    error FusingDuplicateCharactersNotAllowed();

65:    error NameAlreadyRegistered(uint256 nftID);

66:    error TokenNotMinted(uint256 tokenID);

67:    error CannotFuseCharacterWithSkinTone();

// @return missing
90:    function tokenURI(uint256 _id) public view override returns (string memory) {
```

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-identity-protocol/src/SubprotocolRegistry.sol

```solidity
45:    event SubprotocolRegistered(

58:    error SubprotocolAlreadyExists(string name, address owner);

59:    error NoTypeSpecified(string name);

60:    error NotANFT(address passedAddress);
```

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-identity-protocol/src/CidNFT.sol

```solidity
100:    event OrderedDataAdded(

106:    event PrimaryDataAdded(uint256 indexed cidNFTID, string indexed subprotocolName, uint256 subprotocolNFTID);

107:    event ActiveDataAdded(

113:    event OrderedDataRemoved(

119:    event PrimaryDataRemoved(uint256 indexed cidNFTID, string indexed subprotocolName, uint256 subprotocolNFTID);

120:    event ActiveDataRemoved(uint256 indexed cidNFTID, string indexed subprotocolName, uint256 subprotocolNFTID);

125:    error TokenNotMinted(uint256 tokenID);

126:    error SubprotocolDoesNotExist(string subprotocolName);

127:    error NFTIDZeroDisallowedForSubprotocols();

128:    error AssociationTypeNotSupportedForSubprotocol(AssociationType associationType, string subprotocolName);

129:    error NotAuthorizedForCIDNFT(address caller, uint256 cidNFTID, address cidNFTOwner);

130:    error NotAuthorizedForSubprotocolNFT(address caller, uint256 subprotocolNFTID);

131:    error ActiveArrayAlreadyContainsID(uint256 cidNFTID, string subprotocolName, uint256 nftIDToAdd);

132:    error OrderedValueNotSet(uint256 cidNFTID, string subprotocolName, uint256 key);

133:    error PrimaryValueNotSet(uint256 cidNFTID, string subprotocolName);

134:    error ActiveArrayDoesNotContainID(uint256 cidNFTID, string subprotocolName, uint256 nftIDToRemove);
```


https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-identity-protocol/src/AddressRegistry.sol

```solidity
27:    event CIDNFTAdded(address indexed user, uint256 indexed cidNFTID);

28:    event CIDNFTRemoved(address indexed user, uint256 indexed cidNFTID);

33:    error NFTNotOwnedByUser(uint256 cidNFTID, address caller);

34:    error NoCIDNFTRegisteredForUser(address caller);

35:    error RemoveOnTransferOnlyCallableByCIDNFT();
```

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol

```solidity
// @return missing
43:    function tokenURI(uint256 _id) public view override returns (string memory) {
```

---

### State variable and function names [3]

- Variables should be named according to their specifications
- private and internal `variables` should preppend with `underline`
- private and internal `functions` should preppend with `underline`
- constant state variables should be UPPER_CASE

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol

```solidity
// private and internal `variables` should preppend with `underline`
14:    ICidNFT private immutable cidNFT;
32:    mapping(uint256 => ProfilePictureData) private pfp;
```

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol

```solidity
// private and internal `functions` should preppend with `underline`
73:    function characterToUnicodeBytes(
222:    function generateSVG(Tray.TileData[] memory _tiles, bool _isTray) internal pure returns (string memory) {
256:    function iteratePRNG(uint256 _currState) internal pure returns (uint256 iteratedState) {
```

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol

```solidity
// private and internal `variables` should preppend with `underline`
44:    address private revenueAddress;
67:    mapping(uint256 => TileData[TILES_PER_TRAY]) private tiles;
73:    uint256 private prelaunchMinted = type(uint256).max;
```

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol

```solidity
//  private and internal `variables` should preppend with `underline`
23:    address private revenueAddress;
49:    mapping(uint256 => Tray.TileData[]) private nftCharacters;
```

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-identity-protocol/src/CidNFT.sol

```solidity
// private and internal `variables` should preppend with `underline`
84:    mapping(uint256 => mapping(string => SubprotocolData)) internal cidData;
87:    mapping(string => mapping(uint256 => mapping(AssociationType => CIDNFTSubprotocolData))) internal cidDataInverse;
```

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-identity-protocol/src/AddressRegistry.sol

```solidity
// private and internal `variables` should preppend with `underline`
22:    mapping(address => uint256) private cidNFTs;
```