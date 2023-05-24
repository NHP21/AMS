# AMS
The solo-developed Asset Management System utilizes Ethereum blockchain and smart contracts for secure property management, estate planning, and inheritance. It exemplifies determination, innovation, and efficiency in overcoming challenges, delivering a transparent and robust solution for property-related tasks.
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.0;

contract PropertyManagement {
    struct Property {
        string name;
        string kind;
        uint256 price;
        uint256 value;
        string location;
        string description;
        uint256 expiry;
        uint256 lastRenewal;
        address owner;
        address beneficiary;
        address[] willOwners;
    }

    uint256 private _propertyIds;
    mapping(uint256 => Property) private properties;

    event PropertyAdded(uint256 indexed id, address indexed owner);
    event PropertyTransferred(uint256 indexed id, address indexed from, address indexed to);
    event PropertyExpired(uint256 indexed id);
    event PropertyDeleted(uint256 indexed id);
    event WillOwnerAdded(uint256 indexed id, address indexed willOwner);
    event WillOwnerRemoved(uint256 indexed id, address indexed willOwner);
    event OwnershipTransferredByWill(uint256 indexed id, address indexed from, address indexed to);
    event PropertyRenewed(uint256 indexed id);

    modifier onlyOwner(uint256 _id) {
        require(properties[_id].owner == msg.sender, "You are not the owner of this property.");
        _;
    }
    
    modifier onlyWillOwner(uint256 _id) {
        require(isWillOwner(_id, msg.sender), "You are not a will owner of this property.");
        _;
    }

    function addProperty(string memory _name, string memory _kind, uint256 _price, uint256 _value, string memory _location, string memory _description, uint256 _expiry, address _beneficiary) public returns (uint256) {
        _propertyIds++;
        uint256 id = _propertyIds;

        bytes32 hash = sha256(abi.encodePacked(id, block.timestamp));
        properties[id] = Property(_name, _kind, _price, _value, _location, _description, _expiry, block.timestamp, msg.sender, _beneficiary, new address[](0));

        emit PropertyAdded(id, msg.sender);

        return id;
    }

    function transferProperty(uint256 _id, address _to) public onlyOwner(_id) {
        properties[_id].owner = _to;

        emit PropertyTransferred(_id, msg.sender, _to);
    }

    function expireProperty(uint256 _id) public onlyOwner(_id) {
        require(properties[_id].expiry < block.timestamp, "This property has not yet expired.");

        delete properties[_id];

        emit PropertyExpired(_id);
    }

    function getProperty(uint256 _id) public view returns (string memory, string memory, uint256, uint256, string memory, string memory, uint256, address, address) {
        Property storage prop = properties[_id];

        return (prop.name, prop.kind, prop.price, prop.value, prop.location, prop.description, prop.expiry, prop.owner, prop.beneficiary);
    }

    function deleteProperty(uint256 _id) public onlyOwner(_id) {
        delete properties[_id];

        emit PropertyDeleted(_id);
    }

    function addWillOwnership(uint256 _id, address _willOwner) public onlyOwner(_id) {
        properties[_id].willOwners.push(_willOwner);
        emit WillOwnerAdded(_id, _willOwner);
    }

    function removeWillOwnership(uint256 _id, address _willOwner) public onlyOwner(_id) {
        for (uint i = 0; i < properties[_id].willOwners.length; i++) {
            if (properties[_id].willOwners[i] == _willOwner) {
                            properties[_id].willOwners[i] = properties[_id].willOwners[properties[_id].willOwners.length - 1];
            properties[_id].willOwners.pop();
            emit WillOwnerRemoved(_id, _willOwner);
            break;
        }
    }
}

function transferOwnershipToBeneficiary(uint256 _id) public onlyOwner(_id) {
    properties[_id].owner = properties[_id].beneficiary;
    emit PropertyTransferred(_id, msg.sender, properties[_id].beneficiary);
}

function isPropertyExpired(uint256 _id) public view returns (bool) {
    return properties[_id].expiry < block.timestamp;
}

function isPropertyRenewable(uint256 _id) public view returns (bool) {
    return properties[_id].expiry > block.timestamp;
}

function updatePropertyExpiry(uint256 _id, uint256 _expiry) public onlyOwner(_id) {
    require(_expiry > block.timestamp, "Expiry must be in the future");
    properties[_id].expiry = _expiry;
}

function updateLastRenewal(uint256 _id, uint256 _lastRenewal) public onlyOwner(_id) {
    properties[_id].lastRenewal = _lastRenewal;
}

function updateBeneficiaryAndExpiry(uint256 _id, address _newBeneficiary, uint256 _newExpiry) public onlyOwner(_id) {
    require(_newExpiry > block.timestamp, "Expiry must be in the future");
    properties[_id].beneficiary = _newBeneficiary;
    properties[_id].expiry = _newExpiry;
}

function updatePriceAndValue(uint256 _id, uint256 _newPrice, uint256 _newValue) public onlyOwner(_id) {
    properties[_id].price = _newPrice;
    properties[_id].value = _newValue;
}

function updateLocation(uint256 _id, string memory _newLocation) public onlyOwner(_id) {
    properties[_id].location = _newLocation;
}

function updateDescription(uint256 _id, string memory _newDescription) public onlyOwner(_id) {
    properties[_id].description = _newDescription;
}

function updateKind(uint256 _id, string memory _newKind) public onlyOwner(_id) {
    properties[_id].kind = _newKind;
}

function updateName(uint256 _id, string memory _newName) public onlyOwner(_id) {
    properties[_id].name = _newName;
}

function autoTransfer(uint256 _id) public {
    require(properties[_id].lastRenewal + 365 days < block.timestamp, "Property not eligible for automatic transfer");
    properties[_id].owner = properties[_id].beneficiary;
    emit PropertyTransferred(_id, msg.sender, properties[_id].owner);
}

function getWillOwners(uint256 _id) public view returns (address[] memory) {
    return properties[_id].willOwners;
}

function isWillOwner(uint256 _id, address _address) public view returns (bool) {
    for (uint i = 0; i < properties[_id].willOwners.length; i++) {
        if (properties[_id].willOwners[i] == _address) {
            return true;
        }
    }
    return false;
}

function getPropertyValue(uint256 _id) public view returns (uint256) {
    return properties[_id].value;
}
}
