# Compliant-Stablecoin-Demo-dHKD
# Compliant Stablecoin Demo — dHKD

## 1. Project Overview

This project is a hands-on demo of a compliance-oriented fiat-backed stablecoin smart contract deployed on the Ethereum Sepolia testnet.

The goal is to simulate the core on-chain control mechanisms that a regulated stablecoin issuer may need, including:

* token minting and burning;
* role-based access control;
* emergency pause;
* address blacklist / freeze control;
* transparent source code verification on Etherscan.

The token is named **Demo HKD Stablecoin** with the symbol **dHKD**.

This project is part of my Web3 / stablecoin operations portfolio. It demonstrates how traditional financial control concepts, such as authorization, segregation of duties, exception handling, and auditability, can be translated into an on-chain stablecoin context.

---

## 2. Deployment (Remix)

| Item               | Details                                                           |
| ------------------ | ----------------------------------------------------------------- |
| Network            | Ethereum Sepolia Testnet                                          |
| Contract Name      | CompliantStablecoin                                               |
| Token Name         | Demo HKD Stablecoin                                               |
| Token Symbol       | dHKD                                                              |
| Contract Address   | `0xYOUR_CONTRACT_ADDRESS_HERE`                                    |
| Etherscan Link     | https://sepolia.etherscan.io/address/0xYOUR_CONTRACT_ADDRESS_HERE |
| Source Code Status | Verified on Sepolia Etherscan                                     |
| Deployer Wallet    | `0xYOUR_DEPLOYER_WALLET_ADDRESS_HERE`                             |

---

## 3. Key Features

### 3.1 Minting

The contract allows authorized minters to issue new tokens.

In a real fiat-backed stablecoin model, minting would correspond to the issuer receiving fiat funds and then issuing an equivalent amount of stablecoins on-chain.

Function:

```solidity
function mint(address to, uint256 amount) public onlyRole(MINTER_ROLE)
```

Control purpose:

* prevents unauthorized token issuance;
* keeps minting restricted to approved operational roles;
* supports a clear issuance control process.

---

### 3.2 Burning

The contract supports token burning.

In a real redemption process, burning would correspond to the user returning stablecoins and the issuer releasing the equivalent fiat funds off-chain.

Control purpose:

* supports redemption and supply reduction;
* allows total supply to reflect redeemed tokens;
* creates an auditable on-chain record of token destruction.

---

### 3.3 Role-Based Access Control

The contract uses OpenZeppelin `AccessControl` to separate operational permissions.

Defined roles:

| Role                 | Purpose                                           |
| -------------------- | ------------------------------------------------- |
| `DEFAULT_ADMIN_ROLE` | Admin role that can manage other roles            |
| `MINTER_ROLE`        | Authorized to mint new tokens                     |
| `PAUSER_ROLE`        | Authorized to pause and unpause transfers         |
| `BLACKLISTER_ROLE`   | Authorized to blacklist and unblacklist addresses |

Control purpose:

* separates sensitive permissions;
* reduces the risk of a single uncontrolled operator;
* mirrors real-world segregation of duties in financial operations.

---

### 3.4 Emergency Pause

The contract includes a global pause function.

Functions:

```solidity
function pause() public onlyRole(PAUSER_ROLE)
function unpause() public onlyRole(PAUSER_ROLE)
```

When the contract is paused, token transfers are blocked.

Possible use cases:

* smart contract incident;
* suspected exploit;
* reserve or peg-related emergency;
* operational incident requiring temporary suspension.

Control purpose:

* provides an emergency stop mechanism;
* allows the issuer to contain risk quickly;
* supports incident response and business continuity planning.

---

### 3.5 Address Blacklist / Freeze Control

The contract includes address-level blacklist controls.

Functions:

```solidity
function blacklist(address account) public onlyRole(BLACKLISTER_ROLE)
function unBlacklist(address account) public onlyRole(BLACKLISTER_ROLE)
function isBlacklisted(address account) public view returns (bool)
```

Blacklisted addresses cannot send or receive tokens.

Possible use cases:

* sanctioned address exposure;
* suspected illicit activity;
* stolen funds response;
* compliance or law enforcement request.

Control purpose:

* allows targeted freezing of high-risk addresses;
* supports AML / CFT risk controls;
* avoids relying only on a global pause when a specific address needs to be restricted.

---

## 4. Contract Design Logic

The contract checks both global pause status and blacklist status before token transfers are completed.

Core transfer control logic:

```solidity
function _update(address from, address to, uint256 value)
    internal
    override(ERC20, ERC20Pausable)
{
    require(!_blacklisted[from], "Stablecoin: sender blacklisted");
    require(!_blacklisted[to], "Stablecoin: recipient blacklisted");
    super._update(from, to, value);
}
```

This means:

* if the sender is blacklisted, the transfer fails;
* if the recipient is blacklisted, the transfer fails;
* if the contract is paused, transfers fail through `ERC20Pausable`;
* normal transfers are allowed only when both global and address-level controls pass.

---

## 5. Test Evidence

The following screenshots are included as evidence that the contract was deployed, verified, and tested.

| Screenshot                              | Description                                              |
| --------------------------------------- | -------------------------------------------------------- |
| `screenshots/01-deployed-contract.png`  | Contract successfully deployed in Remix                  |
| `screenshots/02-etherscan-verified.png` | Source code verified on Sepolia Etherscan                |
| `screenshots/03-mint-success.png`       | Mint function executed successfully                      |
| `screenshots/04-blacklist-revert.png`   | Transfer blocked after recipient address was blacklisted |
| `screenshots/05-pause-revert.png`       | Transfer blocked while the contract was paused           |
| `screenshots/06-burn-success.png`       | Burn function executed successfully                      |

---

## 6. Test Scenarios

### Test 1 — Mint Tokens

Action:

* Mint tokens to the deployer wallet.

Expected result:

* Token balance increases.
* Total supply increases.

Status:

* Passed.

---

### Test 2 — Normal Transfer

Action:

* Transfer tokens from the deployer wallet to another test wallet.

Expected result:

* Transfer succeeds when neither address is blacklisted and the contract is not paused.

Status:

* Passed.

---

### Test 3 — Blacklist Address

Action:

* Add the second wallet address to the blacklist.
* Attempt to transfer tokens to or from the blacklisted address.

Expected result:

* Transaction fails with blacklist-related revert message.

Status:

* Passed.

---

### Test 4 — Pause Contract

Action:

* Call `pause()`.
* Attempt a token transfer while the contract is paused.
* Call `unpause()` and retry the transfer.

Expected result:

* Transfer fails while paused.
* Transfer resumes after unpause.

Status:

* Passed.

---

### Test 5 — Burn Tokens

Action:

* Burn part of the token balance.

Expected result:

* Token balance decreases.
* Total supply decreases.

Status:

* Passed.

---

## 7. Why This Project Matters

This project demonstrates more than basic ERC-20 deployment.

It shows how stablecoin operations can be connected with practical control mechanisms:

| Operational Risk                 | On-Chain Control                  |
| -------------------------------- | --------------------------------- |
| Unauthorized issuance            | `MINTER_ROLE`                     |
| Emergency incident               | `pause()`                         |
| Sanctioned or suspicious address | `blacklist()`                     |
| Redemption / supply reduction    | `burn()`                          |
| Auditability                     | Verified source code on Etherscan |

The project connects smart contract design with compliance, operations, and internal control thinking.

---

## 8. Tools Used

* Solidity
* OpenZeppelin Contracts
* Remix IDE
* MetaMask
* Ethereum Sepolia Testnet
* Sepolia Etherscan

---

## 9. Disclaimer

This project is for educational and portfolio demonstration purposes only.

It is not intended for production use and has not been audited. A production-grade stablecoin would require additional controls, including independent security audits, multi-signature or MPC key management, formal governance procedures, reserve management controls, compliance monitoring, and regulatory approval where applicable.
