# 1: WORD & TYPING TYPOS

Vulnerability details

## Context:

Word & typing typos

## Proof of Concept

> ***File: Namespace.sol*** 

occurence can be changed to occurrence in the following comment.

https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L152

### Tools Used

Manual Analysis

# 2: FOR MODERN AND MORE READABLE CODE; UPDATE IMPORT USAGE

Vulnerability details

## Context:

Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct polluted the source code with an unnecessary object we were not using because we did not need it.
This was breaking the rule of modularity and modular programming: only import what you need Specific imports with curly braces to allow us to apply this rule better.

## Proof of Concept

#### Contracts 

> ***File: src/ProfilePicture.sol.sol***

https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L5-L6

> ***File: src/Bio..sol***

https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L7

> ***File: src/Namespace.sol***

https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L7-L9

> ***File: src/Tray.sol***

https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L10-L11

> ***File: src/Utils.sol***

https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L4

## Tools Used

Manual Analysis

### Recommended Mitigation Steps

import {contract1 , contract2} from "filename.sol";

#### Example:

import {Owned} from "solmate/auth/Owned.sol";
import {ERC721} from "solmate/tokens/ERC721.sol";
import {LibString} from "solmate/utils/LibString.sol";
import {MerkleProofLib} from "solmate/utils/MerkleProofLib.sol";
import {FixedPointMathLib} from "solmate/utils/FixedPointMathLib.sol";
import {ERC1155, ERC1155TokenReceiver} from "solmate/tokens/ERC1155.sol";
import {toWadUnsafe, toDaysWadUnsafe} from "solmate/utils/SignedWadMath.sol";

# 3: ADD A LIMIT FOR THE MAXIMUM NUMBER OF CHARACTERS PER LINE

Vulnerability details

## Context:

The solidity documentation recommends a maximum of 120 characters.

For reference, see https://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-length

## Proof of Concept

> ***2 Files, 5 Instances*** 

https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L98

https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L21

https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L28

https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L43

https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L247

## Tools Used

Manual Analysis

### Recommended Mitigation Steps

Consider adding a limit of 120 characters or less to prevent large lines.

# 4: INTERCHANGEABLE USAGE OF UINT AND UINT256

Vulnerability details

## Context:

Interchangeable usage of uint and uint256. Below are instances where uint was used rather than uint256 like in the rest of the code.

## Proof of Concept

> ***File: src/Bio.sol***

https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L47

https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L49

https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L55-L56

https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L100

## Tools Used

Manual Analysis

### Recommended Mitigation Steps

Consider using only one approach throughout the codebase, e.g. only uint or only uint256.

# 5: ADD TO BLACKLIST FUNCTION

Vulnerability details

## Context:

NFT thefts have increased recently, so with the addition of hacked NFTs to the platform, NFTs can be converted into liquidity. To prevent this, I recommend adding the blacklist function.

Marketplaces such as Opensea have a blacklist feature that will not list NFTs that have been reported theft, NFT projects such as Manifold have blacklist functions in their smart contracts.

Here is the project example; Manifold

Manifold Contract
https://etherscan.io/address/0xe4e4003afe3765aca8149a82fc064c0b125b9e5a#code
	
## Tools Used

Manual Analysis

### Recommended Mitigation Steps

Add to Blacklist function and modifier.

> ***Example***

    modifier nonBlacklistRequired(address extension) {
        require(!_blacklistedExtensions.contains(extension), "Extension blacklisted");
        _;
    }

# 6: USE UNDERSCORES FOR NUMBER LITERALS

Vulnerability details

## Context:

There are occasions where certain numbers have been hardcoded, either in variables or in the code itself. Large numbers can become hard to read.

## Proof of Concept

> ***File: src/Utils.sol*** 

https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L258-L259

## Tools Used

Manual Analysis

### Recommended Mitigation Steps

Consider using underscores for number literals to improve their readability.

# 7: UPPERCASE IMMUTABLE VARIABLES

Vulnerability details

## Context:

Uppercase immutable variables


## Proof of Concept

> ***File: Tray.sol***

https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L37

uint256 public immutable trayPrice;

https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L50

address public immutable namespaceNFT;

> ***File: Namespace.sol***

https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L17

Tray public immutable tray;

> ***File: ProfilePicture.sol***

https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L14

ICidNFT private immutable cidNFT;

## Tools Used

Manual Analysis

### Recommended Mitigation Steps

Immutable Variables should be UPPERCASE

# 8: MISSING NATSPEC

Vulnerability details

## Context:

Missing NatSpec

Consider adding specifications to the following code blocks:

## Proof of Concept

+ ***File: Tray.sol***

    error CallerNotAllowedToBurn();
    error TrayNotMinted(uint256 tokenID);
    error OnlyOwnerCanMintPreLaunch();
    error MintExceedsPreLaunchAmount();
    error PrelaunchTrayCannotBeUsedAfterPrelaunch(uint256 startTokenId);
    error PrelaunchAlreadyEnded();

https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L85-L90

+ ***File: Namespace.sol***

https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L61-L67

+ ***File: Bio.sol***

https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L28-L29

+ ***File: ProfilePicture.sol***
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L50-L52

+ ***File: Utils.sol***

https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L12-L13

## Tools Used

Manual Analysis

### Recommended Mitigation Steps

Add NatSpec comments accordingly.

# 9: PROJECT UPGRADE AND STOP SCENARIO SHOULD BE

Vulnerability details

## Context:

At the start of the project, the system may need to be stopped or upgraded, I suggest you have a script beforehand and add it to the documentation.
This can also be called an ” EMERGENCY STOP (CIRCUIT BREAKER) PATTERN “.

For reference, see https://github.com/maxwoe/solidity_patterns/blob/master/security/EmergencyStop.sol

## Tools Used

Manual Analysis

### Recommended Mitigation Steps

Create upgrade and stop scenario

