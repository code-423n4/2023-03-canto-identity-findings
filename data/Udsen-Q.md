## 1. `delete` THE VARIABLE WHEN RESETTING IT, INSTEAD OF ASSIGNING THE VALUE TO ZERO

            nftID = 0; 

It is recommended to use `delete` to reset the variable instead of assigning zero value to the variable.

            delete nftID; 
			
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L103

There is 1 more instances of this issue:

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L92-L93

## 2. MENTION THE `uint` SIZE EXPLICITLY WHEN DECLARING `uint` VARIABLES

The `uint` variables are declared as both `uint` and `uint256` in the `Bio.sol` contract.
It is recommended to declare all the `uint` variables with thier sizes explicitly for the improved code readability and understanding.

        uint lengthInBytes = bioTextBytes.length; 

The above can be declared as follows:

        uint256 lengthInBytes = bioTextBytes.length; 

So all the `uint` variables will be declared explicitly with thier respective sizes.

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L47		

There are 2 more instances of this issue:

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L49
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L55

## 3. `_tileOffset` SHOULD BE EXPLICITLY CHECKED TO SEE IF THE VALUE IS BETWEEN `0 - TILES_PER_TRAY - 1`.

The tiles per tray is given the value of `TILES_PER_TRAY = 7` in the `Tray.sol` contract.

In the `getTile` function of the `Tray.sol` contract the `tiledata` is returned as follows:

        tileData = tiles[_trayId][_tileOffset];
		
Here the value of `_tileOffset` should be between `0 - TILES_PER_TRAY - 1`. But it is not explicitly checked for that condition.
Hence the above command could try to access a `TileData` value which is out of bound. Which could result in unexpected code behaviour.

Hence it is recommended to check for the `_tileOffset` value to be with in the range of `0 - TILES_PER_TRAY - 1` as shown below:

	require(_tileOffset <= (TILES_PER_TRAY - 1), "Requested Tile per tray has exceeded the bound");

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L195-L198

## 4. NON-LIBRARY/INTERFACE FILES SHOULD USE FIXED COMPILER VERSIONS, NOT FLOATING ONES

Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

	pragma solidity >=0.8.0;

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L2

There are 4 more instances of this issue:

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L2
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L2
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L2
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L2

## 5. USE NAMED IMPORTS INSTEAD OF PLAIN `IMPORT ‘FILE.SOL’

Can import the required specific contracts, functions or variables by using the named imports explicitly. Plain imports will import the entire context of the imported contract which could lead into variable name conflicts etc ...

Currently the `Turnstile` is imported as follows:

	import "../interface/Turnstile.sol";

But it can be imported explicitly by the name as follows:

	import {Turnstile} from "../interface/Turnstile.sol";

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L5-L6

There are 4 more instances of this issue:

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L7
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L7-L9
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L10-L11
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L4