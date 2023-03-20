# Canto Identity Subprotocols by brianspha

##### The code was reviewed for a total of 10 hours.

# [L-01] [NameSpace] Failure to check of address (0) in constructor

## Proof of Concept

The constructor in the `NameSpace.sol` file fails to check against the zero address for the following paramaters

- \_tray
- \_note
- \_revenueAddress

Although there is a function that can update these addresses these respective functions namely

- changeNoteAddress
- changeRevenueAddress
  Fail to check against zero address

## Impact

Any contract calls made to the tray or note contracts will fail. Any revenue tokens sent in the `fuse` function will be sent to the zero address unless this is acceptable im assuming its not

## Recommendation

Make the following change:

```diff
- constructor(
        address _tray,
        address _note,
        address _revenueAddress
    ) ERC721("Namespace", "NS") Owned(msg.sender) {
        tray = Tray(_tray);
        note = ERC20(_note);
        revenueAddress = _revenueAddress;
    }
+ constructor(
        address _tray,
        address _note,
        address _revenueAddress
    ) ERC721("Namespace", "NS") Owned(msg.sender) {
        assert(
            !Utils.isZeroAddress(_tray) ||
               !Utils.isZeroAddress(_note) ||
                !Utils.isZeroAddress(_revenueAddress)
        );
        tray = Tray(_tray);
        note = ERC20(_note);
        revenueAddress = _revenueAddress;
    }

```

```diff
-     function changeNoteAddress(address _newNoteAddress) external onlyOwner {
        address currentNoteAddress = address(note);
        note = ERC20(_newNoteAddress);
        emit NoteAddressUpdate(currentNoteAddress, _newNoteAddress);
    }
+ function changeNoteAddress(address _newNoteAddress) external onlyOwner {
        assert(!Utils.isZeroAddress(_newNoteAddress));
        address currentNoteAddress = address(note);
        note = ERC20(_newNoteAddress);
        emit NoteAddressUpdate(currentNoteAddress, _newNoteAddress);
    }
```

```diff
-     function changeNoteAddress(address _newNoteAddress) external onlyOwner {
        address currentNoteAddress = address(note);
    }
+ function changeNoteAddress(address _newNoteAddress) external onlyOwner {
        assert(!Utils.isZeroAddress(_newNoteAddress));
        address currentNoteAddress = address(note);
    }
```

```diff
Under the Utils library this function could be included
+     function isZeroAddress(
        address toCheck
    ) public pure returns (bool) {
        assembly {
            if iszero(toCheck) {
                let ptr := mload(0x40)
                mstore(
                    ptr,
                    0xd92e233d00000000000000000000000000000000000000000000000000000000
                ) // Selector for `ZeroAddress()`
                revert(ptr, 0x4)
            }
        }
        return false;
    }
```

# [L-02] [ProfilePicture] Failure to check of address (0) in constructor

The constructor in the `ProfilePicture.sol` file fails to check against the zero address for the following paramaters

- \_cidNFT

## Impact

Any contract calls made to the \_cidNFT contract will fail.

## Recommendation

Make the following change:

```diff
-   constructor(address _cidNFT, string memory _subprotocolName) ERC721("Profile Picture", "PFP") {
        cidNFT = ICidNFT(_cidNFT);

    }
+   constructor(address _cidNFT, string memory _subprotocolName) ERC721("Profile Picture", "PFP") {
       assert(!Utils.isZeroAddress(_cidNFT));
       cidNFT = ICidNFT(_cidNFT);

    }

```
