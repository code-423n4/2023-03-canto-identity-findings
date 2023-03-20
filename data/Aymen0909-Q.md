# QA Report

## Summary

|               | Issue         | Risk     | Instances     |
| :-------------: |:-------------|:-------------:|:-------------:|
| 1 | `burn` function doesn't delete the NFT character data | Low | 1 |
| 2 | Immutable state variables lack zero address checks | Low | 3 |
| 3 | Related data should be grouped in a struct | NC | 2 |
| 4 | Duplicated `require()` checks should be refactored to a modifier  | NC | 3 |
| 5 | `constant` should be defined rather than using magic numbers  | NC | 2 |

## Findings

### 1- `burn` function doesn't delete the NFT character data :

In the `burn` function from the Namespace contract, when the NFT is burned its correspanding name (stored in `tokenToName[_id]`) and tokenId (stored in `nameToToken[associatedName]`) are deleted but the NFT character data stored in the `nftCharacters` struct are not deleted and remains even after the burning operation, this can potentially impact the protocol working in the future if this data is used for a given purpose.

#### Risk : Low

#### Proof of Concept

Issue occurs in the code below :

File: Namespace.sol [Line 184-192](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L184-L192)
```
function burn(uint256 _id) external {
    address nftOwner = ownerOf(_id);
    if (nftOwner != msg.sender && getApproved[_id] != msg.sender && !isApprovedForAll[nftOwner][msg.sender])
        revert CallerNotAllowedToBurn();
    string memory associatedName = tokenToName[_id];
    delete tokenToName[_id];
    delete nameToToken[associatedName];
    /** @audit
        Should also delete the nftCharacters data
    */
    _burn(_id);
}
```

#### Mitigation
Delete the NFT character data when burning a given NFT.


### 2- Immutable state variables lack zero address checks  :

Constructors should check the values written in an immutable state variables(address) is not the zero value (address(0))

#### Risk : Low

#### Proof of Concept
Instances include:

File: ProfilePicture.sol [Line 58](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L58)
```
cidNFT = ICidNFT(_cidNFT);
```

File: Tray.sol [Line 109](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L109)
```
namespaceNFT = _namespaceNFT;
```

File: Namespace.sol [Line 78](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L78)
```
tray = Tray(_tray);
```

#### Mitigation
Add non-zero address checks in the constructors for the instances aforementioned.


### 3- Duplicated input check statements should be refactored to a modifier :

Input check statements used multiple times inside a contract should be refactored to a modifier for better readability and also to save gas.

#### Risk : NON CRITICAL

#### Proof of Concept

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


### 4- Related data should be grouped in a struct :

When there are mappings that use the same key value, having separate fields is error prone, for instance in case of deletion or with future new fields.

#### Risk : Non critical

#### Proof of Concept

Instances include:

File: Namespace.sol [Line 46-49](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L46-L49)

```
46      mapping(uint256 => string) public tokenToName;
49      mapping(uint256 => Tray.TileData[]) private nftCharacters;
```

#### Mitigation

Group the related data in a struct and use one mapping :

```
struct TokenToNftData {
    string tokenToName
    Tray.TileData[] nftCharacters;
}
```

And it would be used as a state variable :

```
mapping(uint256 => TokenToNftData) public tokenToNftData;
```


### 5- `constant` should be defined rather than using magic numbers :

It is best practice to use constant variables rather than hex/numeric literal values to make the code easier to understand and maintain, but if they are used those numbers should be well docummented. 

#### Risk : Non critical

#### Proof of Concept

Instances include:

File: Namespace.sol 

[15485863](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L258)

[2038074743](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L259)

#### Mitigation
Replace the hex/numeric literals aforementioned with `constants`.