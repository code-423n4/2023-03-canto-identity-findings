# L-01 Add two-step ownership transfer

Solmate's `Owned` contract used for `Namespace` and `Tray` doesn't implement two-step ownership transfer.

https://github.com/transmissions11/solmate/blob/main/src/auth/Owned.sol#L39
```solidity
    function transferOwnership(address newOwner) public virtual onlyOwner {
        owner = newOwner;

        emit OwnershipTransferred(msg.sender, newOwner);
    }
```

Wrongly specified `newOwner` will lead to severe consequences, therefore introducing a two-step process should be considered.

# NC-01 String type should not be marked as indexed in an event

In the events `BioAdded` and `NamespaceFused` there are string type properties marked as `indexed`. 

```solidity
event BioAdded(address indexed minter, uint256 indexed nftID, string indexed bio);

event NamespaceFused(address indexed fuser, uint256 indexed namespaceId, string indexed name);
```

String is a reference type. Marking it with `indexed` means that a hash of the value will be logged to the topic, while the strings themselves will not be searchable or retrievable from events. It is better to remove `indexed` so that the string value is stored in the data field.

# NC-02 Use string character length when enforcing bio length

Some UTF-8 characters take more bytes than others.

```solidity
    // We check the length in bytes, so will be higher for UTF-8 characters. But sufficient for this check
    if (bytes(_bio).length == 0 || bytes(_bio).length > 200) revert InvalidBioLength(bytes(_bio).length);
```

Authors ackonwledge this issue but don't explain the reasoning for not fixing it. Character length can be determined using `stringutils` [library](https://github.com/Arachnid/solidity-stringutils/tree/master#getting-the-character-length-of-a-string) or a custom implementation.

# NC-03 nftCharacters is not cleaned up when Namespace is burned

Namespace `burn` function doesn't delete nftCharacters mapping entry for the given NFT. This causes unnecessary bloat for Canto chain

```solidity
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

# NC-04 Improve test coverage for Utils.sol

Code in `Utils.sol` is quite complex with a lot of branching. At the same time, code coverage is poor.

Lines coverage: 29.5%

Small changes in this codebase can introduce bugs. Consider adding more tests for uncovered code.

# NC-05 Consider adding oracle randomness for tray generation

Current implementation of tray generation is pseudorandom. This can lead to a situation where the rarest symbols are minted by those who have more resources for front-running and ahead-of-time calculations. Knowing results in advance can also prevent users from minting, resulting in less fees for the protocol.

True randomness based on oracles can generate more engagement for the protocol.