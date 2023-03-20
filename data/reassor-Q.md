# List
## Low Risk
1. Bio Protocol - Discrepancies between docs and implementation
2. Namespace Protocol - Not following checks-effects-interactions pattern
3. Namespace Protocol - Critical address change
4. Namespace Protocol - Frontrunning buy with change of NOTE address

## Non-Critical Risk
5. Bio Protocol - Usage of `_mint` instead of `_safeMint`
6. Profile Picture Protocol - Can mint multiple PFP NFT pointing to the same NFT
7. Profile Picture Protocol - Can mint PFP NFT to PFP NFT
8. Namespace Protocol - `revenueAddress` is private
9. Namespace Protocol - missing delete of `nftCharacters`
10. Missing natspec
11. Missing pause functionality


# 1. Bio Protocol - Discrepancies between docs and implementation
## Risk
Low

## Impact
Bio Protocol documentation [README.md](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/README.md) states that `bio` has to be shorter than 200 characters:
```
Any user can mint a Bio NFT by calling Bio.mint and passing his biography. It needs to be shorter than 200 characters.
```

but the implementation allows `bio` with 200 characters and only reverts if the bio is longer than 200 characters:
```
if (bytes(_bio).length == 0 || bytes(_bio).length > 200) revert InvalidBioLength(bytes(_bio).length);
```

## Proof of Concept
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L123

## Tools Used
Manual Review / VSCode

## Recommended Mitigation Steps
It is recommended to either fix documentation to reflect that bio can be maximum of 200 characters or adjust implementation and revert if bio length is 200 or more characters.


# 2. Namespace Protocol - Not following checks-effects-interactions pattern
## Risk
Low

## Impact
Functions `Tray.buy` and `Namespace.fuse` do not follow checks-effects-interactions pattern which might lead to reentrancy attacks.

## Proof of Concept
`Tray.sol`:
* https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L150-L168

`Namespace.sol`:
* https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L110-L180

## Tools Used
Manual Review / VSCode

## Recommended Mitigation Steps
It is recommended to first set the effects of `Tray.buy` and `Namespace.fuse` functions and then charge the cost in the end of execution via `SafeTransferLib.safeTransferFrom`.


# 3. Namespace Protocol - Critical address change
## Risk
Low

## Impact
Changing critical addresses such as ownership should be a two-step process where the first transaction (from the old/current address) registers the new address (i.e. grants ownership) and the second transaction (from the new address) replaces the old address with the new one. This gives an opportunity to recover from incorrect addresses mistakenly used in the first step. If not, contract functionality might become inaccessible.

## Proof of Concept
`Tray.sol`:
* https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L13

`Namespace.sol`:
* https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L11

## Tools Used
Manual Review / VSCode

## Recommended Mitigation Steps
It is recommended to implement two-step process for changing ownership.


# 4. Namespace Protocol - Frontrunning buy with change of NOTE address
## Risk
Low

## Impact
Functions `Tray.buy` and `Namespace.fuse` allow buying specified amount of trays or fusing by paying with `$NOTE` token. The issue is that the address of `$NOTE` token can be changed at any time by the owner which might cause issue if the user approved contract `Tray` or `Namespace` to spend any other token than `$NOTE`. Owner might frontrun user and change the address of `$NOTE` token to address of the token user didn't intend to spend.

## Proof of Concept
`Tray.sol`:
* https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L157
* https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L210-L214

`Namespace.sol`:
* https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L114
* https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L196-L200

## Tools Used
Manual Review / VSCode

## Recommended Mitigation Steps
It is recommended to add `_noteAddr`  parameter to `Tray.buy` and `Namespace.fuse` and revert if `_noteAddr` is different than `note` address.


# 5. Usage of `_mint` instead of `_safeMint`
## Risk
Non-Critical

## Impact
Functions `Bio.mint` and `ProfilePicture.mint` are using `_mint` internal function instead of `_safeMint` to mint ERC721 tokens. This might lead to issue if the function is triggered by another smart contract that does not support ERC721 tokens and tokens will get stuck in `Bio` or `ProfilePicture` contracts. 

## Proof of Concept
`Bio.sol`:
* https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L126

`ProfilePicture.sol`:
* https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L86

## Tools Used
Manual Review / VSCode

## Recommended Mitigation Steps
Consider using `_safeMint` instead of `_mint` but make sure to not expose contracts to reentrancy attacks.


# 6. Profile Picture Protocol - Can mint multiple PFP NFT pointing to the same NFT
## Risk
Non-Critical

## Impact
The Profile Picture Protocol allows minting multiple PFP NFTs pointing to the same underlying NFT. This might cause issues if there will be any authorization mechanism implemented by 3rd party protocol that uses PFP NFTs.

## Proof of Concept
* https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L79-L88

## Tools Used
Manual Review / VSCode

## Recommended Mitigation Steps
Consider adding check that will prohibit minting new PFP NFTs in case there is already one owned by the user.


# 7. Profile Picture Protocol - Can mint PFP NFT to PFP NFT
## Risk
Non-Critical

## Impact
Function `ProfilePicture.mint` allows minting PFP NFTs of another PFP NFTs. Since `ProfilePicture.tokenURI` points to the `tokenURI` of the underlying NFT it creates chain of NFTs. This might cause issues if there will be any authorization mechanism implemented by 3rd party protocol that uses PFP NFTs.

## Proof of Concept
* https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L79-L88
* https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L70-L74

## Tools Used
Manual Review / VSCode

## Recommended Mitigation Steps
Consider adding check that will prohibit minting PFP NFTs of the PFP NFTs.


# 8. Namespace Protocol - `revenueAddress` is private
## Risk
Non-Critical

## Impact
Storage variable `revenueAddress` of `Tray` and `Namespace` contracts is private which makes it difficult to query its value. 

## Proof of Concept
`Tray.sol`:
* https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L44

`Namespace.sol`:
* https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L23

## Tools Used
Manual Review / VSCode

## Recommended Mitigation Steps
It is recommended to make `revenueAddress` public so it will be easy to query its value and add transparency to the protocol.


# 9. Namespace Protocol - missing delete of `nftCharacters`
## Risk
Non-Critical

## Impact
Function `Namespace.burn` allows burning specified Namespace NFT. It deletes `tokenToName` and `nameToToken` storage variables and then triggers internal `_burn` function. The issue is that its missing deleting `nftCharacters`.

## Proof of Concept
* https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L184-L192

## Tools Used
Manual Review / VSCode

## Recommended Mitigation Steps
It is recommended to delete `nftCharacters` for the specified Namespace NFT in `Namespace.burn` function.


# 10. Missing natspec
## Risk
Non-Critical

## Impact
Multiple contracts are missing natspec comments which makes code more difficult to read and prone to errors.

## Proof of Concept
`Bio.sol`:
* Missing `@return` param - https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L40-L43

`ProfilePicture.sol`:
* Missing `@return` param - https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L67-L70

`Tray.sol`:
* Missing `@return` param - https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L117-L119
* Missing `@return` param - https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L191-L195
* Missing `@return` param - https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L200-L203
* Missing natspec params for `_drawing` function - https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L245
* Missing `@return` param - https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L276

`Namespace.sol`:
* Missing `@return` param - https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L88-L90

`Utils.sol`:
* Missing `@return` param - https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L69-L73
* Missing `@return` param - https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L219-L222
* Missing `@return` param - https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L263-L270

## Tools Used
Manual Review / VSCode

## Recommended Mitigation Steps
It is recommended to add missing natspec comments.


# 11. Missing pause functionality
## Risk
Non-Critical

## Impact
Protocols Bio, Profile Picture and Namespace are missing pause functionality that could be used in case of security incidents and would block executing selected functions while the contracts are paused.

## Proof of Concept
Namespace protocol:
* https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol
* https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol

Bio protocol:
* https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol

Profile picture protocol:
* https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol

## Tools Used
Manual Review / VSCode

## Recommended Mitigation Steps
It is recommended to add functionality for pausing contracts and go through all publicly/externally accessible functions to decide which one should be forbidden from running while the contracts are paused.