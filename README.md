# Smart-Contract-Specifications
This repository details the specification of a **lab** booking, access and sharing decentralized solution using Solidity smart contracts that includes role-based access control using OpenZeppelin libraries.

---

## 🧩 Diamond Structure

The proposed architecture leverages a **diamond proxy (EIP-2535)**, behind which various smart contracts are deployed. This approach is chosen for two key reasons:
1.	To enable seamless updates and enhancements to the contracts without disrupting client functionality.
2.	To maintain a degree of control during the initial phases of development.

Once the diamond proxy is deployed, the account responsible for the deployment gains the ability to add other accounts, referred to as **“providers”**. Providers are empowered to list or register online laboratories, which can later be modified, deleted, or transferred to a different provider.

Users, on the other hand, are able to reserve laboratories listed by the providers, as well as cancel any of their existing reservations. Additionally, users can access laboratories they have previously reserved, as long as the reservation remains valid. This functionality is achieved through the implementation of a new proposal: **reservable token**, which allows for the reservation of the use of an ERC721 token over a time interval, improving upon existing standards.

The implementation of a dedicated payable token, **$LAB**, through **ERC-20**, is not covered within the scope of this document. This token will be deployed separately from the diamond proxy.

The system is divided into multiple **facets**, each handling specific responsibilities:

- `ProviderFacet`: Handles admin and provider roles.
- `LabFacet`: Manages the lab entities.
- `ReservationFacet`: Manages reservations/bookings.

---

## 👤 Roles & Actors

- **Contract Owner** – Holds `DEFAULT_ADMIN_ROLE`. Can add/remove lab providers.
- **Provider** – Holds `PROVIDER_ROLE`. Can register, update, delete, transfer and list labs for make them reservable.
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

### Lab (Cyber Physical System implemented as an NFT)
| Field       | Type    | Description                    |
|-------------|---------|--------------------------------|
| labId       | uint    | Unique ID                      |
| base.uri    | string  | Off-chain metadata URI         |
| base.price  | uint96  | Price in $LAB tokens           |
| base.auth   | string  | Authentication service URI     |
| base.accessURI   | string  | Lab services acess URI|
| base.accessKey   | string  | Key or ID used for routing/access|

### Reservation (Booking presented as a new reservable token)
| Field       | Type     | Description                  |
|-------------|----------|------------------------------|
| reservationKey | bytes32 | Unique key associated with the reservation |
| labId       | uint     | Associated lab ID            |
| renter      | address  | Renter address               |
| price       | uint96   | Rental price                 |
| start       | uint32   | Start time (Unix)            |
| end         | uint32   | End time (Unix)              |
| status      | status   | State of the reservation: 0 = PENDING, 1 = BOOKED, 2 = USED, 3 = COLLECTED, 4 = CANCELLED |


---

## ⚙️ Functional Requirements

Below, each implemented function is listed.
## ProviderFacet:

- **initialize**: Sets up the initial contract state and assigns admin roles
- **addProvider**: Grants PROVIDER_ROLE to a specified account and mints $LAB tokens.
- **removeProvider**: Removes the caller’s PROVIDER_ROLE if conditions are met.
- **updateProvider**: Updates the caller's provider information (name, email, country).
- **isLabProvider**: Checks if a given account holds the PROVIDER_ROLE.
- **getLabProviders**: Retrieves a list of all lab providers.

Since ProviderFacet extends OpenZeppelin’s [OpenZeppelin’s](https://github.com/OpenZeppelin) [AccessControlUpgradeable.sol](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/access/AccessControlUpgradeable.sol) contract, it inherits all role-based access control functionalities provided by the OpenZeppelin library.
## LabFacet:

- **initialize**: Sets up the ERC721 token with the provided name and symbol. Initializes the labId to 0.
- **addLab**: Allows the contract provider to add a new lab with the on-chain stored metadata.
- **setTokenURI**: Allows the contract provider to set or update the URI for the file containing the off-chain metadata of a specific lab token ID.
- **tokenURI**: Returns the URI for the off-chain metadata of a given lab/token ID. Used for compliance with ERC721 standards.
- **updateLab**: Allows the contract provider to update metadata stored on-chain, including the token URI, which links to the metadata stored off-chain.
- **deleteLab**: Allows the contract provider to delete an existing lab.
- **getLab**: Retrieves the information about the lab structure (on-chain metadata) corresponding to the provided lab ID.
- **getAllLabs**: Retrieves a list with all the lab IDs.

Since LabFacet extends OpenZeppelin’s [OpenZeppelin’s](https://github.com/OpenZeppelin) [ERC721EnumerableUpgradeable.sol](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/token/ERC721/extensions/ERC721EnumerableUpgradeable.sol) contract, it inherits all the standard ERC-721 enumerable functionalities, including token enumeration and indexing capabilities.

## ReservationFacet:

- **reservationRequest**: Allows a user to request a booking for a lab.
- **confimReservationRequest**: The lab provider or authorized account confirms a pending reservation request for a lab.
- **denyReservationRequest**: Denies a pending reservation request and refunds the payment if necessary.
- **cancelBookRequest**: Allows a user to cancel a previously requested reservation and refunds the payment if necessary.
- **cancelBooking**: Allows a user or the lab provider to cancel an existing confirmed booking.
- **requestFunds**: Allows lab providers to claim funds from used or expired reservations.
- **getAllReservations**: Retrieves all reservation records stored in the contract.
- **getLabTokenAddress**: Returns the address of the $LAB token contract, set at ProviderFacet initialization.
- **getSafeBalance**: Retrieves the total balance of $LAB funds held in the contract.

Since ReservationFacet implements the enumerable extension of the newly proposed Reservable Token, as defined in [ReservableTokenEnumerable.sol](erc-reservable-token.md), it also inherits all its functions.

Use case Specification
**Detailed information on how each specific use case is executed is provided below**:

### 💎 **ProviderFacet**:

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
| Definition     | `function addProvider(string memory _name, address _account, string memory _email, string memory _country) external defaultAdminRole` |
| Actors         | Contract owner |
| Purpose        | Adds a new provider by granting the PROVIDER_ROLE and minting tokens for the specified account. |
| Summary        | Only the contract owner can add a new provider. |
| Preconditions  | The account must not already have the PROVIDER_ROLE. |
| Postconditions | The account receives the PROVIDER_ROLE and 1000 $LAB ERC20 tokens are minted for them. |
| Events         | Emits an {ProviderAdded} event if the provider is successfully added. |

|                | Description |
|----------------|-------------|
| Use case       | **REMOVE SPECIFIC PROVIDER** |
| Definition     | `function removeProvider(address _provider) external defaultAdminRole` |
| Actors         | Contract owner |
| Purpose        | Removes a specified provider from the provider list if they do not have any lab. |
| Summary        | Only the contract owner can remove a provider from the list. |
| Preconditions  | The provider must not own any lab. |
| Postconditions | The specified provider's role is revoked if conditions are met. |
| Events         | Emits an {ProviderRemoved} event if the provider is successfully removed. |

|                | Description |
|----------------|-------------|
| Use case       | **UPDATE PROVIDER** |
| Definition     | `function updateProvider(string memory _name, string memory _email, string memory _country) external onlyRole(PROVIDER_ROLE)` |
| Actors         | Provider |
| Purpose        | Updates the provider information for the caller, modifying their name, email, and country details. |
| Summary        | Only a provider can update their own information. |
| Preconditions  | The caller must be an existing provider. |
| Postconditions | The provider's information is updated with the new details provided. |
| Events         | Emits an {ProviderUpdated} event if the provider is successfully updated. |

#### 🔍 View Functions
The functions listed below are queries that do not modify the state of the variables:

| Function Name | Definition | Purpose | Return Type |
|---------------|------------|---------|-------------|
| isLabProvider    | `function isLabProvider(address _account) external view returns (bool)` | Checks if the given account is a lab provider. | bool |
| getLabProviders  | `function getLabProviders() external view returns (Provider[] memory)` | Retrieves the list of all lab providers. | Provider array |

#### 📢 Events

The following table lists the events emitted by ProviderFacet.

| Event            | Description                                        | Parameters                                                                 |
|------------------|----------------------------------------------------|----------------------------------------------------------------------------|
| `ProviderAdded`   | Emitted when a new provider is added to the system. | `(address _account)` Address of the provider<br>`(string _name)` Name of the provider<br>`(string _email)` Email address<br>`(string _country)` Country |
| `ProviderRemoved` | Emitted when a provider is removed.                | `(address _account)` Address of the provider                         |
| `ProviderUpdated` | Emitted when a provider's information is updated.  | `(address _account)` Address of the provider<br>`(string _name)` Name of the provider<br>`(string _email)` Email address<br>`(string _country)` Country |

### 💎 **LabFacet**:

|                | Description |
|----------------|-------------|
| Use case       | **INITIALIZE** |
| Definition     | `function initialize(string memory _name, string memory _symbol) public initializer` |
| Actors         | Contract owner |
| Purpose        | Sets up the ERC721 token with the provided name and symbol, and initializes the labId to 0. |
| Summary        | Only the contract owner can initialize the contract |
| Preconditions  | Have a WALLET and sufficient funds. Can only be executed once |
| Postconditions | The ERC721 token associated with de labs is initializes  |

|                | Description |
|----------------|-------------|
| Use case       | **ADD LAB** |
| Definition     | `function addLab(string memory _uri, uint96 _price, string memory _auth, string memory _accessURI, string memory _accessKey) external isLabProvider` |
| Actors         | Providers |
| Purpose        | Allows the contract provider to add a new lab with the on-chain stored metadata |
| Summary        | Only a provider can add a new lab |
| Preconditions  | Caller must be the lab provider |
| Postconditions | A new lab is registered with the given metadata |
| Events         | Emits a {LabAdded} event if successful |

|                | Description |
|----------------|-------------|
| Use case       | **SET TOKEN URI** |
| Definition     | `function setTokenURI(uint256 _labId, string memory _tokenURI) external` |
| Actors         | Providers |
| Purpose        | Allows the lab provider to set or update the URI for a specific lab ID |
| Summary        | Only the lab provider can modify the lab URI |
| Preconditions  | Caller must be the lab provider; the lab ID must exist |
| Postconditions | The specified lab's URI is updated |
| Events         | Emits a {LabURISet} event if successful |

|                | Description |
|----------------|-------------|
| Use case       | **UPDATE LAB** |
| Definition     | `function updateLab(uint _labId, string memory _uri, uint96 _price, string memory _auth, string memory _accessURI, string memory _accessKey) external onlyLabProvider(_labId)` |
| Actors         | Provider |
| Purpose        | Allows the lab provider to update the on-chain stored metadata of an existing lab |
| Summary        | Only the lab provider can modify lab details |
| Preconditions  | Caller must be the lab provider; lab ID must exist |
| Postconditions | The specified lab's metadata are updated |
| Events         | Emits a {LabUpdated} event if successful |

|                | Description |
|----------------|-------------|
| Use case       | **DELETE LAB** |
| Definition     | `function deleteLab(uint _labId) external onlyLabProvider(_labId)` |
| Actors         | Providers |
| Purpose        | Allows the lab provider to delete an existing lab |
| Summary        | Only the lab provider can remove a lab |
| Preconditions  | Caller must be the lab provider; lab ID must exist and be removable |
| Postconditions | The specified lab is removed from the system |
| Events         | Emits a {LabDeleted} event if successful |

#### 🔍 View Functions
The functions listed below are queries that do not modify the state of the variables:

| Function Name | Definition | Purpose | Return Type |
|---------------|------------|---------|-------------|
| getLab        | `function getLab(uint _labId) public view returns (Lab memory)` | Retrieves the lab associated with the given lab ID. | Lab |
| getAllLabs    | `function getAllLabs() public view returns (uint256[] memory)` | Retrieves the list of the all labs ID. | ID (uint256) array |
| tokenURI      | `function tokenURI(uint256 _labId) public view returns (string memory)` | Retrieves the URI associated with a specific lab ID. | URI string |

#### 📢 Events
The following table lists the events emitted by the LabFacet.

| Event         | Description                                       | Parameters                                                                                                                                  |
|---------------|---------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------|
| `LabAdded`     | Emitted when a new lab is added to the system.     | `(uint256 _labId)` Lab identifier<br>`(address _provider)` Provider address<br>`(string _uri)` Metadata URI<br>`(uint96 _price)` Lab price<br>`(string _auth)` Authorization details<br>`(string _accessURI)` Access URI<br>`(string _accessKey)` Access key |
| `LabUpdated`   | Emitted when a lab is updated.                    | `(uint256 _labId)` Lab identifier<br>`(string _uri)` Updated URI<br>`(uint96 _price)` Updated price<br>`(string _auth)` Updated auth<br>`(string _accessURI)` Updated access URI<br>`(string _accessKey)` Updated access key |
| `LabDeleted`   | Emitted when a lab is deleted.                    | `(uint256 _labId)` Lab identifier                                                                                                           |
| `LabURISet`    | Emitted when the URI of a lab is set.             | `(uint256 _labId)` Lab identifier<br>`(string _uri)` URI of the lab                                                                         |

### 💎 **ReservationFacet**:

|                | Description |
|----------------|-------------|
| Use case       | **RESERVATION REQUEST** |
| Definition     | `function reservationRequest(uint256 _labId, uint32 _start, uint32 _end) external exists(_labId) override` |
| Actors         | Users |
| Purpose        | Initiates a new reservation request |
| Summary        | Creates a reservation request with specified details |
| Preconditions  | The lab must exists |
| Postconditions | Reservation state is updated to PENDING |
| Events         | {ReservationRequested} |

|                | Description |
|----------------|-------------|
| Use case       | **CONFIM RESERVATION REQUEST** |
| Definition     | `function confimReservationRequest(bytes32 _reservationKey) external defaultAdminRole reservationPending(_reservationKey) override` |
| Actors         | Providers or authorized account |
| Purpose        | Confirm and book the reservation |
| Summary        | Creates a reservation request with specified details |
| Preconditions  | Caller must be the lab provider (for now, the DEFAULT_ADMIN_ROLE) |
| Postconditions | State is updated to BOOKED |
| Events         | Emits a {ReservationConfirmed} event if successful |

|                | Description |
|----------------|-------------|
| Use case       | **DENY RESERVATION REQUEST** |
| Definition     | `function denyReservationRequest(bytes32 _reservationKey) external defaultAdminRole reservationPending(_reservationKey) override` |
| Actors         | Providers or authorized account |
| Purpose        | Denies a pending reservation request |
| Summary        | Reservation request is marked as denied |
| Preconditions  | Caller must be the lab provider (for now, the DEFAULT_ADMIN_ROLE) |
| Postconditions | State is updated to CANCELLED |
| Events         | Emits a {ReservationRequestDenied} event if successful |

|                | Description |
|----------------|-------------|
| Use case       | **CANCEL RESERVATION REQUEST** |
| Definition     | `function cancelReservationRequest(bytes32 _reservationKey) external override` |
| Actors         | Users |
| Purpose        | Allows cancellation of an existing reservation |
| Summary        | Cancels an active reservation |
| Preconditions  | Caller must be the renter |
| Postconditions | State is updated to CANCELLED |
| Events         | Emits a {ReservationRequestCanceled} event if successful |

|                | Description |
|----------------|-------------|
| Use case       | **CANCEL BOOKING** |
| Definition     | `function cancelBooking(bytes32 _reservationKey) external override` |
| Actors         | Users, providers |
| Purpose        | Allows cancellation of an existing  booking |
| Summary        | Cancels an active booking |
| Preconditions  | Caller must be the renter or the lab provider |
| Postconditions | State is updated to CANCELLED |
| Events         | Emits a {BookingCanceled} event if successful |

|                | Description |
|----------------|-------------|
| Use case       | **REQUEST FUNDS** |
| Definition     | `function requestFunds() external isLabProvider` |
| Actors         | Providers |
| Purpose        | Allows lab providers to claim funds from used or expired reservations |
| Summary        | Creates a reservation request with specified details |
| Preconditions  | Caller must be a registered lab provider |
| Postconditions | Transfers the total amount of $LAB tokens from all eligible reservations to the provider |
| Events         | None |


#### 🔍 View Functions
The functions listed below are queries that do not modify the state of the variables:

| Function Name   | Definition                                                                 | Purpose                                                   | Return Type   |
|-----------------|----------------------------------------------------------------------------|-----------------------------------------------------------|---------------|
| getAllReservations | `function getAllReservations() external view returns (Reservation[] memory)` | Retrieves all reservations stored in the contract | Reservation[] memory |
| getLabTokenAddress | `function getLabTokenAddress() external view returns (address)` | Returns the address of the $LAB ERC20 token 
| getSafeBalance  | `function getSafeBalance() public view returns (uint256)` | Returns the current balance of Lab tokens held by this contract | uint256 |

#### 📢 Events
The following table lists the events emitted by the ProviderFacet.

| Event         | Description                                       | Parameters |
|---------------|---------------------------------------------------|-------------|
| `ReservationRequested` | Emitted when a user submits a new reservation request. | `(address) renter`<br>`(uint256) tokenId`<br>`(uint256) start`<br>`(uint256) end`<br>`(bytes32) reservationKey` |
| `ReservationConfirmed` | Emitted when a reservation request is confirmed. | `(bytes32) reservationKey` |
| `ReservationRequestDenied` | Emitted when a reservation request is denied. | `(bytes32) reservationKey` |
| `ReservationRequestCanceled` | Emitted when a reservation request is canceled by the user. | `(bytes32) reservationKey` |
| `BookingCanceled` | Emitted when a confirmed booking is canceled. | `(bytes32) reservationKey` |

## 💳 Token Integration

The project uses a dedicated **ERC-20 token: `$LAB`** for all transactions. The token is deployed outside the diamond proxy.
