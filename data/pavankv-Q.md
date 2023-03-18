## 1 No check of existence of id in tokenURI(uint256 _id)
code snippet:-
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L43
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L70

Add :-
 if (!exists(_id)) revert NotExisted(_id);