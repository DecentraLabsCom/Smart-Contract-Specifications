# Smart-Contract-Specifications
This repository details the specification of a **lab** booking, access and sharing decentralized solution using Solidity smart contracts that includes role-based access control using OpenZeppelin libraries. 

---

## 🧩 Diamond Structure

The proposed architecture leverages a **diamond proxy (EIP-2535)**, behind which various smart contracts are deployed. This approach is chosen for two key reasons:
1.	To enable seamless updates and enhancements to the contracts without disrupting client functionality.
2.	To maintain a degree of control during the initial phases of development.

Once the diamond proxy is deployed, the account responsible for the deployment gains the ability to add other accounts, referred to as **“providers”**. Providers are empowered to list or register online laboratories, which can later be modified, deleted, or transferred to a different provider.

Users, on the other hand, are able to reserve laboratories listed by the providers, as well as cancel any of their existing reservations. Additionally, users can access laboratories they have previously reserved, as long as the reservation remains valid.

The implementation of a dedicated payable token, **$LAB**, through **ERC-20**, is not covered within the scope of this document. This token will be deployed separately from the diamond proxy.

The system is divided into multiple **facets**, each handling specific responsibilities:

- `ProviderFacet`: Handles admin and provider roles.
- `LabFacet`: Manages the lab entities.
- `ReservationFacet`: Manages reservations/bookings.

---

## 👤 Roles & Actors

- **Contract Owner** – Holds `DEFAULT_ADMIN_ROLE`. Can add/remove lab providers.
- **Provider** – Holds `PROVIDER_ROLE`. Can register, update, delete, and transfer labs.
- **User** – Can book, cancel, or request refunds for lab reservations.

---

## 📦 Data Models

### Provider
| Field       | Type    | Description                    |
|-------------|---------|--------------------------------|
| account     | address | Wallet address                 |
| base.name   | string  | Provider name                  |
| base.email  | string  | Email address                  |
| base.country| string  | Country                        |

### Lab (Cyber Physical System)
| Field       | Type    | Description                    |
|-------------|---------|--------------------------------|
| labId       | uint    | Unique ID                      |
| provider    | address | Provider address               |
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
## ProviderFacet:

- **initialize**: Sets up the initial contract state and assigns admin roles
- **addProvider**: Grants PROVIDER_ROLE to a specified account and mints $LAB tokens.
- **removeProvider**: Removes the caller’s PROVIDER_ROLE if conditions are met.
- **updateProvider**: Updates the caller's provider information (name, email, country).
- **isLabProvider**: Checks if a given account holds the PROVIDER_ROLE.
- **getLabProviders**: Retrieves a list of all lab providers.

## LabFacet:

- **addLab**: Allows the contract provider to add a new lab with a specified URI and price.
- **setLabURI**: Allows the contract provider to set or update the URI for a specific token ID.
- **updateLab**: Allows the contract provider to update the URI and price of an existing lab.
- **deleteLab**: Allows the contract provider to delete an existing lab.
- **transferLab**: Transfers ownership of a lab to another wallet.
- **getLab**: Retrieves the information about the lab structure corresponding to the provided lab ID.
- **getAllLabs**: Retrieves a list with the information about all the labs.
- **labURI**: Retrieves the URI associated with a specific lab ID.

## ReservationFacet:

- **requestBook**: Allows a user to request a booking for a lab.
- **cancelBookRequest**: Allows a user to cancel a previously requested booking.
- **bookConfirmed**: Confirms a booking request and marks it as approved.
- **bookDenied**: Denies a booking request and removes it from the system.
- **cancelBookLab**: Allows a user or the lab provider to cancel an existing confirmed booking.
- **requestFunds**: Allows a user to request a refund for a canceled, invalid or used booking.
- **getBookings**: Retrieves all bookings related to a specific lab ID.
- **getAllBookings**: Retrieves all booking records stored in the contract.
- **getBooking**: Retrieves booking details for a specific lab ID and start date.
- **getLabBalance**: Retrieves the lab's $LAB token balance of a specific account.
- **getSafeBalance**: Retrieves the total balance of $LAB funds held in the contract.

Use case Specification
**Detailed information on how each specific use case is executed is provided below**:

**ProviderFacet**:

|                | Description |
|----------------|-------------|
| Use case       | **INITIALIZE** |
| Definition     | `function initialize(string memory _name, string memory _email, string memory _country, address _labERC20) public initializer` |
| Actors         | Contract owner |
| Purpose        | Initializes the smart contract setting, the initial admin role and the ERC20 token external contract. |
| Summary        | Only the contract owner can initialize the contract |
| Preconditions  | Have a WALLET and sufficient funds. Can only be executed once. |
| Postconditions | The contract becomes initialized |
| Events         | Emits a `{RoleGranted}` event if the role is successfully granted. |

|                | Description |
|----------------|-------------|
| Use case       | **ADD PROVIDER** |
| Definition     | `function addProvider(string memory _name, address _account, string memory _email, string memory _country) external defaultAdminRole returns (bool success)` |
| Actors         | Contract owner |
| Purpose        | Adds a new provider by granting the PROVIDER_ROLE and minting tokens for the specified account. |
| Summary        | Only the contract owner can add a new provider. |
| Preconditions  | The account must not already have the PROVIDER_ROLE. |
| Postconditions | The account receives the PROVIDER_ROLE and 1000 $LAB ERC20 tokens are minted for them. |
| Events         | Emits an {ProviderAdded} event if the provider is successfully added. |

|                | Description |
|----------------|-------------|
| Use case       | **REMOVE SPECIFIC PROVIDER** |
| Definition     | `function removeProvider(address _provider) external defaultAdminRole returns (bool success)` |
| Actors         | Contract owner |
| Purpose        | Removes a specified provider from the provider list if they do not have any lab. |
| Summary        | Only the contract owner can remove a provider from the list. |
| Preconditions  | The provider must not own any lab. |
| Postconditions | The specified provider's role is revoked if conditions are met. |
| Events         | Emits an {ProviderRemoved} event if the provider is successfully removed. |

|                | Description |
|----------------|-------------|
| Use case       | **UPDATE PROVIDER** |
| Definition     | `function updateProvider(string memory _name, string memory _email, string memory _country) external onlyRole(PROVIDER_ROLE) returns (bool success)` |
| Actors         | Provider |
| Purpose        | Updates the provider information for the caller, modifying their name, email, and country details. |
| Summary        | Only a provider can update their own information. |
| Preconditions  | The caller must be an existing provider. |
| Postconditions | The provider's information is updated with the new details provided. |
| Events         | No explicit event mentioned in the function description. |

**The functions listed below are queries that do not modify the state of the variables**:

| Function Name | Definition | Purpose | Return Type |
|---------------|------------|---------|-------------|
| isLabProvider    | `function isLabProvider(address _account) external view returns (bool)` | Checks if the given account is a lab provider. | bool |
| getLabProviders  | `function getLabProviders() external view returns (Provider[] memory)` | Retrieves the list of all lab providers. | Provider array |


**LabFacet**:

|                | Description |
|----------------|-------------|
| Use case       | **ADD LAB** |
| Definition     | `function addLab(string memory _uri, uint96 _price) external isLabProvider returns (bool success)` |
| Actors         | Providers |
| Purpose        | Allows the provider to add a new lab with a specified URI and price |
| Summary        | Only a provider can add a new lab |
| Preconditions  | Caller must be the lab provider |
| Postconditions | A new lab is registered with the given URI and price |
| Events         | Emits a {LabAdded} event if successful |

|                | Description |
|----------------|-------------|
| Use case       | **SET LAB URI** |
| Definition     | `function setTokenURI(uint256 _labId, string memory _labURI) external` |
| Actors         | Providers |
| Purpose        | Allows the lab provider to set or update the URI for a specific lab ID |
| Summary        | Only the lab provider can modify the lab URI |
| Preconditions  | Caller must be the lab provider; the lab ID must exist |
| Postconditions | The specified lab's URI is updated |
| Events         | Emits a {URIUpdated} event if successful |

|                | Description |
|----------------|-------------|
| Use case       | **UPDATE LAB** |
| Definition     | `function updateLab(uint _labId, string memory _newURI, uint96 _newPrice) external returns (bool success)` |
| Actors         | Provider |
| Purpose        | Allows the lab provider to update the URI and price of an existing lab |
| Summary        | Only the lab provider can modify lab details |
| Preconditions  | Caller must be the lab provider; lab ID must exist |
| Postconditions | The specified lab's URI and price are updated |
| Events         | Emits a {LabUpdated} event if successful |

|                | Description |
|----------------|-------------|
| Use case       | **DELETE LAB** |
| Definition     | `function deleteLab(uint _labId) external isProvider(_labId) returns (bool success)` |
| Actors         | Providers |
| Purpose        | Allows the lab provider to delete an existing lab |
| Summary        | Only the lab provider can remove a lab |
| Preconditions  | Caller must be the lab provider; lab ID must exist and be removable |
| Postconditions | The specified lab is removed from the system |
| Events         | Emits a {LabDeleted} event if successful |

|                | Description |
|----------------|-------------|
| Use case       | **TRANSFER LAB** |
| Definition     | `function transferLab(uint _labId, address _newProvider) external isProvider(_labId) returns (bool success)` |
| Actors         | Provider |
| Purpose        | Allows the lab provider to transfer ownership of a lab to another wallet address |
| Summary        | Only the lab provider can transfer a lab to another wallet address |
| Preconditions  | Caller must be the lab provider; lab ID must exist; new provider must be valid |
| Postconditions | Ownership of the specified lab is transferred to the new provider |
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
| Actors         | Providers |
| Purpose        | Confirms a booking request and marks it as approved |
| Summary        | The lab provider can approve a valid booking request |
| Preconditions  | Caller must be the lab provider; The booking request must exist and be valid |
| Postconditions | The booking request is confirmed and marked as approved |
| Events         | Emits a {BookConfirmed} event if successful |

|                | Description |
|----------------|-------------|
| Use case       | **BOOK DENIED** |
| Definition     | `function bookDenied(uint256 _bookRequestId) public returns (bool success)` |
| Actors         | Providers |
| Purpose        | Denies a booking request and removes it from the system |
| Summary        | The lab provider can reject a booking request |
| Preconditions  | Caller must be the lab provider; The booking request must exist |
| Postconditions | The booking request is denied and removed from the system |
| Events         | Emits a {BookDenied} event if successful |

|                | Description |
|----------------|-------------|
| Use case       | **CANCEL BOOK LAB** |
| Definition     | `function cancelBookLab(uint256 _bookId) public returns (bool success)` |
| Actors         | Users, providers |
| Purpose        | Allows a user or the lab provider to cancel an existing confirmed booking |
| Summary        | Users or the lab provider can cancel a confirmed booking |
| Preconditions  | Caller must be the owner of the booking or the lab provider; The booking must exist |
| Postconditions | The booking is canceled and removed from the system. The funds are returned. |
| Events         | Emits a {BookCancelled} event if successful |

|                | Description |
|----------------|-------------|
| Use case       | **REQUEST FUNDS** |
| Definition     | `function requestFunds(uint256 _bookId) public returns (bool success)` |
| Actors         | Providers |
| Purpose        | Allows a lab provider to reclaim funds from consumed or expired bookings |
| Summary        | Providers can request the funds when a booking is outdated |
| Preconditions  | Caller must be the provider of the lab with bookings pending to retrieve |
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
