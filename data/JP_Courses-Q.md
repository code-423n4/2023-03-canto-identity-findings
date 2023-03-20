https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol

0) 
Consider using more recent version(s) of solidity compiler for the smart contracts, because some code is used that is recognised from newer compiler versions only, and throws errors with older compiler versions. Most contracts seem to be using:
pragma solidity >=0.8.0;  
Consider using from 0.8.12 or newer.
Example: Bio.sol >>> L101 > use of string.concat > not recognised by older compiler versions.

1) 
Only state variables or file-level variables can have a docstring.

SOLUTION:
Either dont use docstring for variables that aren't state variables or file-level, or use pragma solidity >=0.8.4;

    /// @notice Data that is stored per PFP
    struct ProfilePictureData {
        /// @notice Reference to the NFT contract
        address nftContract;
        /// @notice Referenced nft ID
        uint256 nftID;
    }

ERROR:

ParserError: Only state variables or file-level variables can have a docstring.
  --> src/ProfilePicture.sol:23:17:
   |
23 |         address nftContract;
   |    