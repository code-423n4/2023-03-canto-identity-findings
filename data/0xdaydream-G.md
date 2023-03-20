# Summary
| Number | Optimization Details                                                               |     Issues    |
|:------:|------------------------------------------------------------------------------------|:-------------:|
| [G-01] | `x += y (x -= y)` costs more gas than `x = x + y (x = x - y)` for state variables  |       1       |
| [G-02] | Setting the _constructor_ to `payable`                                             |       4       |
| [G-03] | Functions guaranteed to revert when called by normal users can be marked `payable` |       5       |
| [G-04] | Optimize names to save gas                                                         | All contracts |
| [G-05] | Use double if statements instead of &&                                             |       4       |
| [G-06] | Using `private` rather than `public` for constants saves gas                       |       3       |
| [G-07] | Use a more recent version of Solidity                                              | All contracts |

# Details
## [G-01] `x += y (x -= y)` costs more gas than `x = x + y (x = x - y)` for state variables
Using the addition operator instead of plus-equals saves **[113 gas](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)**.

1 issue - 1 file:

```diff
-    numBytes += numBytesChar;
+    numBytes = numBytes + numBytesChar;
```
[Namespace.sol#L150](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L150)

## [G-02] Setting the _constructor_ to `payable`
You can cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor payable. Making the constructor payable eliminates the need for an initial check of `msg.value == 0` and saves **15 gas** on deployment with no security risks.

4 issues - 4 files:

```diff
-    constructor(address _cidNFT, string memory _subprotocolName) ERC721("Profile Picture", "PFP") {
+    constructor(address _cidNFT, string memory _subprotocolName) payable ERC721("Profile Picture", "PFP") {
```
[ProfilePicture.sol#L57](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L57)

```diff
    constructor(
        bytes32 _initHash,
        uint256 _trayPrice,
        address _revenueAddress,
        address _note,
        address _namespaceNFT
-    ) ERC721A("Namespace Tray", "NSTRAY") Owned(msg.sender) {
+    ) payable ERC721A("Namespace Tray", "NSTRAY") Owned(msg.sender) {
```
[Tray.sol#L104](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L104)

```diff
    constructor(
        address _tray,
        address _note,
        address _revenueAddress
-    ) ERC721("Namespace", "NS") Owned(msg.sender) {
+    ) payable ERC721("Namespace", "NS") Owned(msg.sender) {
```
[Namespace.sol#L77](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L77)

```diff
-    constructor() ERC721("Biography", "Bio") {
+    constructor() payable ERC721("Biography", "Bio") {
```
[Bio.sol#L32](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L32)

## [G-03] Functions guaranteed to revert when called by normal users can be marked `payable`
If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided costs an average of about **21 gas per call** to the function, in addition to the extra deployment cost.

5 issues - 2 files:

```diff
-    function changeNoteAddress(address _newNoteAddress) external onlyOwner {
+    function changeNoteAddress(address _newNoteAddress) external payable onlyOwner {
```
[Namespace.sol#L196](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L196)

```diff
-    function changeRevenueAddress(address _newRevenueAddress) external onlyOwner {
+    function changeRevenueAddress(address _newRevenueAddress) external payable onlyOwner {
```
[Namespace.sol#L204](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L204)

```diff
-    function changeNoteAddress(address _newNoteAddress) external onlyOwner {
+    function changeNoteAddress(address _newNoteAddress) external payable onlyOwner {
```
[Tray.sol#L210](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L210)

```diff
-    function changeRevenueAddress(address _newRevenueAddress) external onlyOwner {
+    function changeRevenueAddress(address _newRevenueAddress) external payable onlyOwner {
```
[Tray.sol#L218](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L218)

```diff
-    function endPrelaunchPhase() external onlyOwner {
+    function endPrelaunchPhase() external payable onlyOwner {
```
[Tray.sol#L225](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L225)

## [G-04] Optimize names to save gas
Contracts most called functions could simply save gas by function ordering via Method ID. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because 22 gas are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions. 

See more [here](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92), [here](https://gist.github.com/IllIllI000/a5d8b486a8259f9f77891a919febd1a9) and [here](https://emn178.github.io/online-tools/keccak_256.html).

For example, Tray.sol function names can be named and sorted according to METHOD ID:

```js
Sighash   |   Function Signature
========================
c87b56dd  =>  tokenURI(uint256)
d96a094a  =>  buy(uint256)
42966c68  =>  burn(uint256)
35e37c72  =>  getTile(uint256,uint8)
399d0f4c  =>  getTiles(uint256)
21496de6  =>  changeNoteAddress(address)
92f8b9a3  =>  changeRevenueAddress(address)
ac4081d4  =>  endPrelaunchPhase()
ef435773  =>  _beforeTokenTransfers(address,address,uint256,uint256)
a66a9122  =>  _drawing(uint256)
98995f77  =>  _startTokenId()
```

## [G-05] Use double if statements instead of &&
Using nested if is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

4 issues - 2 files:
```diff
-    if (tileData.fontClass != 0 && _characterList[i].skinToneModifier != 0) {
-        revert CannotFuseCharacterWithSkinTone();
-    }
+    if (tileData.fontClass != 0) {
+        if (_characterList[i].skinToneModifier != 0) {
+            revert CannotFuseCharacterWithSkinTone();
+        }
+    }
```
[Namespace.sol#L136](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L136)

```diff
-    if (nftOwner != msg.sender && getApproved[_id] != msg.sender && !isApprovedForAll[nftOwner][msg.sender])
-        revert CallerNotAllowedToBurn();
+    if (nftOwner != msg.sender) {
+        if (getApproved[_id] != msg.sender) {
+            if (!isApprovedForAll[nftOwner][msg.sender]) {
+                revert CallerNotAllowedToBurn();
+            }
+        }
+    }
```
[Namespace.sol#L186](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L186)

```diff
-    if (numPrelaunchMinted != type(uint256).max && _id <= numPrelaunchMinted)
-        revert PrelaunchTrayCannotBeUsedAfterPrelaunch(_id);
+    if (numPrelaunchMinted != type(uint256).max) {
+        if (_id <= numPrelaunchMinted) {
+            revert PrelaunchTrayCannotBeUsedAfterPrelaunch(_id);
+        }
+    }
```
[Tray.sol#L184](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L184)

```diff
-    if (startTokenId <= numPrelaunchMinted && to != address(0))
-        revert PrelaunchTrayCannotBeUsedAfterPrelaunch(startTokenId);
+    if (startTokenId <= numPrelaunchMinted) {
+        if (to != address(0)) {
+            revert PrelaunchTrayCannotBeUsedAfterPrelaunch(startTokenId);
+        }
+    }
```
[Tray.sol#L240](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L240)

## [G-06] Using `private` rather than `public` for constants saves gas
If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that returns a tuple of the values of all currently-public constants. Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table.

3 issues - 2 files:

```diff
-    Tray public immutable tray;
+    Tray private immutable tray;
```
[Namespace.sol#L17](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L17)

```diff
-    uint256 public immutable trayPrice;
+    uint256 private immutable trayPrice;
```
[Tray.sol#L37](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L37)

```diff
-    address public immutable namespaceNFT;
+    address private immutable namespaceNFT;
```
[Tray.sol#L50](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L50)

## [G-07] Use a more recent version of Solidity

> Solidity 0.8.10 has a useful change that reduced gas costs of external calls which expect a return value. 
> 
> In 0.8.15 the conditions necessary for inlining are relaxed. Benchmarks show that the change significantly decreases the bytecode size (which impacts the deployment cost) while the effect on the runtime gas usage is smaller.
> 
> In 0.8.17 prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call; Simplify the starting offset of zero-length operations to zero. More efficient overflow checks for multiplication.

I recommend that you upgrade the versions of all contracts in scope to the latest version of robustness, `0.8.19`.