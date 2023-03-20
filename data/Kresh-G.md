## Useless check
### Description
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L60
`(i > 0 && (i + 1) % 40 == 0)` has a check `i > 0` which is not needed, because when i = 0, then 1 % 40 != 0

### Recommended Mitigation Steps
Replace `(i > 0 && (i + 1) % 40 == 0)` with `((i + 1) % 40 == 0)`