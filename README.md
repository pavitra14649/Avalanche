# Solidity by Example: ERC20 Token and Vault

## Introduction

This repository contains two smart contracts written in Solidity:
1. **ERC20 Token Contract**: Implements the ERC20 standard with basic minting, burning, transferring, and approval functionalities.
2. **Vault Contract**: Allows users to deposit and withdraw the ERC20 tokens. The shares are calculated based on the token's balance and total supply.

## Contracts

### 1. ERC20 Token Contract

The ERC20 token contract follows the ERC20 standard and includes the following functionalities:
- **Minting**: Allows minting new tokens.
- **Burning**: Allows burning tokens.
- **Transferring**: Allows transferring tokens between accounts.
- **Approval**: Allows an account to approve another account to spend tokens on its behalf.
- **Allowance**: Manages the allowances set by the token holders.

#### Contract Code

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

contract ERC20 {
    uint public totalSupply;
    mapping(address => uint) public balanceOf;
    mapping(address => mapping(address => uint)) public allowance;
    string public name = "Solidity by Example";
    string public symbol = "SOLBYEX";
    uint8 public decimals = 18;

    event Transfer(address indexed from, address indexed to, uint value);
    event Approval(address indexed owner, address indexed spender, uint value);

    function transfer(address recipient, uint amount) external returns (bool) {
        balanceOf[msg.sender] -= amount;
        balanceOf[recipient] += amount;
        emit Transfer(msg.sender, recipient, amount);
        return true;
    }

    function approve(address spender, uint amount) external returns (bool) {
        allowance[msg.sender][spender] = amount;
        emit Approval(msg.sender, spender, amount);
        return true;
    }

    function transferFrom(
        address sender,
        address recipient,
        uint amount
    ) external returns (bool) {
        allowance[sender][msg.sender] -= amount;
        balanceOf[sender] -= amount;
        balanceOf[recipient] += amount;
        emit Transfer(sender, recipient, amount);
        return true;
    }

    function mint(uint amount) external {
        balanceOf[msg.sender] += amount;
        totalSupply += amount;
        emit Transfer(address(0), msg.sender, amount);
    }

    function burn(uint amount) external {
        balanceOf[msg.sender] -= amount;
        totalSupply -= amount;
        emit Transfer(msg.sender, address(0), amount);
    }
}
```

### 2. Vault Contract

The Vault contract allows users to deposit and withdraw the ERC20 tokens. It mints and burns shares based on the amount of tokens deposited or withdrawn, respectively.

#### Contract Code

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

interface IERC20 {
    function totalSupply() external view returns (uint);
    function balanceOf(address account) external view returns (uint);
    function transfer(address recipient, uint amount) external returns (bool);
    function allowance(address owner, address spender) external view returns (uint);
    function approve(address spender, uint amount) external returns (bool);
    function transferFrom(address sender, address recipient, uint amount) external returns (bool);

    event Transfer(address indexed from, address indexed to, uint value);
    event Approval(address indexed owner, address indexed spender, uint value);
}

contract Vault {
    IERC20 public immutable token;
    uint public totalSupply;
    mapping(address => uint) public balanceOf;

    constructor(address _token) {
        token = IERC20(_token);
    }

    function _mint(address _to, uint _shares) private {
        totalSupply += _shares;
        balanceOf[_to] += _shares;
    }

    function _burn(address _from, uint _shares) private {
        totalSupply -= _shares;
        balanceOf[_from] -= _shares;
    }

    function deposit(uint _amount) external {
        uint shares;
        if (totalSupply == 0) {
            shares = _amount;
        } else {
            shares = (_amount * totalSupply) / token.balanceOf(address(this));
        }
        _mint(msg.sender, shares);
        token.transferFrom(msg.sender, address(this), _amount);
    }

    function withdraw(uint _shares) external {
        uint amount = (_shares * token.balanceOf(address(this))) / totalSupply;
        _burn(msg.sender, _shares);
        token.transfer(msg.sender, amount);
    }
}
```

## How to Use

### Deploying Contracts

1. Deploy the ERC20 contract first.
2. Deploy the Vault contract with the address of the deployed ERC20 contract.

### Interacting with Contracts

#### ERC20 Contract

- **Mint Tokens**: Call the `mint` function with the amount of tokens to mint.
- **Burn Tokens**: Call the `burn` function with the amount of tokens to burn.
- **Transfer Tokens**: Call the `transfer` function with the recipient address and the amount to transfer.
- **Approve Tokens**: Call the `approve` function with the spender address and the amount to approve.
- **Transfer From**: Call the `transferFrom` function with the sender address, recipient address, and the amount to transfer.

#### Vault Contract

- **Deposit Tokens**: Call the `deposit` function with the amount of tokens to deposit. This will mint shares to the caller.
- **Withdraw Tokens**: Call the `withdraw` function with the amount of shares to burn. This will transfer the corresponding amount of tokens back to the caller.

## License

This project is licensed under the MIT License.

---

Feel free to use, modify, and distribute this code with proper attribution. If you have any questions or need further assistance, please refer to the official Solidity documentation or reach out to the community. Happy coding!
