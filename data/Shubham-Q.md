# Report


## QA


| |Issue|
|-|:-|
| [L-1](#L-1) | Use of Insecure Function `tx.origin` 
| [NC-1](#NC-1) | Missing NatSpec comments

## [L-1] Use of Insecure Function `tx.origin` 
tx.origin is a security vulnerability. As we recently saw with the Mist wallet, using tx.origin makes you vulnerable to attacks comparable to phishing or cross-site scripting. Once a user has interacted with a malicious contract, that contract can then impersonate the user to any contract relying on tx.origin.

tx.origin breaks compatibility. Using tx.origin means that your contract cannot be used by another contract, because a contract can never be the tx.origin. This breaks the general composability of Ethereum contracts, and makes them less useful. In addition, this is another security vulnerability, because it makes security-based contracts like multisig wallets incompatible with your contract.

```solidity
File: canto-bio-protocol/src/Bio.sol

36:      turnstile.register(tx.origin);

```

```solidity
File: canto-namespace-protocol/src/Namespace.sol

84:      turnstile.register(tx.origin);
```

```solidity
File: canto-namespace-protocol/src/Tray.sol

113:     turnstile.register(tx.origin);

```

```solidity
File: canto-pfp-protocol/src/ProfilePicture.sol

63:      turnstile.register(tx.origin);

```
##### Recommendation
Use msg.sender instead of tx.origin

## [NC-1] Missing NatSpec comments
NatSpec comments provide rich code documentation. The following functions are some examples that miss NatSpec comments. Please consider adding NatSpec comments for functions like these.

```solidity
File: canto-namespace-protocol/src/Tray.sol

245:         function _drawing(uint256 _seed) private pure returns (TileData memory tileData) {


```
