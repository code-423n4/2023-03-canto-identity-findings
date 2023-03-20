This is really more of a strong suggestion than a bug finding, but for running tests, I would highly recommend using real contracts instead of Mocks for everything other than the simplest contracts (such as standard ERC20 and ERC721 contracts). Specifically, for testing the PFP Protocol, Mocks are used for AddressRegistry and even the primary CidNFT contract. For achieving full coverage, I would not consider this to be acceptable.

That being said, I wasn't able to find a vulnerability, even after recreating the tests with the real contracts. 

The Mock in question is imported here:

https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/test/ProfilePicture.t.sol#L9

and used here:

https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/test/ProfilePicture.t.sol#L82-L87

I realize this submission is technically out of scope, but I hope the judges will be merciful as this is my first attempt at an audit. 