# 1. Lines too long

In general, it is a good practice to keep lines of source code within 80 characters in length.
Although, some flexibility is allowed and it is reasonable to let lines be up to 120 characters in some instances.

On modern screens, it is even possible to go beyond this limit.
However, it is recommended to split lines when they reach a length of 164 characters or more, as this is the point at which GitHub will introduce a scroll bar to view the code.

This can help to make the code more readable and easier to work with.

Affected line of code:

- [Bio.sol#L72](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L72)

- [Bio.sol#L41](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L41)

- [Namespace.sol#L117](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L117)

# 2. Follow function order of solidity style guide

The [Solidity style guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions) recommends the following function order:

- constructor

- receive function (if exists)

- fallback function (if exists)

- external

- public

- internal

- private

Within a grouping, place the `view` and `pure` functions last.

This is because "Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier." -solidity style guide

Affected line of code:

- [Tray.sol#L119](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L119)
- [ProfilePicture.sol#L67-L74](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L67-L74)

# 3. Use the `delete` operator to clear variables, rather than assigning a value of `0`/`false`.

To clear variables, consider using the `delete` operator rather than assigning to `false` or zero, because this conveys the intention more clearly and is more idiomatic.

As an example on [line 92-93](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L92-L93) you can refactor the code like so:

```solidity
Line 92:    delete prevByteWasContinuation;
Line 93:    delete bytesOffset;
```

Affected line of code:

- [ProfilePicture.sol#L103](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L103)

- [Bio.sol#L92-L93](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L92-L93)

- [Namespace.sol#L129](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L129)

# 4. Fix typos to improve readability

```solidity
File: canto-namespace-protocol/src/Namespace.sol
line 72:    /// @param _revenueAddress Adress to send the revenue to

...

line  82:    // Register CSR on Canto mainnnet

...

line 152:    // We keep track of the unique trays NFTs (for burning them) and only check the owner once for the last occurence of the tray
```

Consider making the following changes to [Namespace.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol):

- `Adress` To `Address` ([Line 72](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L72))
- `mainnnet` To `mainnet` ([Line 82](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L82))
- `occurence` To `occurrence` ([Line 152](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L152))

```solidity
File: canto-pfp-protocol/src/ProfilePicture.sol
line 61:    // Register CSR on Canto mainnnet

File: canto-bio-protocol/src/Bio.sol
line 34:    // Register CSR on Canto mainnnet

File: canto-namespace-protocol/src/Tray.sol
line 111:    // Register CSR on Canto mainnnet
```

Consider making the following changes to [ProfilePicture.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol), [Tray.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol), and [Bio.sol](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol):

- `mainnnet` To `mainnet` ([Line 61](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L61))

- `mainnnet` To `mainnet` ([Line 111](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L111))
  
- `mainnnet` To `mainnet` ([Line 34](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L34))

# 5. Use newer versions of solidity

Consider using the latest version of solidity as newer versions have bug fixes, as well as new features.

The latest versions provide things like `using for`(`0.8.13` and above.), `string.concat()` instead of `abi.encodePacked()`(`0.8.12` and above.), and `bytes.concat()` instead of `abi.encodePacked()`(`0.8.4` and above.)

Affected lines of code:

- [ProfilePicture.sol#L2](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L2)

- [Bio.sol#L2](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L2)

- [Namespace.sol#L2](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L2)

- [Tray.sol#L2](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L2)

# 6. Add whitespace between comment

To increase the readability of comment codes add at least 1 space at the beginning of single-line comments. If you are using multi-line comments add at least 1 space/newline at the beginning and end.

Here are a few examples of lousy comment spacing:

```solidity
//This is a comment with no whitespace at the beginning

/*This is a comment with no whitespace at the beginning */

/* This is a comment with a whitespace at the beginning but not the end*/
```

Here are a few examples of good comment spacing:

```solidity
// This is a comment with a whitespace at the beginning

/* This is a comment with a whitespace at the beginning */

/*
 * This is a comment with a whitespace at the beginning
 */

/*
This comment has a newline
*/
```

Affected lines of code:

- [Tray.sol#L232](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L232)
- [Tray.sol#L235](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L235)

# 7. Use `_safeMint()` instead of `_mint()`
`_safeMint()` includes additional safety checks that are not present in `_mint`.

Affected lines of code:
- [ProfilePicture.sol#L86](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L86)

- [Bio.sol#L126](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L126)