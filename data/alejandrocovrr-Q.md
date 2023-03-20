## Improve readibility and adding validation to increase code quality on burn function

On [canto-namespace-protocol/src/Namespace.sol#L184](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L184) the burn function has the following structure: 

```
function burn(uint256 _id) external {
        address nftOwner = ownerOf(_id);
        if (nftOwner != msg.sender && getApproved[_id] != msg.sender && !isApprovedForAll[nftOwner][msg.sender])
            revert CallerNotAllowedToBurn();
        string memory associatedName = tokenToName[_id];
        delete tokenToName[_id];
        delete nameToToken[associatedName];
        _burn(_id);
}
```

I'm suggesting the following code as a refactor for QA purposes: 

```
function burn(uint256 _id) external {
    require(_id != 0, "Invalid ID");
    
    address nftOwner = ownerOf(_id);
    require(
        nftOwner == msg.sender ||
        getApproved(_id) == msg.sender ||
        isApprovedForAll[nftOwner][msg.sender],
        "Caller not authorized to burn"
    );
    
    string memory associatedName = tokenToName[_id];
    require(bytes(associatedName).length > 0, "No name associated with token");
    
    delete tokenToName[_id];
    delete nameToToken[associatedName];
    _burn(_id);
}
```

In this revised code:

- Input validation is added to check that the _id parameter is not zero.
- The require statement is used to check that the caller is authorized to burn the NFT. This replaces the if statement and makes the code easier to read and less error-prone.
- A check is added to ensure that there is a name associated with the token being burned.
- The error message in the require statement is made more generic to avoid revealing information about the contract's internal workings.
- The getApproved function is called using the correct syntax for a function call.