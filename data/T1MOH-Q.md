## ```Namespace.burn()``` doesn't remove nftCharacters[_id]
Thus, returning valid tokenUri on burnt nft. I don't think it was intended. Because in Tray.sol tokenUri reverts on burnt tokenId.
```
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
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L184-L192