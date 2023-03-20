Revert the logic of this function :
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#:~:text=function%20getPFP(uint256%20_pfpID)%20public%20view%20returns%20(address%20nftContract%2C%20uint256%20nftID)%20%7B

The code can be replaced by something like this (still optimizable) :

nftContract = address(0);
nftID = 0; 
uint256 cidNFTID = cidNFT.getPrimaryCIDNFT(subprotocolName, _pfpID);
IAddressRegistry addressRegistry = cidNFT.addressRegistry();
if ( !(cidNFTID == 0 || addressRegistry.getAddress(cidNFTID) != ERC721(nftContract).ownerOf(nftID)) ) {
    nftID = pictureData.nftID;
    uint256 cidNFTID = cidNFT.getPrimaryCIDNFT(subprotocolName, _pfpID);
    IAddressRegistry addressRegistry = cidNFT.addressRegistry();
}

The call to getPrimaryCIDNFT is made only if the condition is true to optimize gas consumption. 
Also, the safest state ( nftContract = address(0) and nftID = 0) is reached before the condition and subsequently the external call to "ownerof" method from ERC721 contract. 


