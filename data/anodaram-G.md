## Small optimization in math function

https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L60

In `Bio.sol`, line 60
```
            if ((i > 0 && (i + 1) % 40 == 0) || prevByteWasContinuation || i == lengthInBytes - 1) {
```

Since `(0 + 1) % 40 != 0`, we don't need to check `i > 0`.

Simple we can do like this
```
            if ((i + 1) % 40 == 0 || prevByteWasContinuation || i == lengthInBytes - 1) {
```
