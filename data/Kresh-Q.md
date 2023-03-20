## Improper Error Names
### Description
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L91
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L196
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L204
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L120
Considering how these errors could be reached, their names (`TokenNotMinted` and `TrayNotMinted`) are improper.
Namespace and Tray contracts have burn functions, therefore NFTs could be minted and then transfered to `address(0)` (burned), and therefore the errors could be reached for minted NFTs.
(it is not the same for Bio and ProfilePicture, because Solmate's implementation have a restriction to transfer to `address(0)` without burn, so the errors are not achievable for minted ones)

### Recommended Mitigation Steps
Replace `NotMinted` with `NotExist` into error names.

## Improper Error Names
### Description
Bio, ProfilePicture and Namespace contracts use Solmate's ERC721 implementation, but Tray contract use ERC721A without any significant reason. It violates the homogeneity of the contracts.

### Recommended Mitigation Steps
Use either Solmate's implementation or ERC721A.

## Improper Error Names
### Description
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L103
It is written that there is no sense to reset nftID, but it still happens.
But moreover, there is twice no sense for "reseting" because a real nftID could also be 0, then it also wouldn't mean at least "reset".

### Recommended Mitigation Steps
Remove this line.

