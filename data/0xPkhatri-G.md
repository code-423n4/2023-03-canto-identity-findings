# No state change use memory instead of storage for gas save.

https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L96

`getPFP` function not changing any state, not updating state variable so no need to write storage in pictureData variable, instead use memory it will save gas.
