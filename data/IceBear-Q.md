## Low Issues

### [L-1] SOLMATE’S SAFETRANSFERLIB DOESN’T CHECK WHETHER THE ERC20 CONTRACT EXISTS
Solmate’s SafeTransferLib, which is often used to interact with non-compliant/unsafe ERC20 tokens, does not check whether the ERC20 contract exists. The following code will not revert in case the token doesn’t exist (yet).

##### This is stated in the Solmate library: 
https://github.com/transmissions11/solmate/blob/main/src/utils/SafeTransferLib.sol#L9

*Find (1) instance(s) in contracts*:
```solidity
File: canto-namespace-protocol/src/Tray.sol

157:             SafeTransferLib.safeTransferFrom(note, msg.sender, revenueAddress, _amount * trayPrice);

```
[canto-namespace-protocol/src/Tray.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol)

### [L-2] Use safeTransferOwnership instead of transferOwnership function
transferOwnership function is used to change Ownership from Owned.sol.
safeTransferOwnership, use it is more secure due to 2-stage ownership transfer.
    
#### Recommendation:
Use a 2 structure transferOwnership which is safer.
Use [Ownable2Step.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol) in contracts.

*Find (4) instance(s) in contracts*:
```solidity
File: canto-namespace-protocol/src/Namespace.sol

5: import {Owned} from "solmate/auth/Owned.sol";

11: contract Namespace is ERC721, Owned {

```
[canto-namespace-protocol/src/Namespace.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol)

```solidity
File: canto-namespace-protocol/src/Tray.sol

7: import {Owned} from "solmate/auth/Owned.sol";

13: contract Tray is ERC721A, Owned {

```
[canto-namespace-protocol/src/Tray.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol)

### [L-3] Avoid using tx.origin
`tx.origin` is a global variable in Solidity that returns the address of the account that sent the transaction.

Using the variable could make a contract vulnerable if an authorized account calls a malicious contract. You can impersonate a user using a third party contract.

This can make it easier to create a vault on behalf of another user with an external administrator (by receiving it as an argument).

*Find (4) instance(s) in contracts*:
```solidity
File: canto-bio-protocol/src/Bio.sol

36:             turnstile.register(tx.origin);

```
[canto-bio-protocol/src/Bio.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol)

```solidity
File: canto-namespace-protocol/src/Namespace.sol

84:             turnstile.register(tx.origin);

```
[canto-namespace-protocol/src/Namespace.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol)

```solidity
File: canto-namespace-protocol/src/Tray.sol

113:             turnstile.register(tx.origin);

```
[canto-namespace-protocol/src/Tray.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol)

```solidity
File: canto-pfp-protocol/src/ProfilePicture.sol

63:             turnstile.register(tx.origin);

```
[canto-pfp-protocol/src/ProfilePicture.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol)
