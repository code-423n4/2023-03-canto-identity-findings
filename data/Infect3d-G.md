# Gas Optimizations

## tokenId can be incremented with an unchecked block
### ProfilePicture::mint
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L80
tokenId can be incremented with an unchecked block. This will save gas and is safe because there is no chance of overflow as there will not be more than 2^256 ProfilePictures minted.

```solidity
    function mint(address _nftContract, uint256 _nftID) external {
        //@audit - GAZ: can be unchecked, no chance of overflow
        uint256 tokenId = ++numMinted;
        if (ERC721(_nftContract).ownerOf(_nftID) != msg.sender)
            revert PFPNotOwnedByCaller(msg.sender, _nftContract, _nftID);
        ProfilePictureData storage pictureData = pfp[tokenId];
        pictureData.nftContract = _nftContract;
        pictureData.nftID = _nftID;
        _mint(msg.sender, tokenId);
        emit PfpAdded(msg.sender, tokenId, _nftContract, _nftID);
    }
```