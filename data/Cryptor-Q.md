## The same NFT used to mint a profile picture can be used infinite times
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L79-L88

The function mint in ProfilePicture.sol does not check whether an nft has been used before to mint a profile picture meaning that someone can use the same NFT to mint multiple profile pics






## A user can enter " " as a bio, which is still empty

https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L121-L128

The function mint in Bio.sol can pass if a user simply enters a space, whic counts as a character. See the below foundry test 
```
 function testMint() public {
        string memory _bio = " ";
        uint256 prevNnumMinted = bio.numMinted();
        uint256 nnumMinted = prevNnumMinted + 1;

        vm.expectEmit(true, true, true, true);
        emit BioAdded(alice, nnumMinted, _bio);

        vm.prank(alice);
        bio.mint(_bio);

        assertEq(bio.numMinted(), nnumMinted, "Wrong tokenId");
        assertEq(bio.bio(nnumMinted), _bio, "Wrong _bio");
        assertEq(bio.ownerOf(nnumMinted), alice, "NFT not minted");
    }
```

## The function mint in ProfilePicture.sol does not check if the call is an EOA or not, meaning that a smart contract can call mint multiple times

https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L79-L88

This could be a problem relating to gas costs since the function increases the size of storage struct ProfilePictureData shown here.

https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L83

So it is possible that a smart contract could grief the protocol by calling mint multiple times, inflating the gas costs when ProfilePictureData is retrieved in the function getPFP shown here:

https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L94-L105


It is recommended to require that the user is an EOA before calling the function mint






