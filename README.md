# _ENS Permanent Registrar_ Audit

## Update [2019-05-29]

In addition to the results below, ConsenSys Diligence also audited an [updated `Root` contract](https://github.com/ensdomains/root/blob/646a10cba68f38df4d72f4823be010724ec6829a/contracts/Root.sol) with the SHA1 hash `42d807dc438b978b7ddfd7a0c030cf6140bd49d6`.

No new issues were found.

---

<img height="100px" Hspace="30" Vspace="10" align="right" src="static-content/diligence.png"/>

<!-- Don't remove this -->
* [1 Summary](#1-summary)
  * [1.1 Audit Dashboard](#11-audit-dashboard)
  * [1.2 Audit Goals](#12-audit-goals)
  * [1.3 System Overview](#13-system-overview)
  * [1.4 Key Observations/Recommendations](#14-key-observationsrecommendations)
* [2 Issue Overview](#2-issue-overview)
* [3 Issue Detail](#3-issue-detail)
  * [3.1 Memory corruption in `Buffer`](#31-memory-corruption-in-buffer)
  * [3.2 `SimplePriceOracle.price` is susceptible to integer overflow](#32-simplepriceoracleprice-is-susceptible-to-integer-overflow)
  * [3.3 `ETHRegistrarController.register` is vulnerable to front running](#33-ethregistrarcontrollerregister-is-vulnerable-to-front-running)
  * [3.4 SOA record check on the wrong domain](#34-soa-record-check-on-the-wrong-domain)
  * [3.5 Work towards a trustless model for ENS](#35-work-towards-a-trustless-model-for-ens)
  * [3.6 Consider replacing the `Buffer` implementation](#36-consider-replacing-the-buffer-implementation)
  * [3.7 Overzealous resizing in `Buffer`](#37-overzealous-resizing-in-buffer)
  * [3.8 Pending auctions in the legacy registrar don't result in proper ownership in ENS](#38-pending-auctions-in-the-legacy-registrar-dont-result-in-proper-ownership-in-ens)
  * [3.9 `BaseRegistrarImplementation.acceptRegistrarTransfer` should probably use the `live` modifier](#39-baseregistrarimplementationacceptregistrartransfer-should-probably-use-the-live-modifier)
  * [3.10 Reconsider use of inline assembly in `BytesUtils.sol`](#310-reconsider-use-of-inline-assembly-in-bytesutilssol)
  * [3.11 `BaseRegistrarImplementation.acceptRegistrarTransfer` does not check for invalid names](#311-baseregistrarimplementationacceptregistrartransfer-does-not-check-for-invalid-names)
  * [3.12 Sanity check around `transferPeriodEnds`](#312-sanity-check-around-transferperiodends)
  * [3.13 `StablePriceOracle.price` has an unimportant integer underflow](#313-stablepriceoracleprice-has-an-unimportant-integer-underflow)
  * [3.14 `ETHRegistrarController.register` should  `revert` rather than silently fail](#314-ethregistrarcontrollerregister-should--revert-rather-than-silently-fail)
  * [3.15 `StringUtils.strlen` could be rewritten without assembly](#315-stringutilsstrlen-could-be-rewritten-without-assembly)
* [4 Threat Model](#4-threat-model)
  * [4.1 Overview](#41-overview)
  * [4.2 Threat Analysis](#42-threat-analysis)
* [5 Tool based analysis](#5-tool-based-analysis)
  * [5.1 MythX](#51-mythx)
  * [5.2 Ethlint](#52-ethlint)
  * [5.3 Surya](#53-surya)
  * [5.4 Odyssey](#54-odyssey)
* [6 Test Coverage Measurement](#6-test-coverage-measurement)
* [Appendix 1 - File Hashes](#appendix-1---file-hashes)
* [Appendix 2 - Severity](#appendix-2---severity)
  * [A.2.1 - Minor](#a21---minor)
  * [A.2.2 - Medium](#a22---medium)
  * [A.2.3 - Major](#a23---major)
  * [A.2.4 - Critical](#a24---critical)
* [Appendix 3 - Disclosure](#appendix-3---disclosure)

<!-- Don't remove this -->

## 1 Summary

ConsenSys Diligence conducted a security audit on two new components of the [Ethereum Name Service (ENS)](https://ens.domains/): the `.eth` Permanent Registrar and the new ENS `Root`.

* **`.eth` Permanent Registrar** is a new rent-based registrar for the `.eth` top-level domain.
* **New ENS Root** is to allow for certain `onlyOwner` functions to be disintermediated from the ENS root key holders. The main new functionality is the ability for anyone to register a new TLD based on DNSSEC records.

### 1.1 Audit Dashboard

________________

<img height="120px" Hspace="30" Vspace="10" align="right" src="static-content/dashboard.png"/>

#### Audit Details
* **Project Name:** ENS Permanent Registrar
* **Client Name:** Ethereum Name Service
* **Client Contact:** Nick Johnson (nick@ens.domains)
* **Auditors:** Shayan Eskandari, Steve Marx
* **Languages:** Solidity
* **Date:** March 11th, 2019

#### Number of issues per severity


| |  **Minor**  |  **Medium**  |  **Major**  |  **Critical**  | 
|:-------------:|:-------------:|:-------------:|:-------------:|:-------------:|
|  **Open**  | **0**  |  **0**  | **0**  | **0** |
|  **Closed**  | **6**  |  **5**  | **1**  | **3** |


________________

### 1.2 Audit Goals

The focus of the audit was to verify that the smart contract system is secure, resilient and working according to its specifications. The audit activities can be grouped in the following three categories:  

**Security:** Identifying security related issues within each contract and within the system of contracts.

**Sound Architecture:** Evaluation of the architecture of this system through the lens of established smart contract best practices and general software best practices.

**Code Correctness and Quality:** A full review of the contract source code. The primary areas of focus include:

* Correctness
* Readability
* Sections of code with high complexity
* Improving scalability
* Quantity and quality of test coverage

### 1.3 System Overview

#### Documentation

The following documentation was available to the audit team:

* This [Documentation](https://docs.ens.domains/contract-api-reference/.eth-permanent-registrar) with description about the mechanism of the `.eth Permanent Registrar` and `ETHRegistrarController`.

#### Scope

The audit focus was on the smart contract files, and test suites found in the following repositories of the [ensdomains](https://github.com/ensdomains/) GitHub organization:

|  Directory | Commit hash | Commit date |
|----------|-------------|-------------|
| [ensdomains/ethregistrar](https://github.com/ensdomains/ethregistrar/tree/e52abfc2799ac361364aca6135fc20f9175a29fd)  | e52abfc2799ac361364aca6135fc20f9175a29fd | 29th January 2019 |
| [ensdomains/root](https://github.com/ensdomains/root/tree/c82010e34828d72319efb66aae921609d3c7a704) | cc877549939888727468f635af94643deeaa10b3 | 13th February 2019 |

Specifically, the following files were audited:

* BaseRegistrar.sol
* BaseRegistrarImplementation.sol
* ETHRegistrarController.sol
* Ownable.sol
* PriceOracle.sol
* Root.sol
* SafeMath.sol
* SimplePriceOracle.sol
* StablePriceOracle.sol
* StringUtils.sol

There are other components of the pending ENS upgrade that were _not_ part of this audit. Notably, the DNSSEC oracle and DNSSEC-based registrar were out of scope and not considered during this audit.

#### Architecture

[ENS](https://docs.ens.domains/overview) has two principal components: the registry, and resolvers. The system is broadly analogous to DNS.

The ENS registry is a simple smart contract that tracks for each registered name: who owns it, where the name can be resolved, and how long a client may cache name resolution. _Resolvers_ ([EIP-137](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-137.md)) are responsible for actually translating names into addresses.

A _registrar_ is a contract responsible for allocating subdomains and updating records in the ENS registry. Registrars can be configured at any level of ENS, and they are pointed to by the owner field of the registry.

There exists an auction-based _interim `.eth` registrar_ which will be replaced by the new _permanent `.eth` registrar_ (`BaseRegistrarImplementation`). The new registrar implements an ERC721-compatible non-fungible token. Users can interact directly with the non-fungible token to transfer ownership. To obtain an available subdomain or renew subdomain ownership, users interact with any number of _controllers_ connected to the registrar. At the time of this audit, only one controller exists: the `ETHRegistrarController`. This controller handles domain names that are at least 7 characters long and uses a simple rent-based model similar to `.com` domain ownership in DNS. It uses a price oracle (`StablePriceOracle`) to control rent prices in US dollars and convert those prices to ETH.

A new `Root` contract is being introduced that maps TLDs to registrars. It points the `.eth` TLD at the `BaseRegistrarImplementation`, and it points all others at a registrar specified in DNSSEC records for the TLD or a generic DNSSEC-based registrar as a fallback. This allows DNS owners to dictate how ENS names are resolved for their DNS.

The ENS system itself is controlled by a multisig wallet with key stakeholders from the Ethereum community. They have broad control over the system, up to and including replacing the entire registrar implementation. As such, ENS is only trusted to the extent that these key stakeholders are trusted.

### 1.4 Key Observations/Recommendations  

* The system is well designed. The old registrar does proper checks to be disabled when the new system is deployed.
* The code is well written and considers most of the security best practices.
* **Avoid inline assembly**: A great deal of complexity is added by using libraries with inline assembly code. Wherever possible, we recommend avoiding inline assembly in favor of more straightforward Solidity-based implementations.
* **Insufficient test coverage**: Test _code coverage_ is likely high, but there's a lot of room for improvement. See section 6 for more details.

## 2 Issue Overview  

The following table contains all the issues discovered during the audit. The issues are ordered based on their severity. More detailed description on the  levels of severity can be found in Appendix 2. The table also contains the GitHub status of any discovered issue.

| Chapter | Issue Title  | Issue Status | Severity |
| ------------- | ------------- | ------------- | ------------- |
 | 3.1 | [Memory corruption in `Buffer`](#31-memory-corruption-in-buffer) |  Closed |  Critical | 
 | 3.2 | [`SimplePriceOracle.price` is susceptible to integer overflow](#32-simplepriceoracleprice-is-susceptible-to-integer-overflow) |  Closed |  Critical | 
 | 3.3 | [`ETHRegistrarController.register` is vulnerable to front running](#33-ethregistrarcontrollerregister-is-vulnerable-to-front-running) |  Closed |  Critical | 
 | 3.4 | [SOA record check on the wrong domain](#34-soa-record-check-on-the-wrong-domain) |  Closed |  Major | 
 | 3.5 | [Work towards a trustless model for ENS](#35-work-towards-a-trustless-model-for-ens) |  Closed |  Medium | 
 | 3.6 | [Consider replacing the `Buffer` implementation](#36-consider-replacing-the-buffer-implementation) |  Closed |  Medium | 
 | 3.7 | [Overzealous resizing in `Buffer`](#37-overzealous-resizing-in-buffer) |  Closed |  Medium | 
 | 3.8 | [Pending auctions in the legacy registrar don't result in proper ownership in ENS](#38-pending-auctions-in-the-legacy-registrar-dont-result-in-proper-ownership-in-ens) |  Closed |  Medium | 
 | 3.9 | [`BaseRegistrarImplementation.acceptRegistrarTransfer` should probably use the `live` modifier](#39-baseregistrarimplementationacceptregistrartransfer-should-probably-use-the-live-modifier) |  Closed |  Medium | 
 | 3.10 | [Reconsider use of inline assembly in `BytesUtils.sol`](#310-reconsider-use-of-inline-assembly-in-bytesutilssol) |  Closed |  Minor | 
 | 3.11 | [`BaseRegistrarImplementation.acceptRegistrarTransfer` does not check for invalid names](#311-baseregistrarimplementationacceptregistrartransfer-does-not-check-for-invalid-names) |  Closed |  Minor | 
 | 3.12 | [Sanity check around `transferPeriodEnds`](#312-sanity-check-around-transferperiodends) |  Closed |  Minor | 
 | 3.13 | [`StablePriceOracle.price` has an unimportant integer underflow](#313-stablepriceoracleprice-has-an-unimportant-integer-underflow) |  Closed |  Minor | 
 | 3.14 | [`ETHRegistrarController.register` should  `revert` rather than silently fail](#314-ethregistrarcontrollerregister-should--revert-rather-than-silently-fail) |  Closed |  Minor | 
 | 3.15 | [`StringUtils.strlen` could be rewritten without assembly](#315-stringutilsstrlen-could-be-rewritten-without-assembly) |  Closed |  Minor | 



## 3 Issue Detail  

### 3.1 Memory corruption in `Buffer`

| Severity  | Status |  Remediation Comment |
| ------------- | ------------- | ------------- |
|  Critical  |  Closed   |  Issue has been closed in [ensdomains/buffer#3](https://github.com/ensdomains/buffer/pull/3/) |


#### Description

Although out of scope for this audit, the audit team noticed a memory corruption issue in the `Buffer` library. The `init` function is as follows:



**contracts/Buffer.sol:L22-L41**

```Solidity
    /**
    * @dev Initializes a buffer with an initial capacity.
    * @param buf The buffer to initialize.
    * @param capacity The number of bytes of space to allocate the buffer.
    * @return The buffer, for chaining.
    */
    function init(buffer memory buf, uint capacity) internal pure returns(buffer memory) {
        if (capacity % 32 != 0) {
            capacity += 32 - (capacity % 32);
        }
        // Allocate space for the buffer data
        buf.capacity = capacity;
        assembly {
            let ptr := mload(0x40)
            mstore(buf, ptr)
            mstore(ptr, 0)
            mstore(0x40, add(ptr, capacity))
        }
        return buf;
    }
```



Note that memory is reserved only for `capacity` bytes, but the `bytes` actually requires `capacity + 32` bytes to account for the prefixed array length. Other functions in `Buffer` assume correct allocation and therefore corrupt nearby memory.

Although we didn't immediately spot an ENS exploit for this vulnerability, we consider any memory corruption issue to be important to address.

#### Example

A simple test shows the memory corruption issue:

```solidity
contract Test {
    using Buffer for Buffer.buffer;

    function test() external pure {
        Buffer.buffer memory buffer;
        buffer.init(1);

        // foo immediately follows buffer.buf in memory
        bytes memory foo = new bytes(0);
        
        assert(foo.length == 0);

        buffer.append("A");
 
        // "A" == 65, gets written to the high order byte of foo.length
        assert(foo.length == 65 * 256**31);
    }
}
```

#### Remediation

Allocate an additional 32 bytes as follows, to account for storing the `uint256` size of the `bytes` array:

```solidity
mstore(0x40, add(ptr, add(capacity, 32)))
```

### 3.2 `SimplePriceOracle.price` is susceptible to integer overflow

| Severity  | Status |  Remediation Comment |
| ------------- | ------------- | ------------- |
|  Critical  |  Closed   |  Issue has been closed in [ensdomains/ethregistrar#17](https://github.com/ensdomains/ethregistrar/pull/17) by using `SafeMath`. |


#### Description

`SimplePriceOracle.price` is as follows:



**ethregistrar/contracts/SimplePriceOracle.sol:L26-L28**

```Solidity
    function price(string calldata /*name*/, uint /*expires*/, uint duration) external view returns(uint) {
        return duration * rentPrice;
    }
```



This is susceptible to a simple overflow attack, e.g. setting the duration to `2**256/rentPrice` to give yourself a price of 0.

Severity note: It's unclear whether the `SimplePriceOracle` is expected to be used in practice, but the severity is set here under the assumption that the code may be used somewhere.

#### Remediation

Use `SafeMath` or explicitly check for the overflow.

### 3.3 `ETHRegistrarController.register` is vulnerable to front running

| Severity  | Status |  Remediation Comment |
| ------------- | ------------- | ------------- |
|  Critical  |  Closed   |  Issue has been closed in [ensdomains/ethregistrar#18](https://github.com/ensdomains/ethregistrar/pull/18) |


#### Description

`commit()` and then `register()` appears to serve the purpose of preventing front running. However, because the commitment is not tied to a specific owner, it serves equally well as a commitment for a front-running attacker.

#### Example

1. Alice calls `commit(makeCommitment("mydomain", <secret>))`.
2. 10 minutes later, Alice submits a transaction to `register("mydomain", Alice, ..., <secret>)`.
3. Eve observes this transaction in the transaction pool.
4. Eve submits `register("mydomain", Eve, ..., <secret>)` with a higher gas price and wins the race.

#### Remediation

Commitments should commit to `owner`s in addition to `name`s. This way an attacker can't repurpose a previous commitment. (They would have to buy on behalf of the original committer.)

As an alternative, if it's undesirable to pin down `owner`, the commitment could include `msg.sender` instead (only allowing the original committer to call `register`).

E.g. the following (and corresponding changes to callers):

```solidity
function makeCommitment(
    string memory name,
    address owner, /* or perhaps committer/sender */
    bytes32 secret
)
    pure
    public
    returns(bytes32)
{
    bytes32 label = keccak256(bytes(name));
    return keccak256(abi.encodePacked(label, owner, secret));
}
```

### 3.4 SOA record check on the wrong domain

| Severity  | Status |  Remediation Comment |
| ------------- | ------------- | ------------- |
|  Major  |  Closed   |  During the audit, this issue was discovered by the client development team and already fixed in [ensdomains/root#25](https://github.com/ensdomains/root/pull/25). |


#### Description

The SOA record check in `Root.getAddress` is meant to happen on the root TLD, but in the version of the code audited, it is performed instead on `_ens.nic.<tld>`.

### 3.5 Work towards a trustless model for ENS

| Severity  | Status |  Remediation Comment |
| ------------- | ------------- | ------------- |
|  Medium  |  Closed   |  Acknowledged by client team. As stated, this is a long-term issue for which there is no immediate fix, but work is already in progress. |


#### Description

The ENS registry itself is owned by a multisig wallet owned by a number of reputable Ethereum community members. That multisig wallet can do just about anything, up to and including directly taking over any existing or future registered names.

It's important to note that even if we as a community trust the current owners of the multisig wallet, we also need to consider the possibility of their Ethereum private keys being compromised by malicious actors.

#### Remediation

This centralized control is by design, and the multisig owners have been chosen carefully. However, we do recommend—as is already the plan—that the multisig wallet's power be reduced in future updates to the system. Changes made by that wallet are already quite transparent to the community, but future enhancements might include requiring a waiting period for any changes or disallowing certain types of changes altogether.

In the meantime, wherever possible, the trust model should be made clear so that users understand what guarantees they do and do not have when interacting with ENS.

### 3.6 Consider replacing the `Buffer` implementation

| Severity  | Status |  Remediation Comment |
| ------------- | ------------- | ------------- |
|  Medium  |  Closed   |  There will be no immediate fix for this, but the client team is working on collaborating to get a better audited `Buffer` library in place. |


#### Description

The audit team uncovered two bugs in the `Buffer` library, one each in the only two functions that were looked at. (The library was in general _not_ in scope for this audit.) One bug was a critical memory corruption bug. This calls into question how safe this library is to use in general.

#### Remediation

Consider using a different library, ideally one that has been fully tested and audited and that minimizes the use of inline assembly, particularly around memory allocation.

### 3.7 Overzealous resizing in `Buffer`

| Severity  | Status |  Remediation Comment |
| ------------- | ------------- | ------------- |
|  Medium  |  Closed   |  Issue has been closed in [ensdomains/buffer#4](https://github.com/ensdomains/buffer/pull/4/) |


#### Description

In the following code, the buffer is resized even when sufficient `capacity` is available to perform the write. The `buf.buf.length` term is unnecessary and leads to unnecessary resizing:



**contracts/Buffer.sol:L91-L95**

```Solidity
    function write(buffer memory buf, uint off, bytes memory data, uint len) internal pure returns(buffer memory) {
        require(len <= data.length);

        if (off + len > buf.capacity) {
            resize(buf, max(buf.capacity, len + off) * 2);
```



Contrast with the calculation in a similar function:



**contracts/Buffer.sol:L206-L209**

```Solidity
    function write(buffer memory buf, uint off, bytes32 data, uint len) private pure returns(buffer memory) {
        if (len + off > buf.capacity) {
            resize(buf, (len + off) * 2);
        }
```



#### Remediation

Check just the condition `if (off + len > buf.capacity)` when deciding whether to resize the buffer. This will be a significant gas savings in the common case of reserving exactly the right capacity and then performing two `append` operations.

### 3.8 Pending auctions in the legacy registrar don't result in proper ownership in ENS

| Severity  | Status |  Remediation Comment |
| ------------- | ------------- | ------------- |
|  Medium  |  Closed   |  Addressed in [ensdomains/ethregistrar#23](https://github.com/ensdomains/ethregistrar/pull/23) by reducing the waiting period to 28 days. |


#### Description

If an auction has yet to be finalized in the legacy `HashRegistrar` at the time that the new, permanent `.eth` registrar is put in place, the auction winner doesn't get actual ownership of the ENS entry.

The sequence of events would look like:

1. Auction is started in the `HashRegistrar` for the name `something.eth`
2. The new `BaseRegistrarImplementation` becomes the owner of the `.eth` root node in ENS.
3. The auction is won.
4. The auction winner calls `finalizeAuction`, which calls `trySetSubnodeOwner`, which fails to actually set subnode ownership (as the `HashRegistrar` no longer has ownership of the `.eth` root node).

At this point, there's an owner of the deed for the name `something.eth` in the `HashRegistrar`, but the ENS subnode is unowned. It can't be transferred to the new registrar for 183 days, and the name can't be registered in the new registrar.

The owner _can_ get themselves out of this situation by calling `releaseDeed` in the `HashRegistrar`. If they want to avoid potentially losing their domain in the process, they can transfer the deed to a smart contract which can then release the deed and rent the same name in the new registrar atomically.

#### Remediation

Here are a few ideas of improvements to help in this situation:

1. Discourage (or prevent, if possible) new auctions very close to the launch of the new registrar.
2. Allow domains to be transferred before the 183-day waiting period but require rent payment in those cases. (Perhaps just use the existing grace period to have people renew?)
3. Document the process for rescuing names that get stuck in this state, or better yet provide a tool for doing so.

### 3.9 `BaseRegistrarImplementation.acceptRegistrarTransfer` should probably use the `live` modifier

| Severity  | Status |  Remediation Comment |
| ------------- | ------------- | ------------- |
|  Medium  |  Closed   |  Issue has been closed in [ensdomains/ethregistrar#19](https://github.com/ensdomains/ethregistrar/pull/19). |


#### Description

Most `external` functions in `BaseRegistrarImplementation` have the `live` modifier, which ensures that they can only be called on the current ENS owner of the registrar's base address. The `acceptRegistrarTransfer` function does _not_ have this modifier, which means names can be transferred to the new registrar even if it's not the proper registry owner.

It's hard to think of a real-world example of why this is problematic, especially because the interim registrar appears to protect against this by only transferring to the `ens.owner`, but it seems safer to include the `live` modifier unless there's a specific reason not to.

#### Remediation

Add the `live` modifier to `acceptRegistrarTransfer`.

### 3.10 Reconsider use of inline assembly in `BytesUtils.sol`

| Severity  | Status |  Remediation Comment |
| ------------- | ------------- | ------------- |
|  Minor  |  Closed   |  Issue has been closed in [ensdomains/dnssec-oracle#55](https://github.com/ensdomains/dnssec-oracle/pull/55) |


#### Description

`Root.sol` imports and uses `@ensdomains/dnssec-oracle/contracts/BytesUtils.sol` for byte operations.

`BytesUtils.sol` is mainly written in assembly. In general, inline assembly is concerning from a security perspective because it bypasses compiler checks and inhibits human code reasoning.

e.g.`readUint8()`:
```solidity
function readUint8(bytes memory self, uint idx) internal pure returns (uint8 ret) {
    require(idx + 1 <= self.length);
    assembly {
        ret := and(mload(add(add(self, 1), idx)), 0xFF)
    }
}
```

#### Remediation

Some of the functions in `BytesUtil.sol` can be written in Solidity without affecting the gas costs.

`readUint8()` can be written as following Solidity code which functions the same:

```solidity
function readUint8(bytes memory self, uint idx) internal pure returns (uint8 ret) {
    return uint8(self[idx]);
}
```

### 3.11 `BaseRegistrarImplementation.acceptRegistrarTransfer` does not check for invalid names

| Severity  | Status |  Remediation Comment |
| ------------- | ------------- | ------------- |
|  Minor  |  Closed   |  Short names will be manually canceled in the old registrar during the migration period. Note that this is still feasible with the reduced 28-day lock-up period. |


#### Description

`BaseRegistrarImplementation.acceptRegistrarTransfer` does not explicitly check for invalid names.

In the old registrar it is possible to register domain names with length less than 7 characters. However anyone can call `HashRegistrar.invalidateName()` to invalidate the registration and get half of the deed amount as an incentive.

Assume that an invalid domain is registered in the old registrar and no one invalidates the registration (within the 183 days between the `registrationDate` and the transfer `ETHRegistrarController.acceptRegistrarTransfer`), it is possible to transfer the invalid domain to the new ENS registrar.

#### Remediation

Given that it is easy to check for invalid domains using a rainbow table for all possible <7 character domains, anyone can invalidate them before the new registrar goes live. Note that for the auctions starting right before the new registrar goes live, there will be a 183 days window in which anyone can call `HashRegistrar.invalidateName()` to invalidate the domain names.


### 3.12 Sanity check around `transferPeriodEnds`

| Severity  | Status |  Remediation Comment |
| ------------- | ------------- | ------------- |
|  Minor  |  Closed   |  Issue has been closed in [ensdomains/ethregistrar#23](https://github.com/ensdomains/ethregistrar/pull/23) |


#### Description

`BaseRegistrarImplementation.acceptRegistrarTransfer` has a hardcoded limit such that only domains registered 183 days ago can be transferred in.

This imposes an implicit constraint on the `transferPeriodEnds` state variable. If the transfer period ends too soon after the new registrar is put in place, names that were just registered won't be transferrable during the transfer period (and will thus become available to be rented by another user).

#### Remediation

A sanity check in the constructor would help here, e.g.:

```solidity
require(_transferPeriodEnds > now + 183 days);
```

Note that the true requirement is something more like "The time between when this registrar becomes the ENS node owner of the .eth domain and the time of `transferPeriodEnds` must be at least 183 days plus a sufficient time window for late registrants to have a chance to perform the transfer." But it's hard to see a way to encode this precisely. A broad sanity check will at least avoid simple timing mistakes.

### 3.13 `StablePriceOracle.price` has an unimportant integer underflow

| Severity  | Status |  Remediation Comment |
| ------------- | ------------- | ------------- |
|  Minor  |  Closed   |  Issue has been closed in [ensdomains/ethregistrar#20](https://github.com/ensdomains/ethregistrar/pull/20/) |


#### Description



**ethregistrar/contracts/StablePriceOracle.sol:L57-L63**

```Solidity
    function price(string calldata name, uint /*expires*/, uint duration) view external returns(uint) {
        uint len = name.strlen();
        require(len > 0);
        if(len > rentPrices.length) {
            len = rentPrices.length;
        }
        uint priceUSD = rentPrices[len - 1].mul(duration);
```



If the length of the `rentPrices` array is 0, then the last line above attempts to access `rentPrices[2**256-1]`. This will `assert`, but it might be more friendly (from a gas perspective) to `revert` in this case.

#### Remediation

A simple fix would be to move the `require(len > 0)` down until just before the array access.

### 3.14 `ETHRegistrarController.register` should  `revert` rather than silently fail

| Severity  | Status |  Remediation Comment |
| ------------- | ------------- | ------------- |
|  Minor  |  Closed   |  Issue has been closed in [ensdomains/ethregistrar#22](https://github.com/ensdomains/ethregistrar/pull/22/) |


#### Description

When called with an invalid commitment or unavailable domain, `ETHRegistrarController.register` refunds the sent ether and silently fails rather than `revert`ing:



**ethregistrar/contracts/ETHRegistrarController.sol:L56-L64**

```Solidity
    function register(string calldata name, address owner, uint duration, bytes32 secret) external payable {
        // Require a valid commitment
        bytes32 commitment = makeCommitment(name, secret);
        require(commitments[commitment] + MIN_COMMITMENT_AGE <= now);

        // If the commitment is too old, or the name is registered, stop
        if(commitments[commitment] + MAX_COMMITMENT_AGE < now || !available(name))  {
            msg.sender.transfer(msg.value);
            return;
```



`register` also has no return value, so it's difficult for a caller to know whether the `register` action succeeded or failed.

#### Remediation

It's probably better to use `require(...)` to handle these invalid cases. This is roughly equivalent because no state changes have been made before this early `return`, but it seems less error prone and clearer to callers about what happened.

### 3.15 `StringUtils.strlen` could be rewritten without assembly

| Severity  | Status |  Remediation Comment |
| ------------- | ------------- | ------------- |
|  Minor  |  Closed   |  Issue has been closed in [ensdomains/ethregistrar#21](https://github.com/ensdomains/ethregistrar/pull/21) |


#### Description

`StringUtils.strlen` uses inline assembly to walk through a UTF-8 string and count its character length. In general, inline assembly is concerning from a security perspective because it bypasses compiler checks and inhibits human code reasoning.

#### Remediation

Consider rewriting in Solidity, something similar to the following:

```solidity
function strlen(string memory s) internal pure returns (uint256) {
    uint256 i = 0;
    uint256 len;
    for (len = 0; i < bytes(s).length; len++) {
        byte b = bytes(s)[i];
        if (b < 0x80) {
            i += 1;
        } else if (b < 0xE0) {
            i += 2;
        }
        ...
    }
    
    return len;
}
```



## 4 Threat Model

The creation of a threat model is beneficial when building smart contract systems as it helps to understand the potential security threats, assess risk, and identify appropriate mitigation strategies. This is especially useful during the design and development of a contract system as it allows to create a more resilient design which is more difficult to change post-development.

A threat model was created during the audit process in order to analyze the attack surface of the contract system and to focus review and testing efforts on key areas that a malicious actor would likely also attack. It consists of two parts: a high-level analysis that help to understand the attack surface and a list of threats that exist for the contract system.

### 4.1 Overview

The following assets are managed by contracts and likely targets for an attacker:

* Registered domain names (e.g. `foo.eth`)
* Ether, in the form of rent paid to the `ETHRegistrarController`

The following actors have access to the system to perform an attack:

* System owners (ENS itself, registrars, controllers, price oracles)
* DNS domain/subdomain owners, who can update DNSSEC records
* Users who are registering, renewing, and transferring domains

The following describes the surface area available to attackers:

* DNSSEC records
* Registrars and controllers
* `Root` contract
* Ethereum private keys

Because they were out of scope for this audit, we did _not_ consider some interesting targets such as the DNSSEC oracle, DNSSEC-based registrar, the interim `.eth` registrar, or the multisig wallet used for ENS ownership.

### 4.2 Threat Analysis

The following table contains a list of identified threats, along with their mitigations:

|  Threat | Attack Strategy | Mitigation |
|-------------|-------------|-------------|
| user may try to register/renew a domain for less than the expected price | overflow on the rent price / manipulate the price oracle | SafeMath mitigates some potential math errors |
| user may try to mount denial-of-service attacks on other users (e.g. censor their purchases/renewals) | network DoS | long purchase windows and grace periods |
| user may try to snipe a domain | front-running | a commit/reveal scheme attempts to prevent this but is ineffective (see section 3), a generous grace period prevents race conditions on expiration |
| user may try to register `.eth` TLD | update DNSSEC records | `Root` disallows changes to that node |
| `Root` owner may steal domains, manipulate prices, etc. | ENS root swaps the controller/registrar with malicious code | such manipulation would be transparent today, and future updates may limit the root owners' powers |
| domain owners may take over already-owned subdomains | change DNSSEC to replace registrar for a domain | this is allowed by design |

## 5 Tool based analysis

The issues from the tool based analysis have been reviewed and the relevant issues have been listed in chapter 3 - Issues.

### 5.1 MythX

<img height="120px" align="right" src="static-content/mythril.png"/>

MythX is a security analysis API for Ethereum smart contracts. It performs multiple types of analysis, including fuzzing and symbolic execution, to detect many common vulnerability types. The tool was used for automated vulnerability discovery for all audited contracts and libraries. More details on MythX can be found at [mythx.io](https://mythx.io).

Where possible, we ran the full MythX analysis. MythX is still in beta, and where analysis failed, we fell back to running Mythril Classic, a large subset of the functionality of MythX.

The raw output of MythX and Mythril Classic vulnerability scans can be found [here](./tool-output/mythx/mythx_report.md).

### 5.2 Ethlint

<img height="120px" align="right" src="static-content/ethlint.png"/>

[Ethlint](https://www.ethlint.com/) is an open source project for linting Solidity code. Only security-related issues were reviewed by the audit team.

The raw output of the Ethlint vulnerability scan can be found [here](./tool-output/ethlint/ethlint_report.md).

### 5.3 Surya
Surya is an utility tool for smart contract systems. It provides a number of visual outputs and information about structure of smart contracts. It also supports querying the function call graph in multiple ways to aid in the manual inspection and control flow analysis of contracts.

A complete list of functions with their visibility and modifiers can be found [here](./tool-output/surya/surya_report.md).

### 5.4 Odyssey

<img height="120px" align="right" src="static-content/odyssey.png"/>

Odyssey is an audit tool that acts as the glue between developers, auditors and tools. It leverages GitHub as the platform for building software and aligns to the approach that quality needs to be addressed as early as possible in the development life cycle and small iterative security activities spread out through development help to produce a more secure smart contract system.
In its current version Odyssey helps to better communicate audit issues to development teams and to successfully close them.

## 6 Test Coverage Measurement

Testing is implemented using Truffle. 12 tests are included for the `Root` contract, and they all pass. 30 tests are included for the `.eth` permanent registrar, and they all pass.

We were unable to obtain code coverage numbers for the tests, but the audit team's overall impression is that testing covers a high percentage of code branches. That said, the testing is weak, in particular regarding negative test cases and edge cases. As a specific example, changing the following in `ETHRegistrarController.renew` causes no test failures, which shows a serious lack of coverage:

```solidity
// OLD: require(msg.value >= cost);
// NEW:
require(msg.value > 0);
```

## Appendix 1 - File Hashes

The SHA1 hashes of the source code files in scope of the audit are listed in the table below.

|  File Name  |  SHA-1 Hash  |
|-------------|--------------|
| root/contracts/Ownable.sol | b596da7ad9b5c92a119268e05a5f2190659de8d3 |
| root/contracts/Root.sol | 94c5fd45635c6d78cae15903bdf565e2c587bdfa |
| ethregistrar/contracts/BaseRegistrar.sol | dfadfc8a35024069ff66cbc4a82b67dc48129eab |
| ethregistrar/contracts/BaseRegistrarImplementation.sol | a1e04ce66a9588063155591b59cd695d2d35cabe |
| ethregistrar/contracts/ETHRegistrarController.sol | 7cb180a1d5102efd2acc04b0b518848b6127846e |
| ethregistrar/contracts/PriceOracle.sol | 3257acda730f294f19984163f9fe4a19eabdef4d |
| ethregistrar/contracts/SafeMath.sol | 5effc6db2209b2bf2d49abe4ad1ac247e106f8d9 |
| ethregistrar/contracts/SimplePriceOracle.sol | fc11bff8c93e8471b8d8478f1a14b7f43fff2eef |
| ethregistrar/contracts/StablePriceOracle.sol | 892333542a757ba6089c5c3d19d00b337cb0da78 |
| ethregistrar/contracts/StringUtils.sol | 4d784bb26b409cfd8ed841f43c4e0ffbfddc450b |

## Appendix 2 - Severity

### A.2.1 - Minor

Minor issues are generally subjective in nature, or potentially deal with topics like "best practices" or "readability".  Minor issues in general will not indicate an actual problem or bug in code.

The maintainers should use their own judgment as to whether addressing these issues improves the codebase.

### A.2.2 - Medium

Medium issues are generally objective in nature but do not represent actual bugs or security problems.

These issues should be addressed unless there is a clear reason not to.

### A.2.3 - Major

Major issues will be things like bugs or security vulnerabilities.  These issues may not be directly exploitable, or may require a certain condition to arise in order to be exploited.

Left unaddressed these issues are highly likely to cause problems with the operation of the contract or lead to a situation which allows the system to be exploited in some way.

### A.2.4 - Critical

Critical issues are directly exploitable bugs or security vulnerabilities.

Left unaddressed these issues are highly likely or guaranteed to cause major problems or potentially a full failure in the operations of the contract.

## Appendix 3 - Disclosure

ConsenSys Diligence (“CD”) typically receives compensation from one or more clients (the “Clients”) for performing the analysis contained in these reports (the “Reports”). The Reports may be distributed through other means, including via ConsenSys publications and other distributions.

The Reports are not an endorsement or indictment of any particular project or team, and the Reports do not guarantee the security of any particular project. This Report does not consider, and should not be interpreted as considering or having any bearing on, the potential economics of a token, token sale or any other product, service or other asset. Cryptographic tokens are emergent technologies and carry with them high levels of technical risk and uncertainty. No Report provides any warranty or representation to any Third-Party in any respect, including regarding the bugfree nature of code, the business model or proprietors of any such business model, and the legal compliance of any such business. No third party should rely on the Reports in any way, including for the purpose of making any decisions to buy or sell any token, product, service or other asset. Specifically, for the avoidance of doubt, this Report does not constitute investment advice, is not intended to be relied upon as investment advice, is not an endorsement of this project or team, and it is not a guarantee as to the absolute security of the project. CD owes no duty to any Third-Party by virtue of publishing these Reports.

PURPOSE OF REPORTS The Reports and the analysis described therein are created solely for Clients and published with their consent. The scope of our review is limited to a review of Solidity code and only the Solidity code we note as being within the scope of our review within this report. The Solidity language itself remains under development and is subject to unknown risks and flaws. The review does not extend to the compiler layer, or any other areas beyond Solidity that could present security risks. Cryptographic tokens are emergent technologies and carry with them high levels of technical risk and uncertainty.

CD makes the Reports available to parties other than the Clients (i.e., “third parties”) -- on its GitHub account (https://github.com/ConsenSys). CD hopes that by making these analyses publicly available, it can help the blockchain ecosystem develop technical best practices in this rapidly evolving area of innovation.

LINKS TO OTHER WEB SITES FROM THIS WEB SITE You may, through hypertext or other computer links, gain access to web sites operated by persons other than ConsenSys and CD. Such hyperlinks are provided for your reference and convenience only, and are the exclusive responsibility of such web sites' owners. You agree that ConsenSys and CD are not responsible for the content or operation of such Web sites, and that ConsenSys and CD shall have no liability to you or any other person or entity for the use of third party Web sites. Except as described below, a hyperlink from this web Site to another web site does not imply or mean that ConsenSys and CD endorses the content on that Web site or the operator or operations of that site. You are solely responsible for determining the extent to which you may use any content at any other web sites to which you link from the Reports. ConsenSys and CD assumes no responsibility for the use of third party software on the Web Site and shall have no liability whatsoever to any person or entity for the accuracy or completeness of any outcome generated by such software.

TIMELINESS OF CONTENT The content contained in the Reports is current as of the date appearing on the Report and is subject to change without notice. Unless indicated otherwise, by ConsenSys and CD.

