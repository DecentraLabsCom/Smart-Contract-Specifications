# 🧪 LabERC20 – Laboratory $LAB Token for DecentraLabsCom

This smart contract implements a basic ERC20 token intended for controlled testing environments within the **DecentraLabsCom** ecosystem.

## ✨ Key Features

- ✅ Built on [OpenZeppelin](https://openzeppelin.com/contracts): uses `ERC20`, `Initializable` and `ERC20Upgradeable`.
- 🎛️ Configurable on deployment: token name, symbol, and lab address.
- ⚠️ **Security restrictions (access control for mint/burn/lab updates) are not implemented yet.** Use only in fully trusted environments.

### 📦 Key Functions

This contract introduces new capabilities including (but not limited to):

```solidity
/// @notice Initializes the token with given symbol
/// @dev This function can only be called once due to the initializer modifier
/// @param _symbol The symbol for the token
/// @custom:initializer Sets the token name as "$<symbol>" and mints 10M tokens to the deployer
function initialize(string memory _symbol) public initializer;
```

```solidity
/// @notice Mints new tokens and assigns them to the specified account
/// @dev Anyone can call this function as it is public
/// @param account The address that will receive the minted tokens
/// @param amount The amount of tokens to mint
function mint(address account, uint256 amount) public;
```

```solidity
/// @notice Transfer tokens from one address to another
/// @dev Override of ERC20 transferFrom function. Allowance check is handled in _transfer
/// @param sender The address to transfer tokens from
/// @param recipient The address to transfer tokens to
/// @param amount The amount of tokens to transfer
/// @return bool True if the transfer was successful
function transferFrom(address sender, address recipient, uint256 amount) public override returns (bool)
```

```solidity
/// @notice Returns the number of decimal places used in token amounts
/// @dev This implementation uses 6 decimal places for token precision
/// @return uint8 The number of decimal places (6)
function decimals() public pure override returns (uint8)
```

---

> ⚠️ Access control is not implemented. This contract should only be used in trusted or test environments within DecentraLabsCom.
