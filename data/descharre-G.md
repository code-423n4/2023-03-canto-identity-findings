# Summary
|ID     | Finding|  Gas saved | Instances|
|:----: | :---           |  :----:         |:----:         |
|G-01      |Require or revert statements that check input arguments should be at the top of the function| - |  1 |
|G-02      |Burn all prelaunch trays and remove every check| - |  1 |
|G-03      |Don't create a seperate loop for burning trays| 2269 |  1 |
|G-04      |Add unchecked block for `_drawing()`  | 150 |  1 |

# Details
## [G-01] Require or revert statements that check input arguments should be at the top of the function
In this case no gas is wasted when a function is destined to fail.
- [ProfilePicture.sol#L81-L82](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L81-L82)
## [G-02] Burn all prelaunch trays and remove every check
The functions [`tokenURI()`](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L122-L125) [`burn()`](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L181-L186) and when you mint/transfer a token [`_beforeTokenTransfers()`](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L231-L243) all check whether the tray is a prelaunch tray or not. The prelaunch trays are useless after the prelaunch period so it's better to burn them all so you don't have to perform all those checks. 

Burning the trays and removing the checks will save gas for every mint, burn and transfer of a tray.

[`_beforeTokenTransfers()`](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L231-L243) can then just be empty.

[`burn()`](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L173-L189)
```diff
    function burn(uint256 _id) external {
        address trayOwner = ownerOf(_id);
        if (
            namespaceNFT != msg.sender &&
            trayOwner != msg.sender &&
            getApproved(_id) != msg.sender &&
            !isApprovedForAll(trayOwner, msg.sender)
        ) revert CallerNotAllowedToBurn();
-        if (msg.sender == namespaceNFT) {
-            // Disallow fusing for prelaunch trays after phase has ended
-            uint256 numPrelaunchMinted = prelaunchMinted;
-            if (numPrelaunchMinted != type(uint256).max && _id <= numPrelaunchMinted)
-                revert PrelaunchTrayCannotBeUsedAfterPrelaunch(_id);
-        }
        delete tiles[_id];
        _burn(_id);
    }
```

Removing the check on [`tokenURI()`](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L122-L125) doesn't save any gas because it's a view function but removes a lot of code then for better readability.

## [G-03] Don't create a seperate loop for burning trays
When you fuse trays, the function [`fuse()`](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L110-L180) holds an array of unique trays so it can burn them at the end of the function. This is not necessary and the trays can be instantly burned instead of adding them to the `uniqueTrays` array. This is not a problem because if function revert all the changes to the state will also revert.

2269 gas can be saved with the following optimization:
```diff
L120:
-   uint256 numUniqueTrays;
-   uint256[] memory uniqueTrays = new uint256[](_characterList.length);

L153:
    if (isLastTrayEntry) {
-       uniqueTrays[numUniqueTrays++] = trayID;
        // Verify address is allowed to fuse
        address trayOwner = tray.ownerOf(trayID);
        if (
            trayOwner != msg.sender &&
            tray.getApproved(trayID) != msg.sender &&
            !tray.isApprovedForAll(trayOwner, msg.sender)
        ) revert CallerNotAllowedToFuse();
+       tray.burn(trayID);
    }

L174:
-   for (uint256 i; i < numUniqueTrays; ++i) {
-       tray.burn(uniqueTrays[i]);
-   }
```
## [G-04] Add unchecked block for `_drawing()` 

The function [`_drawing()`](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L245-L273) does a lot of calculations. There is no risk of over/underflow because there is no input parameter that the user can choose. The default "checked" behavior costs more gas when calculating, because under-the-hood the EVM does extra checks for over/underflow. Adding unchecked for the whole `_drawing()` function saves around 150 gas.

Unchecked can also be done for incremeting or decrementing by 1

