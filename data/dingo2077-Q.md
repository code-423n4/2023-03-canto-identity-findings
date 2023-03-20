## [01] There is no revert if `block.chainid` other than `7700`.
Smart contract: ProfilePicture.sol, Bio.sol, Tray.sol, Namespace.sol;
Even if during deployment process `turnstile.register` will be other than 7700 (due to the fork or other reasons) contract will be deployed successfully but will not be registered.

```
if (block.chainid == 7700) {
            // Register CSR on Canto mainnnet
            Turnstile turnstile = Turnstile(0xEcf044C5B4b867CFda001101c617eCd347095B44); 
            turnstile.register(tx.origin); 
        }
```

## [02] There is no owner of contract, sent NFTs to this contract could be lost.
Smart contract: ProfilePicture.sol, Bio.sol
Due to the `onERC721Received` implementation anyone can send NFT to contract `ProfilePicture.sol` but no one can send it from this contract.
It is preferable to set the owner at least with rights for sending such NFTs (mistakenly sent to this contract).

## [03] There is option to mint 2 or more profile NFT with one origin NFT and than frontrun before getPFP().
Smart contract: ProfilePicture.sol
Due to the lack of business logic attacker can:
-Buy original NFT, let's assume cryptoPunk#15;
-Mint PFP using `function mint(address _nftContract, uint256 _nftID)`;
Now everyone can check `function getPFP(uint256 _pfpID)` that current PFP backed by cryptopunk.
-Attacker transfer cryptoPunk#15 to attacker#2
-Attacker mint new PFP with new nftID
Now if someone want to check is it backed or not using `function getPFP(uint256 _pfpID)` attacker can frontrun this and transfer to exact _pfpID cryptoPunk#15.
Impact: Scammers could sell fake PFP at canto markets.

## [04] User could mint PFP backed by PFP.
-User calls `function mint(address _nftContract, uint256 _nftID)` and minting PFP.
-User now has two NFT. (1st origin, 2nd PFP)
-User calls `function mint(address _nftContract, uint256 _nftID)` and provide address of PFP contract and nftID of his previous minted ID).

I suggest add `require` here:

```
function mint(address _nftContract, uint256 _nftID) external { ;
        require(_nftContract != address(this),"CustomError"); //<<--add here.
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

## [05] Absence of `require != address(0)`
There are some places where preferably to add `require(_inputAddress != address(0))`;

-Smart contract: `Namespace.sol`
```
constructor(
        address _tray,
        address _note,
        address _revenueAddress
    )
```
There is no chances for reset `_tray` due to the immutable var. 
Preferably to add require for address != address(0) for every parameter.

-Smart contract: `Namespace.sol`, `Tray.sol`.
`function changeNoteAddress(address _newNoteAddress) external onlyOwner`.
Despite that owner can reset any time `_newNoteAddress`, preferably to add require for address != address(0). 

-Smart contract: `Namespace.sol`, `Tray.sol`.
`function changeRevenueAddress(address _newRevenueAddress) external onlyOwner`.
Despite that owner can reset any time `_newRevenueAddress`, preferably to add require for address != address(0).






