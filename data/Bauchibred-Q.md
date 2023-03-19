# Canto Identity Subprotocols QA Report

# Low Issues

## L-01 Incomplete input parameter validation

### Impact

The `Tray.getTile()` function would always return 0 when `_tileOffset>=7` but users might not know why

### Proof of concept

The `Tray..getTile()` function provides information about one tile and it takes in two parameters `uint256 _trayId` and `uint8 _tileOffset)`, the function does include a check for when the `_trayId` is non existent, [L196](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L196) of Tray.sol:
`        if (!_exists(_trayId)) revert TrayNotMinted(_trayId);`
But there are no checks to confirm that `_tileOffset < 7`, and its explicitly stated that `_tileOffset` needs to be between `0 .. TILES_PER_TRAY - 1` in the param natspec of `_tileOffset` at L194:
`/// @param _tileOffset Offset of the tile within the query, needs to be between 0 .. TILES_PER_TRAY - 1` meaning that when a user calls this function with an invalid tile offset they get a wrong return value and might not understand what the issue is.

### Recommended Mitigation Steps

Create an additional error case:
` error InvalidTileOffset(uint8 _tileOffset)`
Add a conditiion to `Tray.getTile()` that `_tileOffset` must be between `0 .. TILES_PER_TRAY - 1` 
function getTile(uint256 _trayId, uint8 _tileOffset) external view returns (TileData memory tileData) {
        if (!_exists(_trayId)) revert TrayNotMinted(_trayId);
        `if(_tileOffset >= 7) revert  InvalidTileOffset(uint8 _tileOffset)`
        tileData = tiles[_trayId][_tileOffset];
    }
 

## L-02 Changing `note` and/or `revenue` addresses to the zero address should be avoided.

### Impact

If both or either of `note` and `revenue` addresses gets set to the zero address, any call to these contracts while being set to 0 is going to lead to unexpected behaviour such as contract reverts and force redeployments

### Proof of concept

There are no zero address checks in the functions `Namepsace.changeNoteAddress()` and `Namepsace.changeRevenueAddress()`
` function changeNoteAddress(address _newNoteAddress) external onlyOwner {
        address currentNoteAddress = address(note);
        note = ERC20(_newNoteAddress);
        emit NoteAddressUpdate(currentNoteAddress, _newNoteAddress);
    }`

`function changeRevenueAddress(address _newRevenueAddress)
        external
        onlyOwner
    {
        address currentRevenueAddress = revenueAddress;
        revenueAddress = _newRevenueAddress;
        emit RevenueAddressUpdated(currentRevenueAddress, _newRevenueAddress);
    }`

### Recommended Mitigation Steps

Zero address checks should be added

# Non-Critical Issues

## NC-01 Not completely accurate error name in the `Namespace.fuse()` function

### Impact

Users might not understand why their transaction is reverting

### Proof of concept

The revert error `InvalidNumberOfCharacters()` is only used once in Namespace.sol at [L112](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L112)
`        if (numCharacters > 13 || numCharacters == 0) revert InvalidNumberOfCharacters(numCharacters);`
As evident it's used to revert the case when the number of characters are more than 13, but the name of the error suggests otherwise and might lead users to not really understand why their transaction is reverting.

### Recommended Mitigation Steps

Change error name from InvalidNumberOfCharacters(uint256 numCharacters) to NumberOfCharactersTooLong(uint256 numCharacters)

## NC-02 Simplify code

### Impact

Simplified code can help users/developers understand and use code easier.

### Proof of concept

Utils.sol [L179-181](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L179-L181) Includes script and also comments before these lines indicate that the 3 characters come from a different UTF-8 block, and therefore the values got hardcoded, but for all three cases the value returned are the same and as such all three conditions can be combined into one with the help of the OR operator.

### Recommended Mitigation Steps

Change the below:
`if (_characterIndex == 4) return hex"F09D9192";`
`if (_characterIndex == 6) return hex"F09D9194";`
`if (_characterIndex == 14) return hex"F09D919C";`

to:
`if (_characterIndex == 4 || _characterIndex == 6 || _characterIndex == 14) ) return hex"F09D9192";`
