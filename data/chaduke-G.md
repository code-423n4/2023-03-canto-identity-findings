G1. The buy() function keeps modifying state variable ``lastHash`` in a for loop. This is a waste of gas. 
We only need to use a local variable here and only needs to flush the final result back to the state variable ``lastHash``.

```diff
+       bytes32 lastH =  lastHash;
        for (uint256 i; i < _amount; ++i) {
            TileData[TILES_PER_TRAY] memory trayTiledata;
            for (uint256 j; j < TILES_PER_TRAY; ++j) {
-                lastHash = keccak256(abi.encode(lastHash));
+                lastH = keccak256(abi.encode(lastH));
-                trayTiledata[j] = _drawing(uint256(lastHash));
+                trayTiledata[j] = _drawing(uint256(lastH));
            }
            tiles[startingTrayId + i] = trayTiledata;
        }
+       lastHash = lastH;
```