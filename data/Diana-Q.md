
-----------------
| S No. | Issue | Instances |
|-----|-----|-----|
| [01] | Avoid using tx.origin | 4
| [02] | \_safemint() should be used rather than \_mint() wherever possible | 3
| [03] | Denial of Service - tokenToName and nameToToken growing indefinitely | 2
| [04] | Solmate safetransferfrom does not check the codesize of the token address, which may lead to fund loss | 2
| [05] | Use a more recent version of solidity | 5

-------------

## 01 Avoid using tx.origin

`tx.origin` is a global variable in Solidity that returns the address of the account that sent the transaction.

Using the variable could make a contract vulnerable if an authorized account calls a malicious contract. You can impersonate a user using a third party contract.

_There are 4 instances of this issue_

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol

```
File: main/canto-bio-protocol/src/Bio.sol

36: turnstile.register(tx.origin);
```

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol

```
File: main/canto-namespace-protocol/src/Namespace.sol

84: turnstile.register(tx.origin);
```

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol

```
File: main/canto-namespace-protocol/src/Tray.sol

113: turnstile.register(tx.origin);
```

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol

```
File: main/canto-pfp-protocol/src/ProfilePicture.sol

63: turnstile.register(tx.origin);
```

----------

## 02 \_safemint() should be used rather than \_mint() wherever possible

`_mint()` is discouraged in favor of `_safeMint()` which ensures that the recipient is either an EOA or implements `IERC721Receiver`. Both open OpenZeppelin and solmate have versions of this function so that NFTs aren’t lost if they’re minted to contracts that cannot transfer them back out.

_There are 3 instances of this issue_

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol

```
File: main/canto-bio-protocol/src/Bio.sol

126: _mint(msg.sender, tokenId);
```

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol

```
File: main/canto-namespace-protocol/src/Namespace.sol

177: _mint(msg.sender, namespaceIDToMint);
```

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol

```
File: main/canto-pfp-protocol/src/ProfilePicture.sol

86: _mint(msg.sender, tokenId);
```

--------------

## 03 Denial of Service - tokenToName and nameToToken growing indefinitely

State variables `tokenToName` and `nameToToken` are growing indefinitely, this might lead to expensive transactions and effectively denial of service for the user when calling `burn` function that requires iterations over the whole arrays.

The contract is using `tokenToName` and `nameToToken` to track token ids and associated names for tokens. This helps with parsing state variables that contain data for all tokens. The elements for these are deleted by `delete tokenToName[_id]`, `delete nameToToken[associatedName]`, however the delete is only setting data located at the given index to zero. This make all element re-parsed every time the function is called, making the user consuming more gas. This results in denial of service,

_There are 2 instances of this issue_

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol

```
File: main/canto-namespace-protocol/src/Namespace.sol

189: delete tokenToName[_id];
190: delete nameToToken[associatedName];
```

### Recommended Mitigation Steps

It is recommended to remove elements from the arrays `tokenToName`/`nameToToken` using `pop()` function when deleting tokens.

-----------

## 04 Solmate safetransferfrom does not check the codesize of the token address, which may lead to fund loss

In `fuse()` and `buy()`, the `safetransferfrom` doesn’t check the existence of code at the token address. This is a known issue while using solmate’s libraries.  
Hence this may lead to miscalculation of funds and may lead to loss of funds , because if `safetransferfrom()` is called on a token address that doesn’t have contract in it, it will always return success, bypassing the return value check. Due to this protocol will think that funds have been transferred and successful , and records will be accordingly calculated, but in reality funds were never transferred.  
So this will lead to miscalculation and possibly loss of funds

_There are 2 instances of this issue_

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol

```
File: main/canto-namespace-protocol/src/Namespace.sol

114: SafeTransferLib.safeTransferFrom(note, msg.sender, revenueAddress, fusingCosts);
```

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol

```
File: main/canto-namespace-protocol/src/Tray.sol

157:  SafeTransferLib.safeTransferFrom(note, msg.sender, revenueAddress, _amount * trayPrice);
```

### Recommended Mitigation Steps

Use openzeppelin’s safeERC20 or implement a code existence check.

-------

## 05 Use a more recent version of solidity

_There are 5 instances of this issue_

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol

```
File: main/canto-bio-protocol/src/Bio.sol

2: pragma solidity >=0.8.0;
```

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol

```
File: main/canto-namespace-protocol/src/Namespace.sol

2: pragma solidity >=0.8.0;
```

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol

```
File: main/canto-namespace-protocol/src/Tray.sol

2: pragma solidity >=0.8.0;
```

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol

```
File: main/canto-pfp-protocol/src/ProfilePicture.sol

2: pragma solidity >=0.8.0;
```

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol

```
File: main/canto-namespace-protocol/src/Utils.sol

2: pragma solidity >=0.8.0;
```

-----------

