# Report

## 01: allow a user to change NFT data in ProfilePicture

Currently, NFT data for a given ProfilePicture token is immutable. But, in case you want to change the PFP associated with your CidNFT, you have to:
- mint a new ProfilePicture with the new NFT data
- update CidNFT to point to that new ProfilePicture token

That's two transactions. An easier solution is to allow people to change the NFT data for a given ProfilePicture:

```sol
    function changePFP(uint _pfpID, address _nftContract, uint _nftID) external {
        require(_ownerOf[_pfpID] == msg.sender);
        if (ERC721(_nftContract).ownerOf(_nftID) != msg.sender)
            revert PFPNotOwnedByCaller(msg.sender, _nftContract, _nftID);

        ProfilePictureData storage pictureData = pfp[_pfpID];
        pictureData.nftContract = _nftContract;
        pictureData.nftID = _nftID;    

        emit PfpChanged(_pfpId, _nftContract, _nftID);
    }
```
