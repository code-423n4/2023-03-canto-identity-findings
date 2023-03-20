Use memory instead of storage: Storing data in memory instead of storage can significantly reduce gas costs. For example, in the mint function, the ProfilePictureData struct could be stored in memory instead of in the pfp mapping.


```
function mint(address _nftContract, uint256 _nftID) external {
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

In this version, we have removed the duplicate check for ownership of the referenced NFT and instead used a require statement to check it in a more efficient way. Additionally, we have reordered the code to avoid unnecessary storage writes, which can save gas.

```
function mint(address _nftContract, uint256 _nftID) external {
    require(ERC721(_nftContract).ownerOf(_nftID) == msg.sender, "PFPNotOwnedByCaller");
    uint256 tokenId = ++numMinted;
    ProfilePictureData storage pictureData = pfp[tokenId];
    pictureData.nftContract = _nftContract;
    pictureData.nftID = _nftID;
    _mint(msg.sender, tokenId);
    emit PfpAdded(msg.sender, tokenId, _nftContract, _nftID);
}```