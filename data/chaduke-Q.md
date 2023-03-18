QA1. Zero address check is needed for ``_newNoteAddress`` and ``_newRevenueAddress``. Otherwise, uses might mint Tray NFTs for free or revenues might be sent to zero address.

[https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L210-L222](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L210-L222)

