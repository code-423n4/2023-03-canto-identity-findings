

| Letter | Name | Description |
|:--:|:-------:|:-------:|
| L  | Low risk | Potential risk |
| NC |  Non-critical | Non risky findings |
| R  | Refactor | Changing the code |
| O | Ordinary | Often found issues |



# Canto-namespace-protocol

---


## [L-01] Arithmatic OverFlow in buy Function of Tray.sol    


 1.Alice mint  100 tokens  of `$note ` 
 2.then she approve it to  address(tray ) so she can allow Tray contract to transfer her tokens to revenue address
3.Lets assume that the preMint phase is over , Alice call buy function with   exact amount  of tokens  that she minted (100 `$note`  tokens) thinking that there's no `trayPrice`    so  _amount = 100   , in  safeTranfer   `_amount * trayPrice`  so we get this error

```bash
Tray::buy(100) 
    │   ├─ [929] MockToken::transferFrom(Alice: [0xBf0b5A4099F0bf6c8bC4252eBeC548Bae95602Ea], User: [0x5cb738DAe833Ec21fe65ae1719fAd8ab8cE7f23D], 10000000000000000000000) 
    │   │   └─ ← "Arithmetic over/underflow"
    │   └─ ← "TRANSFER_FROM_FAILED"
    └─ ← "TRANSFER_FROM_FAILED"

```

so to handle this error we can use  a if statement with a customError
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L150
eg :
```js
 else {
            if (_amount * trayPrice > note.balanceOf(msg.sender)) {
                revert("Not sufficient amount of NoteToken minted");
            }
            SafeTransferLib.safeTransferFrom(
                note,

                msg.sender,

                revenueAddress,

                _amount * trayPrice

            );

        }

```


```bash
Tray::buy(100) 
    │   ├─ [579] MockToken::balanceOf(Alice: [0xBf0b5A4099F0bf6c8bC4252eBeC548Bae95602Ea]) [staticcall]
    │   │   └─ ← 100000000000000000000
    │   └─ ← "Not sufficient amount of NoteToken minted"
```
`noteToken`  a test token  which is out of scope but  its mint function is having public visibilty  also check about it  because 
anyone can mint arbitray amount of note Tokens ; 

```bash
 function mint(address to, uint256 amount) public {

        _mint(to, amount);

    }
```
## [N-01] Allow  users to mint only a  specific amount of trays 
  
## [L-02] Use nonreentrant modifier for function that use safeTransferFrom , safeTransfer
 
1.Tray.sol    
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L150
### Recommendations

use open zeppelin re-entrancy Guard and nonreentrant modifier 


# Canto-pfp-protocol

---

 ProfilePicture.sol
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol


## [L-03] NO address(0) checks  ProfilePicture.sol

### In Constructor 

To happen this kind of thing likelyhood is very low but  it's better to address(0) check in the constructor 


## Proof of Concept

Before

```bash
   function testEx() public {

        vm.startPrank(deployer);

        pfp = new ProfilePicture(address(0), "SubprotocolName");

        vm.stopPrank();

    }

}
```


```bash
[PASS] testEx() (gas: 1119743)
Traces:
  [1119743] Pwn::testEx() 
    ├─ [0] VM::startPrank(Owner: [0x93B07479cC807d037A98B6bB122CE8201958a595]) 
    │   └─ ← ()
    ├─ [1058421] → new ProfilePicture@0xec1f4727BEc84915fC1c59764F96276EBaA35Ce5
    │   └─ ← 4946 bytes of code
    ├─ [0] VM::stopPrank() 
    │   └─ ← ()
    └─ ← ()
```

After
```js
  constructor(

        address _cidNFT,

        string memory _subprotocolName

    ) ERC721("Profile Picture", "PFP") {

        if (_cidNFT == address(0)) {

            revert("CidNFT cannot be address 0 ");

        }
```

```bash
[FAIL. Reason: CidNFT cannot be address 0 ] testEx() (gas: 84779)
```

## [N-02] Revert if _sublinprotocolName is null_  in constructor 

```bash

if (bytes(_subprotocolName).length == 0) {

            revert("name null");

        }
```

### In functions 
Add address(0) check  for address _nftContract _ 
also lack of access control control in mint function any address for ERC721 could be sent as the parameter
```bash
function mint(address _nftContract, uint256 _nftID) external {

        uint256 tokenId = ++numMinted;

        if (ERC721(_nftContract).ownerOf(_nftID) != msg.sender)
```
	



#  Canto-bio-protocol

---

## [L-04] Lack of Access Control in mint function  

## Proof of Concept
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L121

Better to  give roles like  only USERs   of the protocol allowed to mint the bio ;    
``` js
 function mint(string calldata _bio) external {

        // We check the length in bytes, so will be higher for UTF-8 characters. But sufficient for this check

        if (bytes(_bio).length == 0 || bytes(_bio).length > 200)

            revert InvalidBioLength(bytes(_bio).length);

        uint256 tokenId = ++numMinted;

        bio[tokenId] = _bio;

        _mint(msg.sender, tokenId);

        emit BioAdded(msg.sender, tokenId, _bio);

    }


```