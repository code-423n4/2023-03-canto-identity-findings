##

### [G-1] USE A MORE RECENT VERSION OF SOLIDITY

Use a Solidity version of at least 0.8.2 to get simple compiler automatic inlining.

Use a Solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads.

Use a Solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than revert()/require() strings.

Use a Solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value.

FILE : 2023-03-canto-identity/canto-pfp-protocol/src/ProfilePicture.sol

   2: pragma solidity >=0.8.0;

[ProfilePicture.sol#L2](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L2)

##

### [G-2] OPTIMIZE NAMES TO SAVE GAS

public/external function names and public member variable names can be optimized to save gas.  Below are the interfaces/abstract contracts that can be optimized so that the most frequently-called functions use the least amount of gas possible during method lookup. Method IDs that have two leading zero bytes can save 128 gas each during deployment, and renaming functions to have lower method IDs will save 22 gas per call, [per sorted position shifted](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92)

FILE : 2023-03-canto-identity/canto-pfp-protocol/src/ProfilePicture.sol

    ///@audit mint(),getPFP() 
    8: contract ProfilePicture is ERC721 {

FILE : 2023-03-canto-identity/canto-bio-protocol/src/Bio.sol

    ///@audit mint()
    9: contract Bio is ERC721 {

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Namespace.sol

    ///@audit fuse(),burn(),changeNoteAddress(),changeRevenueAddress()
    11: contract Namespace is ERC721, Owned {

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Tray.sol

    ///@audit buy(),burn(),getTile(),getTiles(),changeNoteAddress(),changeRevenueAddress(),endPrelaunchPhase(),
    13: contract Tray is ERC721A, Owned {

##

### [G-3]  ADD UNCHECKED {} FOR SUBTRACTIONS WHERE THE OPERANDS CANNOT UNDERFLOW BECAUSE OF A PREVIOUS REQUIRE() OR IF-STATEMENT . This saves 30-40 gas

FILE : 2023-03-canto-identity/canto-bio-protocol/src/Bio.sol

  62: if (i != lengthInBytes - 1) {

[Bio.sol#L62](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L62)

##

### [G-4] CAN MAKE THE VARIABLE OUTSIDE THE LOOP TO SAVE GAS

In Solidity, declaring a variable inside a loop can result in higher gas costs compared to declaring it outside the loop. This is because every time the loop runs, a new instance of the variable is created, which can lead to unnecessary memory allocation and increased gas costs

On the other hand, if you declare a variable outside the loop, the variable is only initialized once, and you can reuse the same instance of the variable inside the loop. This approach can save gas and optimize the execution of your code

GAS SAMPLE TEST IN remix ide(Without gas optimizations) :

Before Mitigation :

     function testGas() public view {

        for(uint256 i = 0; i < 10; i++) {
          address collateral = msg.sender;
          address  collateral1 = msg.sender;
            
        }

The execution Cost : 2176 

After Mitigation :

     function testGas() public view {
    address collateral; address  collateral1;
        for(uint256 i = 0; i < 10; i++) {
         collateral = msg.sender;
         collateral1 = msg.sender;
            
        }

The execution Cost : 2086 . 

>  Hence clearly after mitigation the gas cost is reduced. Approximately its possible to save 9 gas for every iterations 

FILE : 2023-03-canto-identity/canto-bio-protocol/src/Bio.sol

[Bio.sol#L56-L61](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L56-L61)

##

### [G-5] Use nested if and, avoid multiple check combinations

FILE : 2023-03-canto-identity/canto-bio-protocol/src/Bio.sol

[Bio.sol#L60](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L60)

[Bio.sol#L71-L83](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L71-L83)

##

### [G-6] MULTIPLE ADDRESS/ID MAPPINGS CAN BE COMBINED INTO A SINGLE MAPPING OF AN ADDRESS/ID TO A STRUCT, WHERE APPROPRIATE

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Namespace.sol


   mapping(string => uint256) public nameToToken;

    /// @notice Maps NFT IDs to (ASCII) names
    mapping(uint256 => string) public tokenToName;

    /// @notice Stores the character data of an NFT
    mapping(uint256 => Tray.TileData[]) private nftCharacters;

[Namespace.sol#L43-L49](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L43-L49)













GAS-1	Use assembly to check for address(0)	5
GAS-2	Cache array length outside of loop	1
GAS-3	State variables should be cached in stack variables rather than re-reading them from storage	1
GAS-4	Use calldata instead of memory for function arguments that do not get mutated	1
GAS-5	Pre-increments and pre-decrements are cheaper than post-increments and post-decrements	3
GAS-6	Use shift Right/Left instead of division/multiplication if possible	14
GAS-7	Use storage instead of memory for structs/arrays	18
GAS-8	Increments can be unchecked in for-loops	14
GAS-9	Use != 0 instead of > 0 for unsigned integer comparison	6
GAS-10	internal functions not called by the contract should be removed	1