## [L-01] Emojis split in different lines.

### Impact

User won’t be able to put the desired bio.

### POC

Add the following code in Bio.t.sol :

```solidity
function testEmoji_split() public {
        string memory text = unicode"AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA🧑🏻‍❤️‍💋‍🧑🏿AAAA";
        uint256 len = bytes(text).length;
        // assertEq(len, 64);
        bio.mint(text);
        uint256 tokenId = bio.numMinted();
        string memory uri = bio.tokenURI(tokenId);
        string memory json = string(Base64.decode(slice(uri, 29, bytes(uri).length)));
        string memory svg = string(Base64.decode(slice(json, 74 + bytes(text).length, bytes(json).length - 2)));
        console.log(svg);
        // make sure the svg still contain the complete emoji
        assertEq(countSubStr(svg, unicode"🧑🏻‍❤️‍💋‍🧑🏿"), 0);
    }
```

### Details

The bio.sol file tries to make sure that the emojis are not split between different lines and does it using the following code:

```solidity
// Do not split when the prev. or next character is a zero width joiner. Otherwise, 👨‍👧‍👦 could become 👨>‍👧‍👦
                    // Furthermore, do not split when next character is skin tone modifier to avoid 🤦‍♂️\n🏻
                    if (
                        // Note that we do not need to check i < lengthInBytes - 4, because we assume that it's a valid UTF8 string and these prefixes imply that another byte follows
                        (nextCharacter == 0xE2 && bioTextBytes[i + 2] == 0x80 && bioTextBytes[i + 3] == 0x8D) ||
                        (nextCharacter == 0xF0 &&
                            bioTextBytes[i + 2] == 0x9F &&
                            bioTextBytes[i + 3] == 0x8F &&
                            uint8(bioTextBytes[i + 4]) >= 187 &&
                            uint8(bioTextBytes[i + 4]) <= 191) ||
                        (i >= 2 &&
                            bioTextBytes[i - 2] == 0xE2 &&
                            bioTextBytes[i - 1] == 0x80 &&
                            bioTextBytes[i] == 0x8D)
                    ) {
                        prevByteWasContinuation = true;
                        continue;
											}
```

For emoji in the above POC, the second check would fail and it would split it. Basically there are many emojis that are split because the second check is incomplete as this emoji converted in hex starts with :

```solidity
0xf0 0x9f 0xa7
```

### Recommendation

A few more checks needed to be placed for emojis like this, a full list of emojis can be found here: [https://unicode.org/Public/emoji/15.0/emoji-test.txt](https://unicode.org/Public/emoji/15.0/emoji-test.txt)

## [L-02] `_nftContract` address not verified.

### Impact

In function `mint` in the contract `ProfilePicture` , the `_nftContract` address is not verified, this means that the address can be put arbitrarily by an attacker. The attacker can manipulate the value returned by `tokenURI` on their token ID, which can be used for social engineering attacks. Eg: If the token is displayed on an NFT marketplace, they can put an image of a legit NFT and put a phishing link over there.

### POC

```solidity
function mint(address _nftContract, uint256 _nftID) external { //@audit _nftContract address not checked
        uint256 tokenId = ++numMinted;
        if (ERC721(_nftContract).ownerOf(_nftID) != msg.sender)
            revert PFPNotOwnedByCaller(msg.sender, _nftContract, _nftID);
        ProfilePictureData storage pictureData = pfp[tokenId];
        pictureData.nftContract = _nftContract;
        pictureData.nftID = _nftID;
        _mint(msg.sender, tokenId);
        emit PfpAdded(msg.sender, tokenId, _nftContract, _nftID);
    }
```

### Tools Used

Manual Analysis

### Recommendation

A whitelist of allowed addresses can be put here.