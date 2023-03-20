1)line 115 in namespace should be 
uint256 namespaceIDToMint;
unchecked{namespaceIDToMint = ++nextNamespaceIDToMint }(this will save 42wei)
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L115

2)line 118 ,120,121 of Namespace should be created above their cycle and not in the top of the function,in case of revert txn will save gas
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L118
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L120
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L121

3)line 114 in Namespace should be 
        uint256 fusingCosts;
        unchecked{
              fusingCosts = 2**(13 - numCharacters) * 1e18;
        }
saves 138 wei
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L113

4)unchecked,line 59 in Bio
            unchecked{
                ++bytesOffset;
            }
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L59

5)it's useless to check "prevByteWasContinuation" in line 60 in Bio,because the variable was just declared and not manipulated yet saves 121 wei
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L60

6-7)by removing all prevByteWasContinuation and leave {continue} at line 79 in Bio saves 211 wei,without breaking the code,because that variable wont be used
by making the function look like this:
https://github.com/dicolas/Code4rena/blob/4ffefc9fbbcb574bf68e1c6ed6ca5be92278e2bf/GasCanto#L2-L69

other than that,we dont need to check all of this data just to change "prevByteWasContinuation" because that wont be used,so for that reason,the big if block in Bio can be removed by saving 577 wei
making the final solution of the 6-7,something like this
https://github.com/dicolas/Code4rena/blob/4ffefc9fbbcb574bf68e1c6ed6ca5be92278e2bf/GasCanto#L72-L124