
# Gas Optimizations Report

This report focuses on Canto Identity Protocol contest, in context of various improvements that can be made in terms of gas cost.

Some of the opportunities identified for improving gas efficiency throughout the codebase of Canto Identity Protocol are categorised into 05 main areas; with further multiple instances in each of the category.


# [G-01] Uint/Int lower than 32 bytes consumes more gas (05 Instances)

Contract gas usage increases as EVM standard operation are of 32 bytes. If any element is smaller than 32 bytes (i.e.; 256 bits) it will cause EVM to consume more gas which can be around 12 gas depending on size for reducing the size to given output like uint8.

Link to the Code:
1.	https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L125
2.	https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L134
3.	https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L12
4.	https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L12
5.	https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L267

# [G-02] 0perator assignment is more gas efficient than compound assignment (01 Instance)

Compound assignment operators (+= / -=) are more expensive in terms of gas consumption and needs to be avoided.

Operator assignment (a = a + b / a - b) is preferable in terms of increasing gas optimization.

Link to the Code:
1.	https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L150

# [G-03] Public visibility consumes more gas as compared to external functions (03 Instances)

Functions with public visibility, if not called within the contract needed to be changed external.

Link to the Code:

1.	https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L43
2.	https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L90
3.	https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L119


# [G-04] Multiple mappings can be combine into a single one (03 Instances)

When multiple mappings are used in same function, it’s better to combined them into a single mapping of an address struct.

Combined mapping reduces storage slot per mapping and also are cheaper in terms of associated Stack operations calculation carried out.

Link to the Code:
1.	https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L43
2.	https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L46
3.	https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L49


# [G-05] Immutable has more gas efficiency than constant (07 Instances)

Using immutable instead of constant, save more gas due to avoiding storage access for state variables.

Variable values are set through constructor when using immutable, which also eliminates the need of assigning initial values to state variable making them more efficient in terms of gas cost

Link to the Code:
1.	https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L19
2.	https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L22
3.	https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L25
4.	https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L28
5.	https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L31
6.	https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L34
7.	https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L37
