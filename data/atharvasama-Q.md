# Low and Non-critical issues

## [01] Solmate’s SafeTransferLib doesn’t check whether the ERC20 contract exists
Solmate’s SafeTransferLib, which is often used to interact with non-compliant/unsafe ERC20 tokens, does not check whether the ERC20 contract exists. The following code will not revert in case the token doesn’t exist (yet).

This is stated in the Solmate library: https://github.com/transmissions11/solmate/blob/main/src/utils/SafeTransferLib.sol#L9

[canto-namespace-protocol/src/Namespace.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol):
```js
114:         SafeTransferLib.safeTransferFrom(note, msg.sender, revenueAddress, fusingCosts);
```

[canto-namespace-protocol/src/Tray.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol):
```js
6: import {SafeTransferLib} from "solmate/utils/SafeTransferLib.sol";

157:             SafeTransferLib.safeTransferFrom(note, msg.sender, revenueAddress, _amount * trayPrice);
```

## [02] Two steps verification before transferring ownership
Solmate's Auth.sol transfers ownership in a single step using the ```transferOwnership()``` function. Transferring of ownership should be a two-step process where the first transaction (from the old/current address) registers the new address (i.e. grants ownership) and the second transaction (from the new address) replaces the old address with the new one (i.e. claims ownership). This gives an opportunity to recover from incorrect addresses mistakenly used in the first step. If not, contract functionality might become inaccessible.

The contracts using Auth.sol include

[canto-namespace-protocol/src/Namespace.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol)

[canto-namespace-protocol/src/Tray.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol)

## [03] Typos
[canto-pfp-protocol/src/ProfilePicture.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol)
```js
61:             // Register CSR on Canto mainnnet
```

[canto-namespace-protocol/src/Namespace.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol)
```js
82:             // Register CSR on Canto mainnnet
```

[canto-namespace-protocol/src/Tray.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol)
```js
95:     /// @param _revenueAddress Adress to send the revenue to

111:             // Register CSR on Canto mainnnet

247:         uint256 charRandValue = Utils.iteratePRNG(_seed); // Iterate PRNG to not have any biasedness / correlation between random numbers
```

[canto-namespace-protocol/src/Utils.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol)
```js
7: /// @notice Utiltities for the on-chain SVG generation of the text data and pseudo randomness
```

## [04] Use a more recent version of Solidity
Use a solidity version of at least 0.8.13 to get the ability to use ```using``` ```for``` with a list of free functions

Use a more recent solidity versions for the following contracts

[canto-bio-protocol/src/Bio.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol)

[canto-namespace-protocol/src/Namespace.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol) 

[canto-namespace-protocol/src/Tray.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol)

[canto-namespace-protocol/src/Utils.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol)

[canto-pfp-protocol/src/ProfilePicture.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol)

## [05] Avoid using tx.origin
tx.origin is a global variable in Solidity that returns the address of the account that sent the transaction.

Using the variable could make a contract vulnerable if an authorized account calls a malicious contract. You can impersonate a user using a third party contract.

[canto-bio-protocol/src/Bio.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol)
```js
36:             turnstile.register(tx.origin);
```

[canto-namespace-protocol/src/Namespace.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol):
```js
84:             turnstile.register(tx.origin);
```

[canto-namespace-protocol/src/Tray.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol):
```js
113:             turnstile.register(tx.origin);
```

[canto-pfp-protocol/src/ProfilePicture.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol)
```js
63:             turnstile.register(tx.origin);
```

## [06] A single point of failure
Add a time lock to critical functions. Admin-only functions that change critical parameters should emit events and have timelocks.

Events allow capturing the changed parameters so that off-chain tools/interfaces can register such changes with timelocks that allow users to evaluate them and consider if they would like to engage/exit based on how they perceive the changes as affecting the trustworthiness of the protocol or profitability of the implemented financial services.

[canto-namespace-protocol/src/Namespace.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol)
```js
196:     function changeNoteAddress(address _newNoteAddress) external onlyOwner {

204:     function changeRevenueAddress(address _newRevenueAddress) external onlyOwner {
```

[canto-namespace-protocol/src/Tray.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol)
```js
210:     function changeNoteAddress(address _newNoteAddress) external onlyOwner {

218:     function changeRevenueAddress(address _newRevenueAddress) external onlyOwner {

225:     function endPrelaunchPhase() external onlyOwner {
```