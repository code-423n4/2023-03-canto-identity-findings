# Gas Optimizations

## Summary

|               | Issue         | Instances     |
| :-------------: |:-------------|:-------------:|
| 1  | Multiple address/IDs mappings can be combined into a single mapping of an address/id to a struct | 2 |
| 2  | Using `storage` instead of `memory` for struct/array saves gas | 1 |
| 3  | Duplicated input check statements should be refactored to a modifier  |  3 |
| 4  | `++i/i++` should be `unchecked{++i}`/`unchecked{i++}` when it is not possible for them to overflow, as in the case when used in for & while loops |  12  |


## Findings

### 1- Multiple address/IDs mappings can be combined into a single mapping of an address/id to a struct :

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations.

There are 2 instances of this issue in `Lottery` :

```
File: Namespace.sol

46      mapping(uint256 => string) public tokenToName;
49      mapping(uint256 => Tray.TileData[]) private nftCharacters;
```

These mappings could be refactored into the following struct and mapping for example :

```
struct TokenToNftData {
    string tokenToName
    Tray.TileData[] nftCharacters;
}
    
mapping(uint256 => TokenToNftData) public tokenToNftData;
```


### 2- Using `storage` instead of `memory` for struct/array saves gas :

When fetching data from a `storage` location, assigning the data to a `memory` variable causes all fields of the struct/array to be read from `storage`, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new `memory` variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the `memory` keyword, declaring the variable with the `storage` keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. 

The only time it makes sense to read the whole struct/array into a `memory` variable, is if the full struct/array is being returned by the function, is being passed to a function that requires `memory`, or if the array/struct is being read from another `memory` array/struct.

There  is 1 instance of this issue :

File: Bio.sol [Line 45](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L45)
```
string memory bioText = bio[_id];
```


### 3- Duplicated input check statements should be refactored to a modifier (saves ~400 gas) :

The following check statements are repeated multiple times :

```
File: Tray.sol

120     if (!_exists(_id)) revert TrayNotMinted(_id);
196     if (!_exists(_trayId)) revert TrayNotMinted(_trayId);
204     if (!_exists(_trayId)) revert TrayNotMinted(_trayId);
```

Because this check is done at the start of the function it can be placed and can be refactored into a modifier to save gas, it should be replaced by a `isAllowed` modifier as follow :

```
modifier isAllowed(uint256 _id){
    if (!_exists(_id)) revert TrayNotMinted(_id);
    _;
}
```


### 4- `++i/i++` should be `unchecked{++i}/unchecked{i++}` when it is not possible for them to overflow, as in the case when used in for & while loops :

There are 12 instances of this issue:

```
File: Tray.sol

56      for (uint i; i < lengthInBytes; ++i)
100     for (uint i; i < lines; ++i) 

File: Namespace.sol

122     for (uint256 i; i < numCharacters; ++i)
127     for (uint256 j = i + 1; j < numCharacters; ++j)
147     for (uint256 j; j < numBytesChar; ++j)
174     for (uint256 i; i < numUniqueTrays; ++i)

File: Utils.sol

146     for (uint256 i; i < numAbove; ++i)
155     for (uint256 i; i < numMiddle; ++i)
164     for (uint256 i; i < numBelow; ++i)
225     for (uint256 i; i < _tiles.length; ++i)

File: Bio.sol

56      for (uint i; i < lengthInBytes; ++i)
100     for (uint i; i < lines; ++i)
```