# LOW FINDINGS 

##

### [L-1] Front running attacks by the onlyOwner

note,revenueAddress,prelaunchMinted  values are not a constant value and can be changed with changeNoteAddress(),changeRevenueAddress() functions, before a function using note,revenueAddress,prelaunchMinted state variable value in the project, changeNoteAddress(),changeRevenueAddress() functions can be triggered by onlyOwner and operations can be blocked

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Namespace.sol

  function changeNoteAddress(address _newNoteAddress) external onlyOwner {
        address currentNoteAddress = address(note);
        note = ERC20(_newNoteAddress);
        emit NoteAddressUpdate(currentNoteAddress, _newNoteAddress);
    }

    /// @notice Change the revenue address
    /// @param _newRevenueAddress New address to use
    function changeRevenueAddress(address _newRevenueAddress) external onlyOwner {
        address currentRevenueAddress = revenueAddress;
        revenueAddress = _newRevenueAddress;
        emit RevenueAddressUpdated(currentRevenueAddress, _newRevenueAddress);
    }

[Namespace.sol#L196-L208](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L196-L208)

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Tray.sol

   function changeNoteAddress(address _newNoteAddress) external onlyOwner {
        address currentNoteAddress = address(note);
        note = ERC20(_newNoteAddress);
        emit NoteAddressUpdate(currentNoteAddress, _newNoteAddress);
    }

    /// @notice Change the revenue address
    /// @param _newRevenueAddress New address to use
    function changeRevenueAddress(address _newRevenueAddress) external onlyOwner {
        address currentRevenueAddress = revenueAddress;
        revenueAddress = _newRevenueAddress;
        emit RevenueAddressUpdated(currentRevenueAddress, _newRevenueAddress);
    }

    /// @notice End the prelaunch phase and start the public mint
    function endPrelaunchPhase() external onlyOwner {
        if (prelaunchMinted != type(uint256).max) revert PrelaunchAlreadyEnded();
        prelaunchMinted = _nextTokenId() - 1;
        emit PrelaunchEnded();
    }

[Tray.sol#L210-L229](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L210-L229)

Recommended Mitigation Steps
Use a timelock to avoid instant changes of the parameters.

##
### [L-2] AVOID USING TX.ORIGIN

tx.origin is a global variable in Solidity that returns the address of the account that sent the transaction.

Using the variable could make a contract vulnerable if an authorized account calls a malicious contract. You can impersonate a user using a third party contract.

FILE : 2023-03-canto-identity/canto-pfp-protocol/src/ProfilePicture.sol

    63:  turnstile.register(tx.origin);

[ProfilePicture.sol#L63](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L63)

FILE : 2023-03-canto-identity/canto-bio-protocol/src/Bio.sol

    36: turnstile.register(tx.origin);

[Bio.sol#L36](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L36)

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Namespace.sol

##

### [L-3] Use the safe variant and ERC721.mint

.mint won’t check if the recipient is able to receive the NFT. If an incorrect address is passed, it will result in a silent failure and loss of asset

OpenZeppelin [recommendation](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC721/ERC721.sol#L277) is to use the safe variant of _mint	

Recommendation
Replace _mint() with _safeMint()

[ProfilePicture.sol#L86](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L86)

FILE : 2023-03-canto-identity/canto-bio-protocol/src/Bio.sol

   126:   _mint(msg.sender, tokenId);

[Bio.sol#L126](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L126)



##

### [L-4] LOSS OF PRECISION DUE TO ROUNDING

FILE : 2023-03-canto-identity/canto-bio-protocol/src/Bio.sol

   49:  uint lines = (lengthInBytes - 1) / 40 + 1;

[Bio.sol#L49](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L49)

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Utils.sol

   224: uint256 dx = 400 / (_tiles.length + 1);

[Utils.sol#L224](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L224)

[Tray.sol#L259-L263](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L259-L263)

##

### [L-5] Consider using OpenZeppelin’s SafeCast library to prevent unexpected overflows when casting from various type int/uint values 

FILE : 2023-03-canto-identity/canto-bio-protocol/src/Bio.sol

   77: uint8(bioTextBytes[i + 4]) >= 187 &&
   78: uint8(bioTextBytes[i + 4]) <= 191) ||

[](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L77-L78)

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Tray.sol

  163: trayTiledata[j] = _drawing(uint256(lastHash));

[Tray.sol#L163](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L163) 

   250 :  tileData.characterIndex = uint16(charRandValue % NUM_CHARS_EMOJIS);

[src/Tray.sol#L250](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L250) 

   252: tileData.characterIndex = uint16(charRandValue % NUM_CHARS_LETTERS);

[Tray.sol#L252](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L252)

       tileData.fontClass = 3 + uint8((res - 80) / 8);
            } else if (res < 104) {
                tileData.fontClass = 5 + uint8((res - 96) / 4);
            } else if (res < 108) {
                tileData.fontClass = 7 + uint8((res - 104) / 2);


[Tray.sol#L259-L263](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L259-L263)

    267:  tileData.characterModifier = uint8(zalgoSeed % 256);

[Tray.sol#L267](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L267)

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Utils.sol

   272: uint8 lastByteVal = uint8(_startingSequence[3]);

   278: _startingSequence[2] = bytes1(uint8(_startingSequence[2]) + 1);

[Utils.sol#L278](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L278)

Recommended Mitigation Steps:
Consider using OpenZeppelin’s SafeCast library to prevent unexpected overflows when casting from uint256.

##

### [L-6] A single point of failure

Impact
The onlyOwner role has a single point of failure and onlyOwner can use critical a few functions.

Even if protocol admins/developers are not malicious there is still a chance for Owner keys to be stolen. In such a case, the attacker can cause serious damage to the project due to important functions. In such a case, users who have invested in project will suffer high financial losses

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Namespace.sol

  function changeNoteAddress(address _newNoteAddress) external onlyOwner {
        address currentNoteAddress = address(note);
        note = ERC20(_newNoteAddress);
        emit NoteAddressUpdate(currentNoteAddress, _newNoteAddress);
    }

    /// @notice Change the revenue address
    /// @param _newRevenueAddress New address to use
    function changeRevenueAddress(address _newRevenueAddress) external onlyOwner {
        address currentRevenueAddress = revenueAddress;
        revenueAddress = _newRevenueAddress;
        emit RevenueAddressUpdated(currentRevenueAddress, _newRevenueAddress);
    }

[Namespace.sol#L196-L208](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L196-L208)

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Tray.sol

   function changeNoteAddress(address _newNoteAddress) external onlyOwner {
        address currentNoteAddress = address(note);
        note = ERC20(_newNoteAddress);
        emit NoteAddressUpdate(currentNoteAddress, _newNoteAddress);
    }

    /// @notice Change the revenue address
    /// @param _newRevenueAddress New address to use
    function changeRevenueAddress(address _newRevenueAddress) external onlyOwner {
        address currentRevenueAddress = revenueAddress;
        revenueAddress = _newRevenueAddress;
        emit RevenueAddressUpdated(currentRevenueAddress, _newRevenueAddress);
    }

    /// @notice End the prelaunch phase and start the public mint
    function endPrelaunchPhase() external onlyOwner {
        if (prelaunchMinted != type(uint256).max) revert PrelaunchAlreadyEnded();
        prelaunchMinted = _nextTokenId() - 1;
        emit PrelaunchEnded();
    }

[Tray.sol#L210-L229](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L210-L229)

Recommended Mitigation Steps
Add a time lock to critical functions. Admin-only functions that change critical parameters should emit events and have timelocks.

Events allow capturing the changed parameters so that off-chain tools/interfaces can register such changes with timelocks that allow users to evaluate them and consider if they would like to engage/exit based on how they perceive the changes as affecting the trustworthiness of the protocol or profitability of the implemented financial services.

Also detail them in documentation and NatSpec comments.

##

### [L-7] Lack of address(0) checks for critical changes 

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Namespace.sol

  function changeNoteAddress(address _newNoteAddress) external onlyOwner {
        address currentNoteAddress = address(note);
        note = ERC20(_newNoteAddress);
        emit NoteAddressUpdate(currentNoteAddress, _newNoteAddress);
    }

    /// @notice Change the revenue address
    /// @param _newRevenueAddress New address to use
    function changeRevenueAddress(address _newRevenueAddress) external onlyOwner {
        address currentRevenueAddress = revenueAddress;
        revenueAddress = _newRevenueAddress;
        emit RevenueAddressUpdated(currentRevenueAddress, _newRevenueAddress);
    }

[Namespace.sol#L196-L208](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L196-L208)

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Tray.sol

   function changeNoteAddress(address _newNoteAddress) external onlyOwner {
        address currentNoteAddress = address(note);
        note = ERC20(_newNoteAddress);
        emit NoteAddressUpdate(currentNoteAddress, _newNoteAddress);
    }

    /// @notice Change the revenue address
    /// @param _newRevenueAddress New address to use
    function changeRevenueAddress(address _newRevenueAddress) external onlyOwner {
        address currentRevenueAddress = revenueAddress;
        revenueAddress = _newRevenueAddress;
        emit RevenueAddressUpdated(currentRevenueAddress, _newRevenueAddress);
    }
[Tray.sol#L210-L229](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L210-L229)

Recommended Mitigation:

Add address(0) check before critical changes 


##

### [L-8] Prevent division by 0

On several locations in the code precautions are not being taken for not dividing by 0, this will revert the code.
These functions can be called with 0 value in the input, this value is not checked for being bigger than 0, that means in some scenarios this can potentially trigger a division by zero

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Utils.sol

  224: uint256 dx = 400 / (_tiles.length + 1);

[Utils.sol#L224](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L224)

Recommanad Mitigation:

Need to check the (_tiles.length + 1) value is not 0 



  


# NON CRITICAL FINDINGS 

##

### [NC-1] Use More recent version of solidity 

Context:
ALL CONTRACT SCOPE

FILE : 2023-03-canto-identity/canto-pfp-protocol/src/ProfilePicture.sol

   2: pragma solidity >=0.8.0;

[ProfilePicture.sol#L2](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L2)

Recommendations:
Use at least solidity 0.8.17

##

### [NC-2] Named imports can be used

It’s possible to name the imports to improve code readability. E.g. import "@openzeppelin/contracts/token/ERC20/IERC20.sol"; can be rewritten as import {IERC20} from “import “@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol”

FILE : 2023-03-canto-identity/canto-pfp-protocol/src/ProfilePicture.sol

     import "../interface/Turnstile.sol";
     import "../interface/ICidNFT.sol";

[ProfilePicture.sol#L5-L6](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L5-L6)

FILE : 2023-03-canto-identity/canto-bio-protocol/src/Bio.sol

    import "../interface/Turnstile.sol";

[Bio.sol#L7](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L7)

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Namespace.sol

   import "./Tray.sol";
   import "./Utils.sol";
   import "../interface/Turnstile.sol";

[Namespace.sol#L7-L9](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L7-L9)

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Tray.sol

   import "./Utils.sol";
   import "../interface/Turnstile.sol";

[Tray.sol#L10-L11](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L10-L11)

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Utils.sol

   4: import "./Tray.sol";

[Utils.sol#L4](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L4)  

Recommandations:

import {IERC20} from “import “@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol” 

##

### [NC-3] AVOID HARDCODED VALUES

It is not good practice to hardcode values, but if you are dealing with addresses much less, these can change between implementations, networks or projects, so it is convenient to remove these values from the source code

FILE : 2023-03-canto-identity/canto-pfp-protocol/src/ProfilePicture.sol

   62: Turnstile turnstile = Turnstile(0xEcf044C5B4b867CFda001101c617eCd347095B44);

[ProfilePicture.sol#L62](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L62)

   60:   if (block.chainid == 7700) {

[ProfilePicture.sol#L60](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L60)

FILE : 2023-03-canto-identity/canto-bio-protocol/src/Bio.sol

      33: if (block.chainid == 7700) {
      35: Turnstile turnstile = Turnstile(0xEcf044C5B4b867CFda001101c617eCd347095B44);

[Bio.sol#L33-L35](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L33-L35)


##

### [NC-4] Include return parameters in NatSpec comments

@return tag is missing 

[ProfilePicture.sol#L67-L70](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L67-L70)

[Bio.sol#L40-L43](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L40-L43)

[Namespace.sol#L88-L90](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L88-L90)

[Utils.sol#L69-L73](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L69-L73)

[Utils.sol#L219-L222](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L219-L222)

[Utils.sol#L263-L267](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L263-L267)

Recommandations:

Add @return tag when ever function returns any value 


##

### [NC-5]  ADD PARAMETER TO EVENT-EMIT

Some event-emit description hasn’t parameter. Add to parameter for front-end website or client app , they can has that something has happened on the blockchain

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Tray.sol

   80: event PrelaunchEnded();

[Tray.sol#L80](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L80)

##

### [NC-6] NOT USING THE NAMED RETURN VARIABLES ANYWHERE IN THE FUNCTION IS CONFUSING

Consider changing the variable to be an unnamed one

[ProfilePicture.sol#L94](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L94)

[Utils.sol#L256](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L256)

[Tray.sol#L245](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L245)
[Tray.sol#L203](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L203)
[Tray.sol#L195](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L195)


##

### [NC-7] Interchangeable usage of uint and uint256

CONTEXT
ALL CONTRACTS

Consider using only one approach throughout the codebase, e.g. only uint or only uint256

[Bio.sol#L52-L55](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L52-L55)

Recommendations:

only uint or only uint256

##

### [NC-8] Immutables should be in uppercase 

FILE : 2023-03-canto-identity/canto-pfp-protocol/src/ProfilePicture.sol

  14: ICidNFT private immutable cidNFT;

[ProfilePicture.sol#L14](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L14)

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Namespace.sol

  17:  Tray public immutable tray;

[Namespace.sol#L17](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L17)

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Tray.sol

   37:  uint256 public immutable trayPrice;

[Tray.sol#L37](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L37)

   50: address public immutable namespaceNFT;

[Tray.sol#L50](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L50)


##

### [NC-9] Use delete to clear variables instead of zero assignment

FILE : 2023-03-canto-identity/canto-pfp-protocol/src/ProfilePicture.sol

   103:    nftID = 0; 

[ProfilePicture.sol#L103](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L103)

FILE : 2023-03-canto-identity/canto-bio-protocol/src/Bio.sol

   93:   bytesOffset = 0;

[Bio.sol#L93](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L93)

##

### [NC-10] Move IF/validation statements to the top of the function when validating input parameters

FILE : 2023-03-canto-identity/canto-pfp-protocol/src/ProfilePicture.sol


     function mint(address _nftContract, uint256 _nftID) external {
        uint256 tokenId = ++numMinted;
        if (ERC721(_nftContract).ownerOf(_nftID) != msg.sender)
            revert PFPNotOwnedByCaller(msg.sender, _nftContract, _nftID);
        ProfilePictureData storage pictureData = pfp[tokenId];
        pictureData.nftContract = _nftContract;

[ProfilePicture.sol#L79-L88](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L79-L88)

## 

### [NC-11] FOR FUNCTIONS, FOLLOW SOLIDITY STANDARD NAMING CONVENTIONS

internal and private functions : the mixedCase format starting with an underscore (_mixedCase starting with an underscore)

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Utils.sol

[Utils.sol#L69-L77](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L69-L77)

[Utils.sol#L222](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L222)

[Utils.sol#L256](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L256)

##

### [NC-13] Don't use numbers without explanation its not a good code practice . Its possible to use constants 

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Utils.sol

      258:  iteratedState = _currState * 15485863;
      259:  iteratedState = (iteratedState * iteratedState * iteratedState) % 2038074743;

[Utils.sol#L258-L259](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L258-L259)

##

### [NC-14] Use underscores for number literals

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Utils.sol

    258: iteratedState = _currState * 15485863;
    259: iteratedState = (iteratedState * iteratedState * iteratedState) % 2038074743;

[Utils.sol#L258-L259](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L258-L259)

##

### [NC-15] NO SAME VALUE INPUT CONTROL 


FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Namespace.sol

  function changeNoteAddress(address _newNoteAddress) external onlyOwner {
        address currentNoteAddress = address(note);
        note = ERC20(_newNoteAddress);
        emit NoteAddressUpdate(currentNoteAddress, _newNoteAddress);
    }

    /// @notice Change the revenue address
    /// @param _newRevenueAddress New address to use
    function changeRevenueAddress(address _newRevenueAddress) external onlyOwner {
        address currentRevenueAddress = revenueAddress;
        revenueAddress = _newRevenueAddress;
        emit RevenueAddressUpdated(currentRevenueAddress, _newRevenueAddress);
    }

[Namespace.sol#L196-L208](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L196-L208)

##

##

### [NC-16] Need Fuzzing test

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Utils.sol

  257:   unchecked {

[Utils.sol#L257](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L257)

Recommendation

Use should fuzzing test like Echidna.

As Alberto Cuesta Canada said:

Fuzzing is not easy, the tools are rough, and the math is hard, but it is worth it. Fuzzing gives me a level of confidence in my smart contracts that I didn’t have before. Relying just on unit testing anymore and poking around in a testnet seems reckless now.

(https://medium.com/coinmonks/smart-contract-fuzzing-d9b88e0b0a05)

##

### [NC-17] NatSpec comments should be increased in contracts

Context
All Contracts

Description:
It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation.

In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability. (https://docs.soliditylang.org/en/v0.8.15/natspec-format.html)

##

###[NC-18] Function writing that does not comply with the Solidity Style Guide

Context
All Contracts

Description
Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

(https://docs.soliditylang.org/en/v0.8.17/style-guide.html)

Functions should be grouped according to their visibility and ordered:

constructor
receive function (if exists)
fallback function (if exists)
external
public
internal
private
within a grouping, place the view and pure functions last

##

### [NC-19] Add a timelock to critical functions

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user). Consider adding a timelock to

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Namespace.sol

  function changeNoteAddress(address _newNoteAddress) external onlyOwner {
        address currentNoteAddress = address(note);
        note = ERC20(_newNoteAddress);
        emit NoteAddressUpdate(currentNoteAddress, _newNoteAddress);
    }

    /// @notice Change the revenue address
    /// @param _newRevenueAddress New address to use
    function changeRevenueAddress(address _newRevenueAddress) external onlyOwner {
        address currentRevenueAddress = revenueAddress;
        revenueAddress = _newRevenueAddress;
        emit RevenueAddressUpdated(currentRevenueAddress, _newRevenueAddress);
    }

[Namespace.sol#L196-L208](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L196-L208)

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Tray.sol

   function changeNoteAddress(address _newNoteAddress) external onlyOwner {
        address currentNoteAddress = address(note);
        note = ERC20(_newNoteAddress);
        emit NoteAddressUpdate(currentNoteAddress, _newNoteAddress);
    }

    /// @notice Change the revenue address
    /// @param _newRevenueAddress New address to use
    function changeRevenueAddress(address _newRevenueAddress) external onlyOwner {
        address currentRevenueAddress = revenueAddress;
        revenueAddress = _newRevenueAddress;
        emit RevenueAddressUpdated(currentRevenueAddress, _newRevenueAddress);
    }

    /// @notice End the prelaunch phase and start the public mint
    function endPrelaunchPhase() external onlyOwner {
        if (prelaunchMinted != type(uint256).max) revert PrelaunchAlreadyEnded();
        prelaunchMinted = _nextTokenId() - 1;
        emit PrelaunchEnded();
    }

[Tray.sol#L210-L229](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L210-L229)

##

### [NC-20] Long lines are not suitable for the ‘Solidity Style Guide’

[ProfilePicture.sol#L56](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L56)

[Bio.sol#L41](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L41)

[Bio.sol#L53](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L53)
[Bio.sol#L69](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L69)
[Bio.sol#L98](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L98)
[Namespace.sol#L35](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L35)
[Namespace.sol#L117](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L117)
[Namespace.sol#L126](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L126)
[Namespace.sol#L152](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L152)
[Tray.sol#L247](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L247)
[Utils.sol#L19-L49](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L19-L49)
[Utils.sol#L69-L72](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L69-L72)
[Utils.sol#L140](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L140)
[Utils.sol#L195](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L195)
[Utils.sol#L247](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L247)

Recommendation
Multiline output parameters and return statements should follow the same style recommended for wrapping long lines found in the Maximum Line Length section.

(https://docs.soliditylang.org/en/v0.8.17/style-guide.html#introduction)

##

### [Nc-21] Assembly Codes Specific – Should Have Comments

Since this is a low level language that is more difficult to parse by readers, include extensive documentation, comments on the rationale behind its use, clearly explaining what each assembly instruction does.

This will make it easier for users to trust the code, for reviewers to validate the code, and for developers to build on or update the code.

Note that using Assembly removes several important security features of Solidity, which can make the code more insecure and more error-prone.

FILE : 2023-03-canto-identity/canto-bio-protocol/src/Bio.sol

  87: assembly {

[Bio.sol#L87](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L87)

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Namespace.sol

   165:      assembly {

[Namespace.sol#L165](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L165) 

##

### [NC-22] Critical changes should use two-step procedure

Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two-step procedure on the critical functions.

Consider adding a two-steps pattern on critical changes to avoid mistakenly transferring ownership of roles or critical functionalities to the wrong address

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Namespace.sol

  function changeNoteAddress(address _newNoteAddress) external onlyOwner {
        address currentNoteAddress = address(note);
        note = ERC20(_newNoteAddress);
        emit NoteAddressUpdate(currentNoteAddress, _newNoteAddress);
    }

    /// @notice Change the revenue address
    /// @param _newRevenueAddress New address to use
    function changeRevenueAddress(address _newRevenueAddress) external onlyOwner {
        address currentRevenueAddress = revenueAddress;
        revenueAddress = _newRevenueAddress;
        emit RevenueAddressUpdated(currentRevenueAddress, _newRevenueAddress);
    }

[Namespace.sol#L196-L208](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L196-L208)

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Tray.sol

   function changeNoteAddress(address _newNoteAddress) external onlyOwner {
        address currentNoteAddress = address(note);
        note = ERC20(_newNoteAddress);
        emit NoteAddressUpdate(currentNoteAddress, _newNoteAddress);
    }

    /// @notice Change the revenue address
    /// @param _newRevenueAddress New address to use
    function changeRevenueAddress(address _newRevenueAddress) external onlyOwner {
        address currentRevenueAddress = revenueAddress;
        revenueAddress = _newRevenueAddress;
        emit RevenueAddressUpdated(currentRevenueAddress, _newRevenueAddress);
    }

    /// @notice End the prelaunch phase and start the public mint
    function endPrelaunchPhase() external onlyOwner {
        if (prelaunchMinted != type(uint256).max) revert PrelaunchAlreadyEnded();
        prelaunchMinted = _nextTokenId() - 1;
        emit PrelaunchEnded();
    }

[Tray.sol#L210-L229](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L210-L229)

##

### [NC-23] Missing NATSPEC

FILE : 2023-03-canto-identity/canto-pfp-protocol/src/ProfilePicture.sol

[ProfilePicture.sol#L40-L45](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L40-L45)
[ProfilePicture.sol#L50-L52](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L50-L52)
[Bio.sol#L23](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L23)
[Namespace.sol#L54-L67](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L54-L67)
[Tray.sol#L78-L90](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L78-L90)

##

### [NC-24] Imports can be grouped together

Group solmate/utils imports

import {ERC721A} from "erc721a/ERC721A.sol";
import {ERC20} from "solmate/tokens/ERC20.sol";
import {SafeTransferLib} from "solmate/utils/SafeTransferLib.sol";
import {Owned} from "solmate/auth/Owned.sol";
import {LibString} from "solmate/utils/LibString.sol";
import {Base64} from "solady/utils/Base64.sol";
import "./Utils.sol";
import "../interface/Turnstile.sol";

[Tray.sol#L4-L11](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L4-L11)

##

### [S-01] Project Upgrade and Stop Scenario should be

At the start of the project, the system may need to be stopped or upgraded, I suggest you have a script beforehand and add it to the documentation. This can also be called an ” EMERGENCY STOP (CIRCUIT BREAKER) PATTERN “.

(https://github.com/maxwoe/solidity_patterns/blob/master/security/EmergencyStop.sol) 







 

   
 








