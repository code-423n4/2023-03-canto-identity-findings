|  | Issue | Instances |
| --- | --- | --- |
| [G-01] | Use nested if, avoid multiple check combinations | 4 |
| [G-02] | Setting the constructor to payable | 4 |
| [G-03] | Use assembly to write address storage values | 8 |
| [G-04] | Cache storage values in memory to minimize SLOADs | 1 |
| [G-05] | Use constants instead of type(uintx).max | 6 |
| [G-06] | Functions guaranteed to revert_ when called by normal users can be marked payable | 5 |
| [G-07] | Use a more recent version of solidity | 5 |
| [G-08] | Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead | * |
| [G-09] | Avoid contract existence checks by using solidity version 0.8.10 or later | 13 |

### [G-01] Use nested `if`, avoid multiple check combinations

Using nested if is cheaper than using `&&` multiple check combinations if it ****is not followed by an `else` statement. There are more advantages, such as easier-to-read code and better coverage reports.

```solidity
File: canto-namespace-protocol\src\Namespace.sol

if (tileData.fontClass != 0 && _characterList[i].skinToneModifier != 0) {
if (nftOwner != msg.sender && getApproved[_id] != msg.sender && !isApprovedForAll[nftOwner][msg.sender])

```

```solidity
File: canto-namespace-protocol\src\Tray.sol

if (numPrelaunchMinted != type(uint256).max && _id <= numPrelaunchMinted)
if (startTokenId <= numPrelaunchMinted && to != address(0))

```

### [G-02] Setting the *constructor* to `payable`

You can cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor payable. Making the constructor payable eliminates the need for an initial check of `msg.value == 0` and saves gas on deployment with no security risks.

```solidity
All contracts
```

### [G-03] Use `assembly` to write *address storage values*

```solidity
File: canto-namespace-protocol\src\Namespace.sol

constructor(address _tray,address _note,address _revenueAddress) ERC721("Namespace", "NS") Owned(msg.sender) {
		...
    note = ERC20(_note);
    revenueAddress = _revenueAddress;
    ...
}

function changeNoteAddress(address _newNoteAddress) external onlyOwner {
    ...
    note = ERC20(_newNoteAddress);
    ...
}

/// @notice Change the revenue address
/// @param _newRevenueAddress New address to use
function changeRevenueAddress(address _newRevenueAddress) external onlyOwner {
    ...
    revenueAddress = _newRevenueAddress;
    ...
}
```

```solidity
File: canto-namespace-protocol\src\Tray.sol

constructor(bytes32 _initHash,uint256 _trayPrice,address _revenueAddress,address _note,address _namespaceNFT) ERC721A("Namespace Tray", "NSTRAY") Owned(msg.sender) {
    ...
    revenueAddress = _revenueAddress;
    note = ERC20(_note);
    ...
}

function changeNoteAddress(address _newNoteAddress) external onlyOwner {
    ...
    note = ERC20(_newNoteAddress);
    ...
}

function changeRevenueAddress(address _newRevenueAddress) external onlyOwner {
    ...
    revenueAddress = _newRevenueAddress;
    ...
}
```

### [G-04] Cache storage values in memory to minimize SLOADs

The code can be optimized by minimizing the number of SLOADs.

SLOADs are expensive (100 gas after the 1st one) compared to MLOADs/MSTOREs (3 gas each). Storage values read multiple times should instead be cached in memory the first time (costing 1 SLOAD) and then read from this cache to avoid multiple SLOADs.

```solidity
File: canto-namespace-protocol\src\Tray.sol

function tokenURI(uint256 _id) public view override returns (string memory) {
    ...
-    TileData[TILES_PER_TRAY] storage storedNftTiles = tiles[_id];
+    TileData[TILES_PER_TRAY] memory storedNftTiles = tiles[_id];
    ...
}

+   Overall gas change: -1371
```

### [G-05] Use constants instead of type(uintx).max

It uses more gas in the distribution process and also for each transaction than constant usage.

```solidity
File: canto-namespace-protocol\src\Tray.sol

uint256 private prelaunchMinted = type(uint256).max;
if (numPrelaunchMinted != type(uint256).max) {
if (prelaunchMinted == type(uint256).max) {
if (numPrelaunchMinted != type(uint256).max && _id <= numPrelaunchMinted)
if (prelaunchMinted != type(uint256).max) revert PrelaunchAlreadyEnded();
if (numPrelaunchMinted != type(uint256).max) {
```

### [G-06] Functions guaranteed to revert_ when called by normal users can be marked `payable`

If a function modifier or require such as onlyOwner-admin is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

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

### [G-07] Use a more recent version of solidity

Use a solidity version of at least 0.8.2 to get simple compiler automatic inlining

Use a solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads

Use a solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than `revert()/require()` strings

Use a solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value

In 0.8.15 the conditions necessary for inlining are relaxed. Benchmarks show that the change significantly decreases the bytecode size (which impacts the deployment cost) while the effect on the runtime gas usage is smaller.

In 0.8.17 prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call; Simplify the starting offset of zero-length operations to zero. More efficient overflow checks for multiplication.

### [G-08] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contracts gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

[https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html](https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html)

Use a larger size than downcast where needed.

### [G-09] Avoid contract existence checks by using solidity version 0.8.10 or later

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value

```solidity
File: canto-pfp-protocol\src\ProfilePicture.sol

turnstile.register(tx.origin);
return ERC721(nftContract).tokenURI(nftID);
if (ERC721(_nftContract).ownerOf(_nftID) != msg.sender)
uint256 cidNFTID = cidNFT.getPrimaryCIDNFT(subprotocolName, _pfpID);
IAddressRegistry addressRegistry = cidNFT.addressRegistry();
if (cidNFTID == 0 || addressRegistry.getAddress(cidNFTID) != ERC721(nftContract).ownerOf(nftID)) {
```

```solidity
File: canto-namespace-protocol\src\Namespace.sol

Tray.TileData memory tileData = tray.getTile(trayID, tileOffset); // Will revert if tileOffset is too high
address trayOwner = tray.ownerOf(trayID);
tray.getApproved(trayID) != msg.sender &&
!tray.isApprovedForAll(trayOwner, msg.sender)
tray.burn(uniqueTrays[i]);
```

```solidity
File: canto-namespace-protocol\src\Tray.sol

turnstile.register(tx.origin);
```

```solidity
File: canto-bio-protocol\src\Bio.sol

turnstile.register(tx.origin);
```