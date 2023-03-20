## String Indexed in event
### Description 
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L23
if there is string indexed it hashing and you can't search for it.
Topic about that:
https://ethereum.stackexchange.com/questions/6840/indexed-event-with-string-not-getting-logged
https://stackoverflow.com/questions/73232215/how-to-decode-the-indexed-string-param-in-an-event-using-web3-js
### Recommendation
Use not indexed string.
## Better constructor
In bio.sol it's better to revert if chainid is not 7700. Also better to pass address of `Turnstile` in calldata. There can be some migration's and to prevent future mistakes.

Better:
```
    constructor(address turnstile_address) ERC721("Biography", "Bio") {
        if (block.chainid == 7700) {
            // Register CSR on Canto mainnnet
            Turnstile turnstile = Turnstile(turnstile_address);
            turnstile.register(tx.origin);
        }
        else {
            revert("Wrong Network");
        }
    }
```
