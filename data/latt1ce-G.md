## change storage to memory can save gas

In https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L96 

```solidity
function getPFP(uint256 _pfpID) public view returns (address nftContract, uint256 nftID) {
    if (_ownerOf[_pfpID] == address(0)) revert TokenNotMinted(_pfpID);
    ProfilePictureData storage pictureData = pfp[_pfpID];
    nftContract = pictureData.nftContract;
    nftID = pictureData.nftID;
    uint256 cidNFTID = cidNFT.getPrimaryCIDNFT(subprotocolName, _pfpID);
    IAddressRegistry addressRegistry = cidNFT.addressRegistry();
    if (cidNFTID == 0 || addressRegistry.getAddress(cidNFTID) != ERC721(nftContract).ownerOf(nftID)) {
        nftContract = address(0);
        nftID = 0; // Strictly not needed because nftContract has to be always checked, but reset nevertheless to 0
    }
}
```
The `pictureData ` can be declared as memory to save gas because it is a view function and do not change state of pfp[_pfpID]; 