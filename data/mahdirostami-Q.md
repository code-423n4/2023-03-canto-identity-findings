## Non-library/interface files should use fixed compiler versions

Contracts should be deployed using the same compiler version/flags with which they have been tested. Locking the pragma (for e.g. by not using ^ or >= in pragma solidity) ensures that contracts do not accidentally get deployed using an older compiler version with unfixed bugs.

Instances:
```
pragma solidity >=0.8.0;
canto-pfp-protocol/src/ProfilePicture.sol
canto-bio-protocol/src/Bio.sol
canto-namespace-protocol/src/Namespace.sol
canto-namespace-protocol/src/Tray.sol
```

## Use delete to clear variables instead of zero assignment
You can use the delete keyword instead of setting the variable as zero.
Instances:

```
canto-bio-protocol/src/Bio.sol
L92       prevByteWasContinuation = false;
L93       bytesOffset = 0;
```
```
canto-pfp-protocol/src/ProfilePicture.sol
L103       nftID = 0;
```

## Events is missing indexed fields
Index event fields make the field more quickly accessible to off-chain.
Each event should use three indexed fields if there are three or more fields.

instance:
```
canto-namespace-protocol/src/Tray.sol
L80    event PrelaunchEnded();
```