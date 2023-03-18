# LOW FINDINGS 

##
### [L-1] AVOID USING TX.ORIGIN

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

### [L-2] Use the safe variant and ERC721.mint

.mint won’t check if the recipient is able to receive the NFT. If an incorrect address is passed, it will result in a silent failure and loss of asset

OpenZeppelin [recommendation](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC721/ERC721.sol#L277) is to use the safe variant of _mint	

Recommendation
Replace _mint() with _safeMint()

[ProfilePicture.sol#L86](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L86)

##

### [L-3] LOSS OF PRECISION DUE TO ROUNDING

FILE : 2023-03-canto-identity/canto-bio-protocol/src/Bio.sol

   49:  uint lines = (lengthInBytes - 1) / 40 + 1;

[Bio.sol#L49](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L49)


# NON CRITICAL FINDINGS 

##

### [NC-1] Use More recent version of solidity 

FILE : 2023-03-canto-identity/canto-pfp-protocol/src/ProfilePicture.sol

   2: pragma solidity >=0.8.0;

[ProfilePicture.sol#L2](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L2)

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

### [NC-4]  INCLUDE RETURN PARAMETERS IN NATSPEC COMMENTS

@return tag is missing 

[ProfilePicture.sol#L67-L70](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L67-L70)

[Bio.sol#L40-L43](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L40-L43)

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

##

### [NC-7] Interchangeable usage of uint and uint256

CONTEXT
ALL CONTRACTS

Consider using only one approach throughout the codebase, e.g. only uint or only uint256

[Bio.sol#L52-L55](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L52-L55)

   
 








NC-1	Missing checks for address(0) when assigning values to address state variables	4
NC-2	Constants should be defined rather than using magic numbers	3
NC-3	Functions not used internally could be marked external	4

L-1	Unspecific compiler version pragma	5