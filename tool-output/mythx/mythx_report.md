# Root

In order to run MythX, `Root.sol` contract was flattened. [flat_root.sol](flat_root.sol) line numbers reflect on the output of `truffle-flattener contracts/Root.sol`.

```
Title: Floating Pragma
Head: A floating pragma is set.
Description: It is recommended to make a conscious choice on what version of Solidity is used for compilation. Currently any version equal or greater than "0.4.24" is allowed.
Source code:

flat_root.sol 1:0
--------------------------------------------------
pragma solidity ^0.4.24;
--------------------------------------------------

==================================================
 Title: Shadowing State Variables
Head: State variable shadows another state variable.
Description: The state variable "CLASS_INET" in contract "DNSClaimChecker" shadows another state variable with the same name "CLASS_INET" in contract "Root".
Source code:

flat_root.sol 869:4
--------------------------------------------------
uint16 constant CLASS_INET = 1
--------------------------------------------------

==================================================
 Title: Shadowing State Variables
Head: State variable shadows another state variable.
Description: The state variable "TYPE_TXT" in contract "DNSClaimChecker" shadows another state variable with the same name "TYPE_TXT" in contract "Root".
Source code:

flat_root.sol 870:4
--------------------------------------------------
uint16 constant TYPE_TXT = 16
--------------------------------------------------

==================================================
 Title: Shadowing State Variables
Head: Local variable shadows a state variable.
Description: The local variable "owner" in contract "ENS" shadows the state variable with the same name "owner" in contract "Ownable".
Source code:

flat_root.sol 20:58
--------------------------------------------------
address owner
--------------------------------------------------
flat_root.sol 22:36
--------------------------------------------------
address owner
--------------------------------------------------

==================================================
 Title: Shadowing State Variables
Head: Local variable shadows a state variable.
Description: The local variable "owner" in contract "Root" shadows the state variable with the same name "owner" in contract "Ownable".
Source code:

flat_root.sol 1012:44
--------------------------------------------------
address owner
--------------------------------------------------
flat_root.sol 1037:36
--------------------------------------------------
address owner
--------------------------------------------------

==================================================
 Title: Shadowing State Variables
Head: Local variable shadows a state variable.
Description: The local variable "oracle" in contract "DNSClaimChecker" shadows the state variable with the same name "oracle" in contract "Root".
Source code:

flat_root.sol 881:29
--------------------------------------------------
DNSSEC oracle
--------------------------------------------------

==================================================
```

# BaseRegistrarImplementation

Mythril Classic results for `BaseRegistrarImplementation` are as follows. [flat_BaseRegistrarImplementation.sol](flat_BaseRegistrarImplementation.sol) line numbers reflect on the output of `truffle-flattener contracts/BaseRegistrarImplementation.sol`

```==== Exception State ====
SWC ID: 110
Severity: Low
Contract: BaseRegistrarImplementation
Function name: [available(uint256), available(uint256)] (ambiguous)
PC address: 4722
Estimated Gas Usage: 3414 - 38591
A reachable exception has been detected.
It is possible to trigger an exception (opcode 0xfe). Exceptions can be caused by type errors, division by zero, out-of-bounds array access, or assert violations. Note that explicit `assert()` should only be used to check invariants. Use `require()` for regular input checking.
--------------------
In file: flat_BaseRegistrarImplementation.sol:1443

previousRegistrar.state(bytes32(id)) == Registrar.Mode.Open

--------------------
```

# ETHRegistrarController

Mythril Classic results for `ETHRegistrarController` are as follows. [flat_ETHRegistrarController.sol](flat_ETHRegistrarController.sol) line numbers reflect on the output of `truffle-flattener contracts/ETHRegistrarController.sol`.

```==== Multiple Calls in a Single Transaction ====
SWC ID: 113
Severity: Medium
Contract: ETHRegistrarController
Function name: rentPrice(string,uint256)
PC address: 996
Estimated Gas Usage: 5179 - 79151
Multiple sends are executed in one transaction.
Consecutive calls are executed at the following bytecode offsets:
Offset: 2947
Offset: 3202
Try to isolate each external call into its own transaction, as external calls can fail accidentally or deliberately.

--------------------
In file: flat_ETHRegistrarController.sol:1467

function rentPrice(string memory name, uint duration) view public returns(uint) {
        bytes32 hash = keccak256(bytes(name));
        return prices.price(name, base.nameExpires(uint256(hash)), duration);
    }

--------------------

==== Dependence on predictable environment variable ====
SWC ID: 116
Severity: Low
Contract: ETHRegistrarController
Function name: register(string,address,uint256,bytes32)
PC address: 3552
Estimated Gas Usage: 2056 - 6247
Sending of Ether depends on a predictable variable.
The contract sends Ether depending on the values of the following variables:
- block.timestamp
- block.timestamp
- block.timestamp
Note that the values of variables like coinbase, gaslimit, block number and timestamp are predictable and/or can be manipulated by a malicious miner. Don't use them for random number generation or to make critical decisions.
--------------------
In file: flat_ETHRegistrarController.sol:1498

msg.sender.transfer(msg.value)

--------------------

==== Integer Overflow ====
SWC ID: 101
Severity: High
Contract: ETHRegistrarController
Function name: register(string,address,uint256,bytes32)
PC address: 5332
Estimated Gas Usage: 2333 - 9254
The binary addition can overflow.
The operands of the addition operation are not sufficiently constrained. The addition could therefore result in an integer overflow. Prevent the overflow by checking inputs or ensure sure that the overflow is caught by an assertion.
--------------------
In file: flat_ETHRegistrarController.sol:1411

add(mload(s), ptr)

--------------------
```


