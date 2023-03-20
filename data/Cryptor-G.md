## The function mint in ProfilePicture.sol does not check if the call is an EOA or not, meaning that a smart contract can call mint multiple times

https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L79-L88

This could be a problem relating to gas costs since the function increases the size of storage struct ProfilePictureData shown here.

https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L83

So it is possible that a smart contract could grief the protocol by calling mint multiple times, inflating the gas costs when ProfilePictureData is retrieved in the function getPFP shown here:

https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L94-L105


It is recommended to require that the user is an EOA before calling the function mint






