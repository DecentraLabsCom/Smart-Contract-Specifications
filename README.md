# Smart-Contract-Specifications
This project includes the specification of a **lab** booking, access and sharing decentralized solution using Solidity smart contracts that includes role-based access control using OpenZeppelin libraries. 

---

## 🧩 Diamond Structure

The proposed architecture leverages a **diamond proxy (EIP-2535)**, behind which various smart contracts are deployed. This approach is chosen for two key reasons:
1.	To enable seamless updates and enhancements to the contracts without disrupting client functionality.
2.	To maintain a degree of control during the initial phases of development.

Once the diamond proxy is deployed, the account responsible for the deployment gains the ability to add other accounts, referred to as **“owners”**. Owners are empowered to list or register online laboratories, which can later be modified, deleted, or transferred to a different owner.

Users, on the other hand, are able to reserve laboratories listed by the owners, as well as cancel any existing reservations. Additionally, users can access laboratories they have previously reserved, as long as the reservation remains valid.

The implementation of a dedicated payable token, **$LAB**, through **ERC-20**, is not covered within the scope of this document. This token will be deployed separately from the diamond proxy.

The system is divided into multiple **facets**, each handling specific responsibilities:

- `OwnerFacet`: Handles admin and owner roles.
- `LabFacet`: Manages the lab entities.
- `ReservationFacet`: Manages reservations/bookings.

---

## 👤 Roles & Actors

- **Contract Owner** – Holds `DEFAULT_ADMIN_ROLE`. Can add/remove lab owners.
- **Owner** – Holds `OWNER_ROLE`. Can register, update, delete, and transfer labs.
- **User** – Can book, cancel, or request refunds for lab reservations.

---

## 📦 Data Models

### Owner
| Field       | Type    | Description                    |
|-------------|---------|--------------------------------|
| account     | address | Wallet address                 |
| base.name   | string  | Owner name                     |
| base.email  | string  | Email address                  |
| base.country| string  | Country                        |

### Lab (Cyber Physical System)
| Field       | Type    | Description                    |
|-------------|---------|--------------------------------|
| labId       | uint    | Unique ID                      |
| owner       | address | Owner address                  |
| base.uri    | string  | Metadata URI                   |
| base.price  | uint96  | Price in $LAB tokens           |

### Reservation (Booking)
| Field       | Type     | Description                  |
|-------------|----------|------------------------------|
| labId       | uint     | Associated lab ID            |
| renter      | address  | Renter address               |
| price       | uint96   | Rental price                 |
| start       | uint32   | Start time (Unix)            |
| end         | uint32   | End time (Unix)              |

---

## ⚙️ Functional Requirements

Below, each implemented function is listed
### OwnerFacet:

- **initialize**: Sets up the initial contract state and assigns admin roles
- **addOwner**: Grants OWNER_ROLE to a specified account and mints tokens.
- **removeOwner**: Removes the caller’s OWNER_ROLE if conditions are met.
- **updateOwner**: Updates the caller's owner information (name, email, country).
- **isLabOwner**: Checks if a given account holds the OWNER_ROLE.
- **getLabOwners**: Retrieves a list of all lab owners.

## LabFacet:

- **addLab**: Allows the contract owner to add a new lab with a specified URI and price.
- **setLabURI**: Allows the contract owner to set or update the URI for a specific token ID.
- **updateLab**: Allows the contract owner to update the URI and price of an existing lab.
- **deleteLab**: Allows the contract owner to delete an existing lab.
- **transferLab**: Transfers ownership of a lab to another owner.
- **getLab**: Retrieves the information about the lab structure corresponding to the provided lab ID.
- **getAllLabs**: Retrieves a list with the information about all the labs.
- **labURI**: Retrieves the URI associated with a specific lab ID.

## ReservationFacet:

- **requestBook**: Allows a user to request a booking for a lab.
- **cancelBookRequest**: Allows a user to cancel a previously requested booking.
- **bookConfirmed**: Confirms a booking request and marks it as approved.
- **bookDenied**: Denies a booking request and removes it from the system.
- **cancelBookLab**: Allows a user or the contract owner to cancel an existing confirmed booking.
- **requestFunds**: Allows a user to request a refund for a canceled, invalid or used booking.
- **getBookings**: Retrieves all bookings related to a specific lab ID.
- **getAllBookings**: Retrieves all booking records stored in the contract.
- **getBooking**: Retrieves booking details for a specific lab ID and start date.
- **getLabBalance**: Retrieves the lab's $LAB token balance of a specific account.
- **getSafeBalance**: Retrieves the total balance of $LAB funds held in the contract.

Use case Specification
**Detailed information on how each specific use case is executed is provided below**:

**OwnerFacet**:

|                | Description |
|----------------|-------------|
| Use case       | **INITIALIZE** |
| Definition     | `function initialize(string memory _name, string memory _email, string memory _country, address _labERC20) public initializer` |
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
| Postconditions | The account receives the OWNER_ROLE and 1000 $LAB ERC20 tokens are minted for them. |
| Events         | Emits an {OwnerAdded} event if the owner is successfully added. |

|                | Description |
|----------------|-------------|
| Use case       | **REMOVE SPECIFIC OWNER** |
| Definition     | `function removeOwner(address _owner) external defaultAdminRole returns (bool success)` |
| Actors         | Contract owner, owners |
| Purpose        | Removes a specified owner from the owner list if they do not have any lab. |
| Summary        | Only the contract owner can remove an owner from the list. |
| Preconditions  | The owner must not own any lab. |
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
| isLabOwner    | `function isLabOwner(address _account) external view returns (bool)` | Checks if the given account is a lab owner. | bool |
| getLabOwners  | `function getLabOwners() external view returns (Owner[] memory)` | Retrieves the list of all lab owners. | Owner array |


**LabFacet**:

|                | Description |
|----------------|-------------|
| Use case       | **ADD LAB** |
| Definition     | `function addLab(string memory _uri, uint96 _price) external isLabOwner returns (bool success)` |
| Actors         | Owners |
| Purpose        | Allows the contract owner to add a new lab with a specified URI and price |
| Summary        | Only the contract owner can add a new lab |
| Preconditions  | Caller must be the contract owner |
| Postconditions | A new lab is registered with the given URI and price |
| Events         | Emits a {LabAdded} event if successful |

|                | Description |
|----------------|-------------|
| Use case       | **SET LAB URI** |
| Definition     | `function setTokenURI(uint256 _labId, string memory _labURI) external` |
| Actors         | Owners |
| Purpose        | Allows the lab owner can set or update the URI for a specific token ID |
| Summary        | Only the lab owner can modify the token URI |
| Preconditions  | Caller must be the lab owner; the token ID must exist |
| Postconditions | The specified token's URI is updated |
| Events         | Emits a {URIUpdated} event if successful |

|                | Description |
|----------------|-------------|
| Use case       | **UPDATE LAB** |
| Definition     | `function updateLab(uint _labId, string memory _newURI, uint96 _newPrice) external returns (bool success)` |
| Actors         | Owner |
| Purpose        | Allows the lab owner to update the URI and price of an existing lab |
| Summary        | Only the lab owner can modify lab details |
| Preconditions  | Caller must be the contract owner; lab ID must exist |
| Postconditions | The specified lab's URI and price are updated |
| Events         | Emits a {LabUpdated} event if successful |

|                | Description |
|----------------|-------------|
| Use case       | **DELETE LAB** |
| Definition     | `function deleteLab(uint _labId) external isOwner(_labId) returns (bool success)` |
| Actors         | Owners |
| Purpose        | Allows the lab owner to delete an existing lab |
| Summary        | Only the lab owner can remove a lab |
| Preconditions  | Caller must be the contract owner; lab ID must exist and be removable |
| Postconditions | The specified lab is removed from the system |
| Events         | Emits a {LabDeleted} event if successful |

|                | Description |
|----------------|-------------|
| Use case       | **TRANSFER LAB** |
| Definition     | `function transferLab(uint _labId, address _newOwner) external isOwner(_labId) returns (bool success)` |
| Actors         | Owner |
| Purpose        | Allows the lab owner to transfer ownership of a lab to another owner |
| Summary        | Only the lab owner can transfer a lab to another owner |
| Preconditions  | Caller must be the lab owner; lab ID must exist; new owner must be valid |
| Postconditions | Ownership of the specified lab is transferred to the new owner |
| Events         | Emits a {LabTransferred} event if successful |

**The functions listed below are queries that do not modify the state of the variables**:

| Function Name | Definition | Purpose | Return Type |
|---------------|------------|---------|-------------|
| getLab        | `function getLab(uint _labId) public view returns (Lab memory)` | Retrieves the lab associated with the given lab ID. | Lab |
| getAllLabs    | `function getAllLabs() public view returns (Lab[] memory)` | Retrieves the list of all labs. | Lab array |
| labURI        | `function labURI(uint256 _labId) public view returns (string memory)` | Retrieves the URI associated with a specific lab ID. | URI string |

**ReservationFacet**:

|                | Description |
|----------------|-------------|
| Use case       | **REQUEST BOOK** |
| Definition     | `function requestBook(uint256 _labId, uint32 _start, uint32 _end) external returns (bool success)` |
| Actors         | User |
| Purpose        | Allows a user to request a booking for a lab |
| Summary        | Users can request a booking for a listed lab |
| Preconditions  | The lab must be available and user must have enough funds |
| Postconditions | A booking request is created and stored |
| Events         | Emits a {BookRequested} event if successful |

|                | Description |
|----------------|-------------|
| Use case       | **CANCEL BOOK REQUEST** |
| Definition     | `function cancelBookRequest(uint256 _bookRequestId) public returns (bool success)` |
| Actors         | User |
| Purpose        | Allows a user to cancel a previously requested booking |
| Summary        | Users can cancel their pending booking requests |
| Preconditions  | Caller must be the requester of the booking; The booking request must exist |
| Postconditions | The booking request is removed from the system |
| Events         | Emits a {BookRequestCancelled} event if successful |

|                | Description |
|----------------|-------------|
| Use case       | **BOOK CONFIRMED** |
| Definition     | `function bookConfirmed(uint256 _bookRequestId) public returns (bool success)` |
| Actors         | Owners |
| Purpose        | Confirms a booking request and marks it as approved |
| Summary        | The lab owner can approve a valid booking request |
| Preconditions  | Caller must be the lab owner; The booking request must exist and be valid |
| Postconditions | The booking request is confirmed and marked as approved |
| Events         | Emits a {BookConfirmed} event if successful |

|                | Description |
|----------------|-------------|
| Use case       | **BOOK DENIED** |
| Definition     | `function bookDenied(uint256 _bookRequestId) public returns (bool success)` |
| Actors         | Owners |
| Purpose        | Denies a booking request and removes it from the system |
| Summary        | The lab owner can reject a booking request |
| Preconditions  | Caller must be the lab owner; The booking request must exist |
| Postconditions | The booking request is denied and removed from the system |
| Events         | Emits a {BookDenied} event if successful |

|                | Description |
|----------------|-------------|
| Use case       | **CANCEL BOOK LAB** |
| Definition     | `function cancelBookLab(uint256 _bookId) public returns (bool success)` |
| Actors         | Users, owners |
| Purpose        | Allows a user or the lab owner to cancel an existing confirmed booking |
| Summary        | Users or the lab owner can cancel a confirmed booking |
| Preconditions  | Caller must be the owner of the booking or the contract owner; The booking must exist |
| Postconditions | The booking is canceled and removed from the system. The funds are returned. |
| Events         | Emits a {BookCancelled} event if successful |

|                | Description |
|----------------|-------------|
| Use case       | **REQUEST FUNDS** |
| Definition     | `function requestFunds(uint256 _bookId) public returns (bool success)` |
| Actors         | Owners |
| Purpose        | Allows a lab owner to reclaim funds from consumed or expired bookings |
| Summary        | Owners can request the funds when a booking is outdated |
| Preconditions  | Caller must be the lab owner of the bookings pending to retrieve |
| Postconditions | A refund request is initiated and processed |
| Events         | Emits a {FundsRequested} event if successful |

**The functions listed below are queries that do not modify the state of the variables**:

| Function Name   | Definition                                                                 | Purpose                                                   | Return Type   |
|-----------------|----------------------------------------------------------------------------|-----------------------------------------------------------|---------------|
| getBookings     | `function getBookings(uint labId) external view returns (Rental[] memory)` | Retrieves all bookings related to a specific lab ID       | Reservation array  |
| getAllBookings  | `function getAllBookings() public view returns (Rental[] memory)`          | Retrieves all booking records stored in the contract      | Reservation array  |
| getBooking      | `function getBooking(uint _labId, uint32 _startDate) public view returns (Rental memory)` | Retrieves booking details for a specific lab ID and start date | Reservation        |
| getLabBalance   | `function getLabBalance(address account) public view returns (uint256)`    | Retrieves the lab token balance of a specific account     | uint256       |
| getSafeBalance  | `function getSafeBalance() public view returns (uint256)`                  | Retrieves the total balance of funds held in the contract | uint256       |

## 💳 Token Integration

The project uses a dedicated **ERC-20 token: `$LAB`** for all transactions. The token is deployed outside the diamond proxy.
