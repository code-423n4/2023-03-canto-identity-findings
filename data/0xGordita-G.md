# [G-01] Avoid duplicate calculation of lengths. 

By calculating the length of `_bio` only once and using the variable for further checks, we can save gas in the `mint` function.
```
canto-bio-protocol\src\Bio.sol:
124 if (bytes(_bio).length == 0 || bytes(_bio).length > 200) revert InvalidBioLength(bytes(_bio).length);
```
_characterList.length is calculated again after being assigned to numCharacters
numCharacters can be used when declaring uniqueTrays instead of computing the length a second time for _characterList.
```
canto-namespace-protocol/src/Namespace.sol:
121 uint256[] memory uniqueTrays = new uint256[](_characterList.length);    
```
_tiles.length is calculated twice in utils. Additionally storing the array length beforehand for for-loops is more efficient.
```
		  canto-namespace-protocol/src/Utils.sol:
		    224 uint256 dx = 400 / (_tiles.length + 1);
		    225 for (uint256 i; i < _tiles.length; ++i) {
```