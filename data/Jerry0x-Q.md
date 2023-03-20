```
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L113
```

```
uint256 fusingCosts = 2**(13 - numCharacters) * 1e18;
```
To
```
uint256 fusingCosts = 2**(13 - numCharacters) * 10**note.decimals();
```
Or modify the changeNoteAddress function.
```
function changeNoteAddress(address _newNoteAddress) external onlyOwner {
        require(ERC20(_newNoteAddress).decimals() == 18, "Invalid decimals for $NOTE token");
        address currentNoteAddress = address(note);
        note = ERC20(_newNoteAddress);
        emit NoteAddressUpdate(currentNoteAddress, _newNoteAddress);
    }
```
It is impossible to determine whether the modified ERC20 decimal places are the same.