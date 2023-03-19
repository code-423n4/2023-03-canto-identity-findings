
The protocol `ProfilePicture.sol` has the state variable `subprotocolName` in line number 35. which is a string data type. This result in excessive gas usage when the variable is accessed repeatedly since string variables are expensive to store and manipulate.

`contract ProfilePicture is ERC721 {
    string public subprotocolName;
    constructor(address _cidNFT, string memory _subprotocolName) ERC721("Profile Picture", "PFP") {
        cidNFT = ICidNFT(_cidNFT);
        subprotocolName = _subprotocolName;  }
    function tokenURI(uint256 _id) public view override returns (string memory) {
    }}`

the `subprotocolName` variable should be changed from a `string` data type to a `bytes32` data type. This will reduce the gas costs of accessing the variable since `bytes32` variables are cheaper to store and manipulate. as shown below code snippet.

`contract ProfilePicture is ERC721 {
    byte32 public subprotocolName;
    constructor(address _cidNFT, string memory _subprotocolName) ERC721("Profile Picture", "PFP") {
        cidNFT = ICidNFT(_cidNFT);
        subprotocolName = _subprotocolName;
        _registerInterface(bytes4(keccak256('tokenURI(uint256)')));    }
    function tokenURI(uint256 _id) public view override returns (string memory) {
    }}`