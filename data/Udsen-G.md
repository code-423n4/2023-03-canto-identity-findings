## 1. USE THE CACHED VALUE `numCharacters` IN PLACE OF THE `_characterList.length` TO SAVE GAS, RATHER THAN CALLING IT REPETITIVELY.

In the `fuse()` function of the `Namespace.sol` contract, the `_characterList.length` value is cached as below:

        uint256 numCharacters = _characterList.length;
		
Hence the `numCharacters` can be used for the following `_characterList.length` calls. But the function again calls the `_characterList.length` as follows:

        uint256[] memory uniqueTrays = new uint256[](_characterList.length);
		
Hence this can be replaced with the `numCharacters` variable as follows to save gas.

        uint256[] memory uniqueTrays = new uint256[](numCharacters);
		
Hence gas can be saved since only memory read is required when cached value is used, instead of calculating the `length` of the calldata array again, which consumes more gas.

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L121