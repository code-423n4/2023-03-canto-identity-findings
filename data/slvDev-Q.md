### [L-01] **A single-step ownership transfer pattern is dangerous**

Inheriting from solmate `Owned` contract means you are using a single-step ownership transfer pattern. If an admin provides an incorrect address for the new owner this will result in none of the `onlyOwner` marked methods being callable again. The better way to do this is to use a two-step ownership transfer approach, where the new owner should first claim his new rights before they are transferred. Use OpenZeppelin's `Ownable2Step` instead of solmate `Owned`

### [L-02] ****Consider using OpenZeppelin’s SafeCast library to prevent unexpected overflows when casting from uint256****

```solidity
File: canto-bio-protocol\src\Bio.sol

77:		uint8(bioTextBytes[i + 4]) >= 187 &&
78:		uint8(bioTextBytes[i + 4]) <= 191) ||
```

```solidity
File: canto-namespace-protocol\src\Tray.sol

250:   tileData.characterIndex = uint16(charRandValue % NUM_CHARS_EMOJIS);
252:   tileData.characterIndex = uint16(charRandValue % NUM_CHARS_LETTERS);
255:   tileData.characterIndex = uint16(charRandValue % NUM_CHARS_LETTERS_NUMBERS);
259:   tileData.fontClass = 3 + uint8((res - 80) / 8);
261:   tileData.fontClass = 5 + uint8((res - 96) / 4);
263:   tileData.fontClass = 7 + uint8((res - 104) / 2);
267:   tileData.characterModifier = uint8(zalgoSeed % 256);
```

```solidity
File: canto-namespace-protocol\src\Utils.sol

135:   return abi.encodePacked(bytes1(asciiStartingIndex + uint8(_characterIndex)));
145:   bytes memory character = abi.encodePacked(bytes1(asciiStartingIndex + uint8(_characterIndex)));
215:   return _getUtfSequence(startingSequence, uint8(_characterIndex));
272:   uint8 lastByteVal = uint8(_startingSequence[3]);
278:   _startingSequence[2] = bytes1(uint8(_startingSequence[2]) + 1);
```

### [L-03] Solmate `Owned` Lacks Address(0) Check in `transferOwnership()` Function

Recomindated to use OpenZeppelin's `Ownable` instead of Solmate `Owned`.

```solidity
File: \lib\solmate\src\auth\Owned.sol

function transferOwnership(address newOwner) public virtual onlyOwner {
    owner = newOwner;
    emit OwnershipTransferred(msg.sender, newOwner);
}
```

### [L-04] **Prefer using `_safeMint` over `_mint`**

`Tray.sol` contract use `_mint` with an explanation of why, but other contracts without explanation use  ERC721's `_mint` method, which is missing the check if the account to mint the NFT to is a smart contract that can handle ERC721 tokens. The `_safeMint` method does exactly this, so prefer using it over `_mint`

```solidity
File: canto-bio-protocol\src\Bio.sol

function mint(string calldata _bio) external {
    ...
    _mint(msg.sender, tokenId);
		...
}
```

```solidity
File: canto-namespace-protocol\src\Namespace.sol

function fuse(CharacterData[] calldata _characterList) external {
    ...
    _mint(msg.sender, namespaceIDToMint);
    ...
}
```

```solidity
File: canto-pfp-protocol\src\ProfilePicture.sol

function mint(address _nftContract, uint256 _nftID) external {
  ...
  _mint(msg.sender, tokenId);
}
```

### [L-05] **Missing Event for `mint` and `burn`**

Events help non-contract tools to track changes, and events prevent users from being surprised by changes.

```solidity
File: canto-namespace-protocol\src\Namespace.sol

function burn(uint256 _id) external {
    address nftOwner = ownerOf(_id);
    if (nftOwner != msg.sender && getApproved[_id] != msg.sender && !isApprovedForAll[nftOwner][msg.sender])
        revert CallerNotAllowedToBurn();
    string memory associatedName = tokenToName[_id];
    delete tokenToName[_id];
    delete nameToToken[associatedName];
    _burn(_id);
}
```

```solidity
File: canto-namespace-protocol\src\Tray.sol

function buy(uint256 _amount) external {
	   ...
    _mint(msg.sender, _amount);

function burn(uint256 _id) external {
    ...
    _burn(_id);
}
```

### [L-06] A single point of failure

The `onlyOwner` role has a single point of failure and `onlyOwner` can use critical a few functions.

Even if protocol admins/developers are not malicious there is still a chance for Owner keys to be stolen. In such a case, the attacker can cause serious damage to the project due to important functions. In such a case, users who have invested in project will suffer high financial losses.

`onlyOwner` functions:

```solidity
File: canto-namespace-protocol\src\Namespace.sol

function changeNoteAddress(address _newNoteAddress) external onlyOwner {
function changeRevenueAddress(address _newRevenueAddress) external onlyOwner {
```

```solidity
File: canto-namespace-protocol\src\Tray.sol

function changeNoteAddress(address _newNoteAddress) external onlyOwner {
function changeRevenueAddress(address _newRevenueAddress) external onlyOwner {
function endPrelaunchPhase() external onlyOwner {
```

### [L-07] Solmate’s SafeTransferLib doesn’t check whether the ERC20 contract exists

Solmate’s SafeTransferLib, which is often used to interact with non-compliant/unsafe ERC20 tokens, does not check whether the ERC20 contract exists. The following code will not revert in case the token doesn’t exist (yet).

```solidity
File: canto-namespace-protocol\src\Namespace.sol

function fuse(CharacterData[] calldata _characterList) external {
    ...
    SafeTransferLib.safeTransferFrom(note, msg.sender, revenueAddress, fusingCosts);
    ...
}
```

```solidity
File: canto-namespace-protocol\src\Tray.sol

function buy(uint256 _amount) external {
   ...
   SafeTransferLib.safeTransferFrom(note, msg.sender, revenueAddress, _amount * trayPrice);
   ...
}
```

### [Q-01] Consider making subprotocolName `constant`

Since there is no functionality to change `subprotocolName` and subprotocol contract deployed only once consider making `subprotocolName` variable constant.

```solidity
File: canto-pfp-protocol\src\ProfilePicture.sol

string public subprotocolName;
```

### [Q-02] Miss checks for `immutable` variables in the constructor

Since these variables can't be changed, good practice to add validation(range) and address(0) checks.

```solidity
File: canto-pfp-protocol\src\ProfilePicture.sol

ICidNFT private immutable cidNFT;
constructor(address _cidNFT, string memory _subprotocolName) ERC721("Profile Picture", "PFP") {
    cidNFT = ICidNFT(_cidNFT);
    ...
}

```

```solidity
File: canto-namespace-protocol\src\Namespace.sol

Tray public immutable tray;
constructor(address _tray,address _note,address _revenueAddress) ERC721("Namespace", "NS") Owned(msg.sender) {
    tray = Tray(_tray);
    ...
}
```

```solidity
File: canto-namespace-protocol\src\Tray.sol

uint256 public immutable trayPrice;
address public immutable namespaceNFT;

constructor(bytes32 _initHash,uint256 _trayPrice,address _revenueAddress,address _note,address _namespaceNFT) ERC721A("Namespace Tray", "NSTRAY") Owned(msg.sender) {
    ...
    trayPrice = _trayPrice;
    ...
    namespaceNFT = _namespaceNFT;
    ...
}
```

### [Q-03] Missing checks for `address(0)` when assigning values to address state variables

These cases are missed by the automatic finding script

```solidity
File: canto-namespace-protocol\src\Namespace.sol

ERC20 public note;
function changeNoteAddress(address _newNoteAddress) external onlyOwner {
    address currentNoteAddress = address(note);
    note = ERC20(_newNoteAddress);
    emit NoteAddressUpdate(currentNoteAddress, _newNoteAddress);
}
```

```solidity
File: canto-namespace-protocol\src\Tray.sol

ERC20 public note;
function changeNoteAddress(address _newNoteAddress) external onlyOwner {
    address currentNoteAddress = address(note);
    note = ERC20(_newNoteAddress);
    emit NoteAddressUpdate(currentNoteAddress, _newNoteAddress);
}
```

### [Q-04] Remove an unused function

The declared internal function is not used in any other function.

Similar functionality used `tokenURI(uint256 _id)` I suppose this function was used or should be used there but it didn’t. 

```solidity
File: canto-namespace-protocol\src\Tray.sol

function _beforeTokenTransfers(address,address to,uint256 startTokenId,uint256) internal view override {
    uint256 numPrelaunchMinted = prelaunchMinted;
    if (numPrelaunchMinted != type(uint256).max) {
        // We do not allow any transfers of the prelaunch trays after the phase has ended
        if (startTokenId <= numPrelaunchMinted && to != address(0))
            revert PrelaunchTrayCannotBeUsedAfterPrelaunch(startTokenId);
    }
}

function tokenURI(uint256 _id) public view override returns (string memory) {
    ...
    uint256 numPrelaunchMinted = prelaunchMinted;
    if (numPrelaunchMinted != type(uint256).max) {
        // Prelaunch trays become invalid after the phase has ended
        if (_id <= numPrelaunchMinted) revert PrelaunchTrayCannotBeUsedAfterPrelaunch(_id);
    }
    ...
}
```

### [Q-05] ****Incomplete NatSpec Comments****

Missing NatSpet for a contract in all contracts. eg:

```solidity
/// @title 
/// @author 
/// @notice 
/// @dev 
/// @custom:experimental 
```

Missing `@return` in NatSpec 

```solidity
File: canto-pfp-protocol\src\ProfilePicture.sol

function tokenURI(uint256 _id) public view override returns (string memory) {
```

```solidity
File: canto-namespace-protocol\src\Namespace.sol

function tokenURI(uint256 _id) public view override returns (string memory) {
```

```solidity
File: canto-namespace-protocol\src\Tray.sol

function tokenURI(uint256 _id) public view override returns (string memory) {
function getTile(uint256 _trayId, uint8 _tileOffset) external view returns (TileData memory tileData) {
function getTiles(uint256 _trayId) external view returns (TileData[TILES_PER_TRAY] memory tileData) {
```

```solidity
File: canto-bio-protocol\src\Bio.sol

function tokenURI(uint256 _id) public view override returns (string memory) {
```

### [Q-06] ****Missing NatSpec comments****

```solidity
File: canto-namespace-protocol\src\Tray.sol

function _beforeTokenTransfers(address,address to, uint256 startTokenId,uint256) internal view override {
function _drawing(uint256 _seed) private pure returns (TileData memory tileData) {
```

### [Q-07] **`Function writing` that does not comply with the `Solidity Style Guide`**

Order of Functions; ordering helps readers identify which functions they can call and find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

[https://docs.soliditylang.org/en/v0.8.17/style-guide.html](https://docs.soliditylang.org/en/v0.8.17/style-guide.html)

Functions should be grouped according to their visibility and ordered:

- constructor
- receive function (if exists)
- fallback function (if exists)
- external
- public
- internal
- private
- within a grouping, place the view and pure functions last

```solidity
Files:

canto-pfp-protocol\src\ProfilePicture.sol
canto-namespace-protocol\src\Namespace.sol
canto-namespace-protocol\src\Tray.sol
canto-bio-protocol\src\Bio.sol
```

### [Q-08] ****Uppercase immutable variables****

```solidity
File: canto-namespace-protocol\src\Namespace.sol
Tray public immutable tray;

File: canto-namespace-protocol\src\Tray.sol
uint256 public immutable trayPrice;
address public immutable namespaceNFT;

File: canto-pfp-protocol\src\ProfilePicture.sol
ICidNFT private immutable cidNFT;
```