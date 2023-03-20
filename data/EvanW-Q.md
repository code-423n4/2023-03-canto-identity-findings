[NC-01] The for-loop for concatenating the html content in **tokenURI** function should use **insertedLines** as limit instead of **lines**.

```
canto-bio-protocol/src/Bio.sol
100:    for (uint i; i < lines; ++i) {
101:         text = string.concat(text, '<tspan x="50%" dy="20">', strLines[i], "</tspan>");
102:    }
```
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L100-L102

[NC-02] Missing checks for address(0) when assigning values to address state variables

```
canto-namespace-protocol/src/Namespace.sol
198:        note = ERC20(_newNoteAddress);

canto-namespace-protocol/src/Tray.sol
212:        note = ERC20(_newNoteAddress);
```
