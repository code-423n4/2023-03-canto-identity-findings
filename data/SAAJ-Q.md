
# Low Risk and Non-Critical Issues

# [L-01] Minting tokens to the zero address should be avoided (04 Instances)

Address(0) check is missing in mint function, consider applying check to ensure tokens aren’t minted to the zero address.

Link to the code:
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L86
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L126
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L177
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L167


# [N-01] Floating of Pragma (05 Instances)
Locking pragma version ensures contracts are not being deployed on a version with known issues.

Link to the code:
1.	https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L2
2.	https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L2
3.	https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L2
4.	https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L2
5.	https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L2

# [N-02] Outdated Solidity Version (05 Instances)
Outdated version of solidity is used for all contracts in scope.
Recent version should be used to avoid known bugs that can impact the project negatively.


# [N-03] Constructor lacks address(0) check (03 Instances)

Zero-address check should be used in the constructor, to avoid the risk of setting a storage variable as address(0) at deploying time.

Link to the code:
1.	https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L57
2.	https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L73
3.	https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L98
