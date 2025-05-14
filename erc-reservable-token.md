
# 🌐 ERC-XXXX: Reservable Token Standard

> 📝 Draft proposal for a new ERC extension to ERC-721 enabling time-based reservation of non-fungible tokens. ⚠️ Not finished ⚠️ .

## ✨ Simple Summary

A standard interface for ERC-721 tokens that allows owners to list their tokens for reservation during specified time intervals, enabling decentralized booking, rentals, and time-slot management.

## 📜 Abstract

This standard proposes an extension to the ERC-721 Non-Fungible Token Standard to support reservable NFTs. It enables the reservation of NFTs for specific time intervals, supporting use cases such as decentralized rentals, lab/equipment booking, and access control based on time. The standard introduces reservation states, calendar conflict prevention, and lifecycle events for decentralized time-bound token usage.

> ⚠️ This proposal serves as a refined and complete alternative to previous proposals like [ERC-809](https://github.com/ethereum/EIPs/issues/809) and [ERC-1201](https://github.com/ethereum/EIPs/issues/1201), which did not reach final standard status. It addresses missing functionality, unification of terminology, and broader applicability.

## 🎯 Motivation

Use cases for NFTs increasingly include real-world and digital assets requiring time-bound access — such as booking shared resources, event tickets, or property rentals. This extension enables standardized reservation and scheduling functionality, avoiding ad hoc implementations and enabling interoperable reservation-based systems.

## 🧩 Specification

### Interfaces

```solidity
iinterface IReservableToken {
    /// @notice Lists a token, making it available for reservation
    /// @param _tokenId The ID of the token to list
    function listToken(uint256 _tokenId) external;

    /// @notice Unlists a token, removing it from the reservation system
    /// @param _tokenId The ID of the token to unlist
    function unlistToken(uint256 _tokenId) external;

    /// @notice Requests a reservation for a specific time range
    /// @param _tokenId The ID of the token to reserve
    /// @param _start The start timestamp of the reservation
    /// @param _end The end timestamp of the reservation
    function reservationRequest(uint256 _tokenId, uint32 _start, uint32 _end) external;

    /// @notice Cancels a pending reservation request
    /// @param _reservationKey The unique key identifying the reservation
    function cancelReservationRequest(bytes32 _reservationKey) external;

    /// @notice Confirms a pending reservation request
    /// @param _reservationKey The unique key identifying the reservation
    function confimReservationRequest(bytes32 _reservationKey) external;

    /// @notice Denies a pending reservation request
    /// @param _reservationKey The unique key identifying the reservation
    function denyReservationRequest(bytes32 _reservationKey) external;

    /// @notice Cancels an existing booking
    /// @param _reservationKey The unique key identifying the reservation
    function cancelBooking(bytes32 _reservationKey) external;

    /// @notice Retrieves reservation details by reservation key
    /// @param _reservationKey The unique key of the reservation
    /// @return Reservation struct containing reservation data
    function getReservation(bytes32 _reservationKey) external view returns (
        uint256 labId,
        address renter,
        uint96 price,
        uint32 start,
        uint32 end,
        uint status
    );

    /// @notice Checks if a reservation is actively booked by a user
    /// @param _reservationKey The reservation key to check
    /// @param _user The address of the user
    /// @return True if the user has an active booking
    function hasActiveBooking(bytes32 _reservationKey, address _user) external view returns (bool);

    /// @notice Returns the address of the user who owns the reservation
    /// @param _reservationKey The reservation key
    /// @return Address of the user
    function userOfReservation(bytes32 _reservationKey) external view returns (address);

    /// @notice Checks whether a token is listed as reservable
    /// @param _tokenId The token ID to check
    /// @return True if listed, false otherwise
    function isTokenListed(uint256 _tokenId) external view returns (bool);

    /// @notice Checks if a token is available for reservation in a given time range
    /// @param _tokenId The ID of the token
    /// @param _start The start time of the requested reservation
    /// @param _end The end time of the requested reservation
    /// @return True if the time range is available
    function checkAvailable(uint256 _tokenId, uint256 _start, uint256 _end) external view returns (bool);
}


```

## 🤔 Rationale

The interface provides essential functionality to:
- ✅ List/unlist tokens as reservable
- 📆 Request, confirm, and cancel reservations
- 🗓️ View reservation calendars per token
- ⛔ Avoid overlapping intervals using a time-interval library (e.g., AVL tree)

This encourages composability with marketplaces, decentralized identity, and access management.

## 🔐 Security Considerations

- ⛓️ Prevent time-slot collisions using interval trees
- 🔐 Require authentication for token owner and reservation initiator
- ⚖️ Handle overlapping and concurrent reservations gracefully
- 📡 Emit events to support transparent off-chain indexing

## 🛠️ Reference Implementation

A full implementation is available at:
[GitHub Link TBD – based on ReservableToken.sol]

## ⚖️ Copyright

Copyright and license under [GNU GPLv2 or later]


---

## 🔄 Extension: ReservableTokenEnumerable

> 📦 This abstract contract extends `ReservableToken` to include enumerable reservation capabilities using `EnumerableSet`.

### ✨ Features Added

- 🔢 Enumeration of reservations per token
- 📇 Query reservations by renter or token
- 📬 Efficient indexing using `EnumerableSet.Bytes32Set`
- 🕒 Precise time interval conflict detection using interval trees

### 📦 Key Functions

This contract introduces new capabilities including (but not limited to):

```solidity
/// @notice Returns the total number of reservation entries across all tokens
/// @return The number of total reservation keys
function totalReservations() external view;

/// @notice Retrieves all reservation keys associated with a specific user
/// @param _user The address of the user
/// @return Array of reservation keys for the given user
function reservationsOf(address _user) external view;

/// @notice Returns the reservation key of a specific user at a given index
/// @param _user The address of the user
/// @param _index The index in the user's reservation set
/// @return The reservation key at the specified index
function reservationKeyOfUserByIndex(address _user, uint256 _index) external view;

/// @notice Returns the reservation key at a specific global index
/// @param _index The global index of the reservation key
/// @return The reservation key at the specified index
function reservationKeyByIndex(uint256 _index) external view;

/// @notice Checks if a user has an active booking on a given token
/// @param _tokenId The ID of the token
/// @param _user The address of the user
/// @return True if the user has an active booking, false otherwise
function hasActiveBookingByToken(uint256 _tokenId, address _user) external view;

/// @notice Retrieves all reservations made for a specific token
/// @param _tokenId The ID of the token
/// @return Array of reservations for the token
function getReservationsOfToken(uint256 _tokenId) public view;

/// @notice Retrieves a reservation of a specific token by index
/// @param _tokenId The ID of the token
/// @param _index The index of the reservation in the token's reservation list
/// @return The reservation at the specified index
function getReservationOfTokenByIndex(uint256 _tokenId, uint256 _index) external view;

/// @notice Internal: Returns all reservation keys stored in a calendar tree
/// @param _calendar The interval tree where reservations are stored
/// @return Array of reservation keys in the calendar
function _getAllKeys(Tree storage _calendar) internal view;

/// @notice Internal: Returns the number of nodes (reservations) in a calendar tree
/// @param _calendar The interval tree where reservations are stored
/// @return The size of the calendar tree
function _size(Tree storage _calendar) internal view;

```

> 💡 Reservations are tracked internally using interval trees and sets of hashes or keys to allow enumeration and lookup efficiency.

### 🧪 Libraries Used

- `RivalIntervalTreeLibrary` — for managing overlapping time ranges
- `EnumerableSet` from OpenZeppelin — for indexed access to reservation identifiers

### 🧠 Use Cases Enhanced

- Lab booking systems with user dashboards
- NFT-based rental markets needing calendar views
- Event space or asset scheduling where users need to see all their upcoming reservations



---


## 📣 Events

These events enable off-chain services (e.g., UIs, analytics, or subgraphs) to track the lifecycle of a reservation and its associated status.

```solidity
/// @notice Emitted when a reservation is requested for a token.
/// @param renter The address of the user requesting the reservation.
/// @param tokenId The ID of the token being reserved.
/// @param start The start timestamp of the reservation period.
/// @param end The end timestamp of the reservation period.
/// @param reservationKey A unique key identifying the reservation.
event ReservationRequested(
    address renter,
    uint256 tokenId,
    uint256 start,
    uint256 end,
    bytes32 reservationKey
);

/// @notice Emitted when a reservation is successfully confirmed.
/// @param reservationKey The unique identifier for the confirmed reservation.
event ReservationConfirmed(bytes32 reservationKey);

/// @notice Emitted when a reservation request is denied.
/// @param reservationKey The unique key identifying the reservation that was denied.
event ReservationRequestDenied(bytes32 reservationKey);

/// @notice Emitted when a reservation request is canceled.
/// @param reservationKey The unique identifier of the reservation that was canceled.
event ReservationRequestCanceled(bytes32 reservationKey);

/// @notice Emitted when a booking associated with a specific reservation key is canceled.
/// @param reservationKey The unique identifier for the reservation that was canceled.
event BookingCanceled(bytes32 reservationKey);
```

> ℹ️ These events are crucial for tracking reservation status transitions in decentralized booking systems, especially in UIs and indexing layers like The Graph.


## 💎 Compatibility: EIP-2535 Diamond Standard

> This standard is designed to be fully compatible with [EIP-2535 Diamonds](https://eips.ethereum.org/EIPS/eip-2535), enabling modular deployment of facets that implement reservable functionality.

### 🏗️ Facet Architecture

The reservable token logic can be deployed as a facet within a Diamond proxy system. This allows developers to:

- Add, replace, or remove reservation logic without redeploying the entire contract
- Share state using a central `AppStorage` struct pattern
- Keep separation of concerns between reservation management, token core, and marketplace logic

### 📐 Recommended Setup

- Use `ReservableToken` as the base facet with core reservation functionality
- Extend with `ReservableTokenEnumerable` if enumeration or querying capabilities are needed
- Share storage using `LibAppStorage` with the `DiamondStorage` pattern
- Interact through `delegatecall` to preserve upgradeability and modular logic separation

### 🔐 Access Control & Security

- Ensure facets verify caller permissions (owner, renter) properly
- Emit standard events for off-chain indexing and UI syncing
- All state modifications are performed via Diamond-compatible storage layout

### 🧩 Benefits of Diamond Integration

- Gas-efficient modular upgrades
- Composability with other facet-based systems
- Code separation between UI/logic/data layers

> 🛠️ Example: `DiamondCutFacet` manages upgrades, while `ReservableFacet` handles ERC-721 reservations using interval trees and enumerable sets.

