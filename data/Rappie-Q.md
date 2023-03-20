## `Tray.buy` runs out of gas when buying a large amount
The function `Tray.buy` takes in an `_amount` argument representing the amount of trays to buy. When the amount is above a certain threshold the function runs out of gas. This threshold seems to be around 73.

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L150

## Use custom error in `Tray.getTile` if tile offset is out of range

The function `Tray.getTile` takes in an `_tileOffset` argument representing the offset of the desired tile to retrieve. If this offset is out of bounds an "Index out of bounds" revert occurs. Consider replacing this with a custom error.

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L197
