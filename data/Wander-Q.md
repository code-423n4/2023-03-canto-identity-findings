# 1. Attacker can use malicious BioText
## Summary and Detail
an attacker can mint a bio NFT that its biotext contains a malicious script , this allows attacker to run arbitrary script on client .
Due that tokenURI function returns an svg that contains bio text of the nft , so if biotext is a script it will run on client browser .
## Scenario 
1 . Attacker mints a bio nft and enter a script as biotext ( example script : ``<script>alert("Hacked")</script>``) . 
 ```
function mint(string calldata _bio) external {
        // We check the length in bytes, so will be higher for UTF-8 characters. But sufficient for this check
        if (bytes(_bio).length == 0 || bytes(_bio).length > 200) revert InvalidBioLength(bytes(_bio).length);
        uint256 tokenId = ++numMinted;
        bio[tokenId] = _bio;
        _mint(msg.sender, tokenId);
        emit BioAdded(msg.sender, tokenId, _bio);
    }
```
2 . Now tokenURI function returns an svg that contains the script . 
3 . Everyone that sees svg on browser will be affected by script . 
## Lines of code 
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L43-L117
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L119-L128
## Recommendation
Consider validating bioText in mint function.


# 2. Fuzz test failed in Bio.t.sol
```
[FAIL. Reason: Index out of bounds Counterexample: calldata=0x590b00630000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000002900000000000000000000000000000000000000000000000000000000000000000000000000000000f00000000000000000000000000000000000000000000000, args=[                                        �]] testLongString(string) (runs: 56, μ: 974261, ~: 944494)
```