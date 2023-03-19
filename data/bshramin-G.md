## In the Tray contract, the Buy function
Change the for loop:
```solidity
for (uint256 i; i < _amount; ++i) {
  TileData[TILES_PER_TRAY] memory trayTiledata;
  for (uint256 j; j < TILES_PER_TRAY; ++j) {
      lastHash = keccak256(abi.encode(lastHash));
      trayTiledata[j] = _drawing(uint256(lastHash));
  }
  tiles[startingTrayId + i] = trayTiledata;
}
```

To:
```solidity
for (uint256 i; i < _amount; ++i) {
  TileData[TILES_PER_TRAY] memory trayTiledata;
  bytes32 _lastHash = lastHash;
  for (uint256 j; j < TILES_PER_TRAY; ++j) {
      _lastHash = keccak256(abi.encode(_lastHash));
      trayTiledata[j] = _drawing(uint256(_lastHash));
  }
  lastHash = _lastHash;
  tiles[startingTrayId + i] = trayTiledata;
}
```


Gas report changes before and after modification:

| Function Name            | min             | avg    | median | max    | # calls |
|--------------------------|-----------------|--------|--------|--------|---------|
| original buy | 1127 | 4028910 | 225888 | 176060229 | 51 |
| modified buy | 1115 | 3980629 | 223752 | 173936217 | 51 |

