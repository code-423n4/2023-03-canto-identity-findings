## [L-01] Prefer using `_safeMint` over `_mint`
Bio.sol, Namespace.sol, Tray.sol and ProfilePicture.sol use ERC721's `_mint` method, which is missing the check if the account to mint the NFT to is a smart contract that can handle ERC721 tokens. The `_safeMint` method does exactly this, so prefer using it over `_mint` but always add a `nonReentrant` modifier, since calls to `_safeMint` can reenter.

```solidity
File: canto-bio-protocol/src/Bio.sol
126:    _mint(msg.sender, tokenId);
```
[Bio.sol#L126](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L126)

```solidity
File: canto-namespace-protocol/src/Namespace.sol
177:    _mint(msg.sender, namespaceIDToMint);
```
[Bio.sol#L177](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L177)

```solidity
File: canto-namespace-protocol/src/Tray.sol
167:    _mint(msg.sender, _amount); // We do not use _safeMint on purpose here to disallow callbacks and save gas
```
[Bio.sol#L167](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L167)

```solidity
File: canto-pfp-protocol/src/ProfilePicture.sol
126:    _mint(msg.sender, tokenId);
```
[Bio.sol#L86](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L86)

## [N-01] Incosistent banner-like comments
`STATE` comment in `ProfilePicture.sol` is duplicated and can be removed for consistency and clarity.

```diff
/*//////////////////////////////////////////////////////////////
                                STATE
//////////////////////////////////////////////////////////////*/

/// @notice Reference to the CID NFT
ICidNFT private immutable cidNFT;
-
-    /*//////////////////////////////////////////////////////////////
-                                    STATE
-    //////////////////////////////////////////////////////////////*/

/// @notice Data that is stored per PFP
struct ProfilePictureData {
    /// @notice Reference to the NFT contract
    address nftContract;
    /// @notice Referenced nft ID
    uint256 nftID;
}
```

[ProfilePicture.sol#L15](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L15)
## [N-02] Incomplete NatSpec Comments
NatSpec comments provide rich code documentation. The following functions are some examples that miss the `@return` comment. Please consider completing the NatSpec comments for functions like these.

```solidity
File: canto-pfp-protocol/src/ProfilePicture.sol
70:    function tokenURI(uint256 _id) public view override returns (string memory) {
```
[ProfilePicture.sol#L70](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L70)

```solidity
File: canto-bio-protocol/src/Bio.sol
43:    function tokenURI(uint256 _id) public view override returns (string memory) {
```
[Bio.sol#L43](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L43)

```solidity
File: canto-namespace-protocol/src/Namespace.sol
90:    function tokenURI(uint256 _id) public view override returns (string memory) {
```
[Namespace.sol#L90](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L90)

```solidity
File: canto-namespace-protocol/src/Tray.sol
119:    function tokenURI(uint256 _id) public view override returns (string memory) {
195:    function getTile(uint256 _trayId, uint8 _tileOffset) external view returns (TileData memory tileData) {
203:    function getTiles(uint256 _trayId) external view returns (TileData[TILES_PER_TRAY] memory tileData) {
276:    function _startTokenId() internal pure override returns (uint256) {
```
[Tray.sol#L119](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L119)

[Tray.sol#L195](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L195)

[Tray.sol#L203](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L203)

[Tray.sol#L276](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L276)

```solidity
File: canto-namespace-protocol/src/Utils.sol
73:    function characterToUnicodeBytes(
222:    function generateSVG(Tray.TileData[] memory _tiles, bool _isTray) internal pure returns (string memory) {
267:    function _getUtfSequence(bytes memory _startingSequence, uint8 _characterIndex)
```
[Utils.sol#L73](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L73)

[Utils.sol#L222](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L222)

[Utils.sol#L267](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L267)
## [N-03] Missing NatSpec comments
NatSpec comments provide rich code documentation. The following functions are some examples that miss NatSpec comments. Please consider adding NatSpec comments for functions like these.

```solidity
File: canto-namespace-protocol/src/Tray.sol
231:    function _beforeTokenTransfers(
245:    function _drawing(uint256 _seed) private pure returns (TileData memory tileData) {
```
[Tray.sol#L231](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L231)

[Tray.sol#L245](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L245)