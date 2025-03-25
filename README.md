# Smart-Contract-Specifications
This project implements a **Cyber-Physical System (CPS)** booking platform using Solidity smart contracts deployed under the **Diamond Proxy Pattern**. It leverages **ERC-20 tokens** for payments and includes role-based access control using OpenZeppelin libraries.

---

## 🧩 Diamond Structure

Once the diamond proxy is deployed, the owner of the account that deployed it has the possibility of adding other accounts called owners. Owners acquire the main capability of list or register a CPS, which can then be modified, deleted, or transferred to another owner.
For their part, users can reserve CPSs that owners have already listed, as well as to cancel existing reservations. These users can also access the CPSs they have previously reserved and for which their reservation is still valid. The implementation of the dedicated payable token through ERC-20, called $CPS, as this smart contract is to be deployed outside the diamond proxy.

The system is divided into multiple **facets**, each handling specific responsibilities:

- `CPSAccessControlFacet`: Handles admin and owner roles.
- `CPSFacet`: Manages the CPS entities.
- `CPSBookFacet`: Manages reservations/bookings.

---

## 👤 Roles & Actors

- **Contract Owner** – Holds `DEFAULT_ADMIN_ROLE`. Can add/remove CPS owners.
- **Owner** – Holds `OWNER_ROLE`. Can register, update, delete, and transfer CPSs.
- **User** – Can book, cancel, or request refunds for CPS reservations.

---

## 📦 Data Models

### Owner
| Field       | Type    | Description                    |
|------------|---------|--------------------------------|
| account     | address | Wallet address                 |
| base.name   | string  | Owner name                     |
| base.email  | string  | Email address                  |
| base.country| string  | Country                        |

### CPS (Cyber Physical System)
| Field       | Type    | Description                    |
|------------|---------|--------------------------------|
| CPSId       | uint    | Unique ID                      |
| owner       | address | Owner address                  |
| base.uri    | string  | Metadata URI                   |
| base.price  | uint96  | Price in $CPS tokens           |

### Rental (Booking)
| Field       | Type     | Description                  |
|------------|----------|------------------------------|
| CPSId       | uint     | Associated CPS ID            |
| renter      | address  | Renter address               |
| price       | uint96   | Rental price                 |
| start       | uint32   | Start time (Unix)            |
| end         | uint32   | End time (Unix)              |

---

## ⚙️ Functional Requirements

Below, each implemented function is listed
### CPSAccessControlFacet:

- **initialize**: Sets up the initial contract state and assigns admin roles
- **addOwner**: Grants OWNER_ROLE to a specified account and mints tokens.
- **removeOwner**: Removes the caller’s OWNER_ROLE if conditions are met.
- **updateOwner**: Updates the caller's owner information (name, email, country).
- **isCPSOwner**: Checks if a given account holds the OWNER_ROLE.
- **getCPSOwners**: Retrieves a list of all CPS owners.

## CPSFacet:

- **addCPS**: Allows the contract owner to add a new CPS with a specified URI and price.
- **setTokenURI**: Allows the contract owner to set or update the URI for a specific token ID.
- **updateCPS**: Allows the contract owner to update the URI and price of an existing CPS.
- **deleteCPS**: Allows the contract owner to delete an existing CPS.
- **transferCPS**: Transfers ownership of a CPS to another owner.
- **getCPS**: Retrieve the information about the CPS structure corresponding to the provided CPS ID.
- **getAllCPSs**: Retrieve a list with the information about all the CPS listed.
- **tokenURI**: Retrieves the URI associated with a specific token ID.

## CPSBookFacet:

- **requestBook**: Allows a user to request a booking for a CPS.
- **cancelBookRequest**: Allows a user to cancel a previously requested booking.
- **bookConfirmed**: Confirms a booking request and marks it as approved.
- **bookDenied**: Denies a booking request and removes it from the system.
- **cancelBookCPS**: Allows a user or the contract owner to cancel an existing confirmed booking.
- **requestFunds**: Allows a user to request a refund for a canceled, invalid or used booking.
- **getBookings**: Retrieves all bookings related to a specific CPS ID.
- **getAllBookings**: Retrieves all booking records stored in the contract.
- **getBooking**: Retrieves booking details for a specific CPS ID and start date.
- **getCPSBalance**: Retrieves the CPS token balance of a specific account.
- **getSafeBalance**: Retrieves the total balance of funds held in the contract.

Use case Specification
**Detailed information on how each specific use case is executed is provided below**:

**CPSControlFacet**:

|                | Description |
|----------------|-------------|
| Use case       | **INITIALIZE** |
| Definition     | `function initialize(string memory _name, string memory _email, string memory _country, address _cPSERC20) public initializer` |
| Actors         | Contract owner |
| Purpose        | Initializes the smart contract setting, up the initial admin role and the ERC20 token external contract. |
| Summary        | Only the contract owner can initialize the contract |
| Preconditions  | Have a WALLET and sufficient funds. Can only be executed once. |
| Postconditions | The contract becomes initialized |
| Events         | Emits a `{RoleGranted}` event if the role is successfully granted. |

|                | Description |
|----------------|-------------|
| Use case       | **ADD OWNER** |
| Definition     | `function addOwner(string memory _name, address _account, string memory _email, string memory _country) external defaultAdminRole returns (bool success)` |
| Actors         | Contract owner |
| Purpose        | Adds a new owner by granting the OWNER_ROLE and minting tokens for the specified account. |
| Summary        | Only the contract owner can add a new owner. |
| Preconditions  | The account must not already have the OWNER_ROLE. |
| Postconditions | The account receives the OWNER_ROLE and 1000 CPS ERC20 tokens are minted for them. |
| Events         | Emits an {OwnerAdded} event if the owner is successfully added. |

|                | Description |
|----------------|-------------|
| Use case       | **REMOVE SPECIFIC OWNER** |
| Definition     | `function removeOwner(address _owner) external defaultAdminRole returns (bool success)` |
| Actors         | Contract owner, owners |
| Purpose        | Removes a specified owner from the owner list if they do not have any CPSs. |
| Summary        | Only the contract owner can remove an owner from the list. |
| Preconditions  | The owner must not have any CPSUsers or CPSs. |
| Postconditions | The specified owner's role is revoked if conditions are met. |
| Events         | Emits an {OwnerRemoved} event if the owner is successfully removed. |

|                | Description |
|----------------|-------------|
| Use case       | **UPDATE OWNER** |
| Definition     | `function updateOwner(string memory _name, string memory _email, string memory _country) external onlyRole(OWNER_ROLE) returns (bool success)` |
| Actors         | Contract owner |
| Purpose        | Updates the owner information for the caller, modifying their name, email, and country details. |
| Summary        | Only the owner can update their own information. |
| Preconditions  | The caller must be an existing owner. |
| Postconditions | The owner's information is updated with the new details provided. |
| Events         | No explicit event mentioned in the function description. |

**The functions listed below are queries that do not modify the state of the variables**:

| Function Name | Definition | Purpose | Return Type |
|---------------|------------|---------|-------------|
| isCPSOwner    | `function isCPSOwner(address _account) external view returns (bool)` | Checks if the given account is a CPS owner. | bool |
| getCPSOwners  | `function getCPSOwners() external view returns (Owner[] memory)` | Retrieves the list of all CPS owners. | Owner array |


**CPSFacet**:

|                | Description |
|----------------|-------------|
| Use case       | **ADD CPS** |
| Definition     | `function addCPS(string memory _uri, uint96 _price) external isCPSOwner returns (bool success)` |
| Actors         | Owners |
| Purpose        | Allows the contract owner to add a new CPS with a specified URI and price |
| Summary        | Only the contract owner can add a new CPS |
| Preconditions  | Caller must be the contract owner |
| Postconditions | A new CPS is registered with the given URI and price |
| Events         | Emits a {CPSAdded} event if successful |

|                | Description |
|----------------|-------------|
| Use case       | **SET TOKEN URI** |
| Definition     | `function setTokenURI(uint256 _tokenId, string memory _tokenURI) external` |
| Actors         | Owners |
| Purpose        | Allows the CPS owner can set or update the URI for a specific token ID |
| Summary        | Only the CPS owner can modify the token URI |
| Preconditions  | Caller must be the CPS owner; the token ID must exist |
| Postconditions | The specified token's URI is updated |
| Events         | Emits a {URIUpdated} event if successful |

|                | Description |
|----------------|-------------|
| Use case       | **UPDATE CPS** |
| Definition     | `function updateCPS(uint _CPSId, string memory _newURI, uint96 _newPrice) external returns (bool success)` |
| Actors         | Owner |
| Purpose        | Allows the CPS owner to update the URI and price of an existing CPS |
| Summary        | Only the CPS owner can modify CPS details |
| Preconditions  | Caller must be the contract owner; CPS ID must exist |
| Postconditions | The specified CPS's URI and price are updated |
| Events         | Emits a {CPSUpdated} event if successful |

|                | Description |
|----------------|-------------|
| Use case       | **DELETE CPS** |
| Definition     | `function deleteCPS(uint _CPSId) external isOwner(_CPSId) returns (bool success)` |
| Actors         | Owners |
| Purpose        | Allows the CPS owner to delete an existing CPS |
| Summary        | Only the CPS owner can remove a CPS |
| Preconditions  | Caller must be the contract owner; CPS ID must exist and be removable |
| Postconditions | The specified CPS is removed from the system |
| Events         | Emits a {CPSDeleted} event if successful |

|                | Description |
|----------------|-------------|
| Use case       | **TRANSFER CPS** |
| Definition     | `function transferCPS(uint _CPSId, address _newOwner) external isOwner(_CPSId) returns (bool success)` |
| Actors         | Owner |
| Purpose        | Allows the CPS owner to transfer ownership of a CPS to another owner |
| Summary        | Only the contract owner can transfer a CPS to another owner |
| Preconditions  | Caller must be the contract owner; CPS ID must exist; new owner must be valid |
| Postconditions | Ownership of the specified CPS is transferred to the new owner |
| Events         | Emits a {CPSTransferred} event if successful |

**The functions listed below are queries that do not modify the state of the variables**:

| Function Name | Definition | Purpose | Return Type |
|---------------|------------|---------|-------------|
| getCPS        | `function getCPS(uint _CPSId) public view returns (CPS memory)` | Retrieves the CPS associated with the given CPS ID. | CPS |
| getAllCPSs    | `function getAllCPSs() public view returns (CPS[] memory)` | Retrieves the list of all CPS. | CPS array |
| tokenURI      | `function tokenURI(uint256 _tokenId) public view returns (string memory)` | Retrieves the URI associated with a specific token ID. | URI string |

**CPSBookFacet**:

|           | Description |
|----------------|-------------|
| Use case       | **REQUEST BOOK** |
| Definition     | `function requestBook(uint256 _CPSId, uint32 _start, uint32 _end) external returns (bool success)` |
| Actors         | User |
| Purpose        | Allows a user to request a booking for a CPS |
| Summary        | Users can request a booking for a listed CPS |
| Preconditions  | The CPS must be available and user must have enough funds |
| Postconditions | A booking request is created and stored |
| Events         | Emits a {BookRequested} event if successful |

|           | Description |
|----------------|-------------|
| Use case       | **CANCEL BOOK REQUEST** |
| Definition     | `function cancelBookRequest(uint256 _bookRequestId) public returns (bool success)` |
| Actors         | User |
| Purpose        | Allows a user to cancel a previously requested booking |
| Summary        | Users can cancel their pending booking requests |
| Preconditions  | Caller must be the requester of the booking; The booking request must exist |
| Postconditions | The booking request is removed from the system |
| Events         | Emits a {BookRequestCancelled} event if successful |

|           | Description |
|----------------|-------------|
| Use case       | **BOOK CONFIRMED** |
| Definition     | `function bookConfirmed(uint256 _bookRequestId) public returns (bool success)` |
| Actors         | Owners |
| Purpose        | Confirms a booking request and marks it as approved |
| Summary        | The CPS owner can approve a valid booking request |
| Preconditions  | Caller must be the CPS owner; The booking request must exist and be valid |
| Postconditions | The booking request is confirmed and marked as approved |
| Events         | Emits a {BookConfirmed} event if successful |

|           | Description |
|----------------|-------------|
| Use case       | **BOOK DENIED** |
| Definition     | `function bookDenied(uint256 _bookRequestId) public returns (bool success)` |
| Actors         | Owners |
| Purpose        | Denies a booking request and removes it from the system |
| Summary        | The CPS owner can reject a booking request |
| Preconditions  | Caller must be the CPS owner; The booking request must exist |
| Postconditions | The booking request is denied and removed from the system |
| Events         | Emits a {BookDenied} event if successful |

|           | Description |
|----------------|-------------|
| Use case       | **CANCEL BOOK CPS** |
| Definition     | `function cancelBookCPS(uint256 _bookId) public returns (bool success)` |
| Actors         | Users, owners |
| Purpose        | Allows a user or the CPS owner to cancel an existing confirmed booking |
| Summary        | Users or the CPS owner can cancel a confirmed booking |
| Preconditions  | Caller must be the owner of the booking or the contract owner; The booking must exist |
| Postconditions | The booking is canceled and removed from the system. The funds are returned. |
| Events         | Emits a {BookCancelled} event if successful |

|           | Description |
|----------------|-------------|
| Use case       | **REQUEST FUNDS** |
| Definition     | `function requestFunds(uint256 _bookId) public returns (bool success)` |
| Actors         | Owners |
| Purpose        | Allows a CPS owner to reclaim funds from consumed or expired bookings |
| Summary        | Owners can request the funds when a booking is outdated |
| Preconditions  | Caller must be the CPS owner of the bookings pending to retrieve |
| Postconditions | A refund request is initiated and processed |
| Events         | Emits a {FundsRequested} event if successful |

**The functions listed below are queries that do not modify the state of the variables**:

| Function Name   | Definition                                                                 | Purpose                                                   | Return Type   |
|----------------|----------------------------------------------------------------------------|-----------------------------------------------------------|---------------|
| getBookings     | `function getBookings(uint CPSId) external view returns (Rental[] memory)` | Retrieves all bookings related to a specific CPS ID       | Rental array  |
| getAllBookings  | `function getAllBookings() public view returns (Rental[] memory)`          | Retrieves all booking records stored in the contract      | Rental array  |
| getBooking      | `function getBooking(uint _CPSId, uint32 _startDate) public view returns (Rental memory)` | Retrieves booking details for a specific CPS ID and start date | Rental        |
| getCPSBalance   | `function getCPSBalance(address account) public view returns (uint256)`    | Retrieves the CPS token balance of a specific account     | uint256       |
| getSafeBalance  | `function getSafeBalance() public view returns (uint256)`                  | Retrieves the total balance of funds held in the contract | uint256       |

## 💳 Token Integration

The project uses a dedicated **ERC-20 token: `$CPS`** for all transactions. The token is deployed outside the diamond proxy.
