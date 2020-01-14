# Aragon Chain Specification

### Created by ChainSafe Systems
Authors: _Gregory Markou and Colin Schwarz_
Editor: _Colin Schwarz_

## Contents

- [**Abstract**](#Abstract)
- [**Introduction**](#Introduction)
  - [What is Aragon Chain?](#What-is-Aragon-Chain)
  - [Migration](##Migration)
  - [Ethermint Compatibility](#Ethermint-Compatibility)
  - [Known Issues](###Known-Issues)

- [**Configurations**](#Configurations)
  - [Validators](###Validators)
  - [Staking](#Staking)
  - [EVM](#EVM)
  - [Account Versioning](###Account-Versioning)
  - [Opcode Repricing](###Opcode-Repricing)

- [**Ethereum Bridge**](#Ethereum-Bridge)
  - [Technical Details](##Technical-Details)
  - [Chain Identification](##Chain-Identification)
  - [Token Transfers](##Token-Transfers)
  - [Core](##Core)
  - [Transaction Poster](##Transaction-Poster)
  - [Bridge Development](##Bridge-Development)

- [**Test Results**](#Test-Results)

- [**Future Features**](#Future-Features)
  - [Multi-Signature Accounts](#Multi-Signature-Accounts)
  - [Precompiles](##Precompiles)


# **Abstract**

This specification outlines the steps and considerations that must be taken to build Aragon Chain and to bridge it to Ethereum. It begins with a definition of Aragon Chain and moves on to outline the migrion of Aragon contracts and web portal from Ethereum to Ethermint. The paper then explores the known issues with Ethermint, as they relate to Aragon Chain, and proposes solutions where appropriate. Next, the document offers an in depth examination of various considerations for configuring Aragon Chain. The final section offers a detailed specification for the bridge that will allow value and information to be transferred both ways between Aragon on Ethereum and Aragon Chain. Results from preliminary migration tests are detailed at the end of this paper, but a full analysis of these tests is out of scope and will be addressed in a separate document after some necessary modifications are made to Ethermint.

# **Introduction**

This document is a specification for Aragon Chain, a custom Cosmos Zone built on Ethermint to create an Aragon ecosystem outside Ethereum. The document represents a comprehensive assessment of the process of migrating contracts and other features from Aragon on Ethereum to Aragon Chain and the possible risks associated with this migration. It also outlines a specification for a bridge between Aragon on Ethereum and Aragon Chain.

## **What is Aragon Chain?**

Aragon Chain is a custom Cosmos zone built on Ethermint and optimized for Aragon usage. Aragon Chain will be built by implementing the existing Aragon smart contracts and web portal on Ethermint. Aragon Chain will consist of two primary components: a migration of Aragon contracts and web portal from Ethereum to Ethermint, and a bridge that facilitates the transfer of value and information between Aragon on Ethereum and Aragon Chain. This means that Aragon Chain will provide the same functionality to which users are accustomed on mainnet Ethereum, with the added bonus of lower fees and faster blocktimes and transactions. Aragon Chain will also allow for a much higher level of customization than Aragon on Ethereum and can be optimized for Aragon use because this is the only use case the network is beholden to. This will allow for a greater amount of control and customization when undertaking future Aragon initiatives and development. An exciting area of customization is the ability to interact with Comos SDK modules in conjunction with the EVM, since the EVM module is simply a step in the whole process. [1] This means that additional features can be written (in golang) during the Ethermint state transition function.

1. Discussed below.

## Migration

The process of migrating Aragon to Aragon Chain does not imply the abandonment of Ethereum. Rather, the purpose of Aragon Chain will be to experiment with functionalities that simply do not exist on Ethereum. For the migration to be successful the existing infrastructure must not change. This means that the current set of smart contracts, deployment tools, and development tools must operate as they do on Ethereum. Since Ethermint is both EVM and Ethereum API (JSON-RPC) compatible, the existing tooling should work as expected. [2] Secondly, the existing dashboards, GUI&#39;s, and DApps will not require drastic changes to migrate. Once again, because Ethermint is Ethereum API compatible, the DApps will be deployable with little to no additional overhead. For the remainder of this paper, the tools mentioned above will be referred to as &quot;Aragon Tooling&quot;.

One interacts with the Ethereum blockchain via a provider library such as ethers.js or web3.js. These packages contain easy to use interfaces that surround the Ethereum API and allow any DApp to easily interact with the blockchain. Aragon predominantly uses the web3.js package, when instantiating a connection to a node. This means the only variable that would need to change during the migration would be the url for a given provider:
```javascript=
var web3 = new Web3("https://mainnet.infura.io");
```
OR
```javascript=
var web3 = new Web3("https://yourEthermintNode.io");
```
Since Ethermint uses the same Ethereum API, transitioning Aragon Tooling will require nothing more than providing the URL to a valid Ethermint node that exposes the Ethereum API server. This means there is little additional overhead when migrating developer tooling and webapps from Ethereum to Ethermint.

Finally, private keys from Ethereum can also be migrated with little additional overhead. The default private key curve for Cosmos is SECP. This means Ethermint also supports SECP private keys, the same ones used in Ethereum. Therefore, raw private keys from Ethereum, MetaMask, hardware wallets, and ultimately any private key management tool, will work out of the box on Ethermint. Alternatively, if a user chooses to manage keys themselves, Ethermint has a key generation tool in the CLI. There is one caveat: if a user chooses to generate their own key from a mnemonic, they must supply a 24 word mnemonic since 12 word menomic seed phrases are not supported. [3]

2. There are some Caveats to this that will be addressed below.
3. See [here](https://github.com/cosmos/cosmos-sdk/issues/4278) for open issue discussing the problem.

## **Ethermint Compatibility**

### **Known Issues**

Although the compatibility between Ethermint and Ethereum should prevent any serious problems when porting Aragon contracts from Ethereum, there are still some minor known issues that could impact the migration and will be addressed here. Most of these issues are related to either gas prices or dev tooling although a few do not fall under either category. Statically defined gas prices pose a problem, although the latest release of the Aragon contracts removed the majority of gas assumptions. [4] However, this issue can be overridden with a feature to configure the EVM in Ethermint. There are two possible ways to do this. The first is to override the gas costs for instructions by using a fork of Geth and forking Ethermint. In this case, the values for opcode gas pricing would have to be adjusted to fit the new context. If one does not wish to fork Ethermint, the other solution is to fork Geth to make the jump table public and implement the ability to set gas costs on Ethermint. Another issue surrounding gas is that gas refunds don&#39;t occur in Ethermint, although they do in Ethereum. This means that trying to match gas costs between Cosmos and Ethereum will not be a simple undertaking. The difference causing this issue is that Cosmos pays their validators in the Ante handler and Ethereum handles the cost and refund all within the state transition. Although matching gas costs between Ethereum and Ethermint is not a concern in general, getting close to a match will be important in the Aragon context. If gas costs between the two networks are not exactly equal, all that needs to be done to maintain parity between the networks is to implement the gas refunds for validators at the end of the state transition. The Cosmos team has confirmed that this is possible. This technique will not work if the gas costs are too far apart so it will be important to keep them as close together as possible.

In addition to issues around gas prices, there are a few Ethereum API methods that are unimplemented in Ethermint: filters and ``eth_gasPrice``. Additionally, ``eth_getLogs`` has been implemented but has not yet been heavily tested. An analysis of the Aragon code base indicates that filters are relied upon heavily. These filters proved to be a major blocker during initial tests deploying Aragon contracts in Ethermint. The Ethermint team is currently implementing filters to remedy this issue.

Another known issue involves libraries, such as Ethers.js, although the Aragon stack does not use it. In its current state, Ethers.js returns an error when submitting transactions as well as a few other methods, even though the transaction is successfully included in a block. This limitation is caused by sanity checks from the Ethers.js codebase. When a transaction is submitted, Ethers.js ensures the returned transaction hash was RLP encoded by comparing it against the transaction object that was sent. The hashes will never match because Ethermint uses Amino encoding causing validation to fail. There are three solutions to this issue: fork Ethers.js and remove the sanity checks, add arbitrary encoding support to Ethers.js, or migrate Ethermint away from Amino encoding. We suggest forking Ethers.js, removing the sanity checks. This would require less work than forking Ethermint to use RLP, and the current structure of how Ethers.js handles encoding would require a fairly large refactor to the code base. [5]

Another known issue is that unlike on the Ethereum mainnet, pending state is not available within Ethermint. This is because in Tendermint, the state transition happens at the end of the block whereas Ethereum keeps a pending block and applies the transactions as they arrive. This issue is fairly trivial since pending state should not be trusted for any serious use cases, but it is worth mentioning as some dev tooling uses it.

Finally, WebSockets are currently not supported on Ethermint, but the existing JSON-RPC server can be upgraded to support WebSockets.

4. [Delegate proxy](https://github.com/aragon/aragonOS/blob/master/contracts/common/DelegateProxy.sol) makes an assumption on whether or not an interaction is a deposit. Here are the [relevant tests](https://github.com/aragon/aragonOS/blob/next/test/contracts/common/depositable_delegate_proxy.js#L14) outlining the 10k gas call. 
5. Conversations on the possibilities of supporting arbitrary encoding are underway with ethers.js team.

# Configurations

### Validators

Under the hood, Ethermint uses Tendermint Core which comprises both a consensus protocol, and a networking protocol. The consensus protocol, Tendermint BFT, is a proof of stake (PoS) protocol which means that a set of validators must exist in order to secure the chain. Aragon Chain will need to secure its own validators. There are many validator pools and companies actively partaking in the Cosmos ecosystem which could be sourced for this purpose. In a PoS system, the number of validators has an impact on a network&#39;s security and level of decentralization. It is difficult to determine the perfect number of validators to introduce at genesis. Previous chains have successfully launched on Cosmos with around 20 to 30 validators, a fairly low number. We recommend striving for at least 60 validators at the genesis of Aragon Chain.

### Staking

Staking on Aragon Chain will be done via a new token, ARA. Prospective validators will be able to obtain ARA through a lock and hold mechanism over the bridge. [6] The distribution mechanism for ARA will be very similar to Aragon Court's ANJ in so far as ARA can be minted and burned by depositing or withdrawing ANT in an Aragon Fundraising bonding curve. [7] The ratio and logistics of converting ANT into ARA is not yet finalized and so will not be discussed here. A subsequent report will cover the details of this conversion.

6. More on this below.
7. See [here](https://blog.aragon.one/aragon-chain/).
## EVM

Because Ethermint is an EVM based chain there will be a lot of similarities between Aragon on Ethereum and Aragon Chain in terms of the virtual machine. This leaves much room for creative design, both functional and experimental. It will allow for the manipulation of existing Ethereum mainnet protocol features and parameters  and to take on more ambitious EIPs that were previously unrealistic. 

### Account Versioning

Generalized account versioning is a unique feat accomplished through EIP-1702. [8] The goal is to create a system in which hardforks can be safely applied without the risk of breaking existing contracts and their functionality. [9] On Ethereum, EIP-1702 accomplishes this by allowing for multiple different versions of the EVM to be executed during the same state transition. An additional field is added to accounts named &quot;version&quot;. This &quot;version&quot; denotes which EVM the contract should be executed against. Enabling this functionality will allow Aragon Chain to experiment with the EVM, perform upgrades, and not risk having a similar situation that occurred with EIP-1884. [10]

8. See [here](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-1702.md).
9. See [here](https://ethereum.corepaper.org/compatibility/) for information on backwards compatibility and account versioning.
10. More on this below.
### Opcode Repricing

The most recent Ethereum hardfork, Istanbul, introduced EIP-1884. [11] EIP 1884 is intended for: "Repricing for trie-size dependent opcodes." [12] This serves to rebalance underpriced opcodes. Unfortunately, this EIP will cause over 2,500 Aragon smart contracts to break due to fixed gas costs when making internal calls across the contracts. Aragon Chain can safely use this, because the most recent Aragon contract release, removed those concerns. Situations such as these would not arise in the future should account versioning be implemented, as this allows the EVM to be safely upgraded.

11. See [here](https://eips.ethereum.org/EIPS/eip-1679) for Istanbul.
12. See [here](https://eips.ethereum.org/EIPS/eip-1884) for EIP 1884.

# **Ethereum Bridge**

There are several reasons for bridging data between Ethereum and Aragon Chain. The most important is the need to utilize ANT on Aragon Chain and ARA to stake. [13] Aragon Chain has a few options for transferring data between itself and other blockchains. The Inter-Blockchain Communication protocol (IBC) is one means by which this could be achieved. IBC is under heavy research and development by the Tendermint team. IBC aims to create a common format for blockchains to communicate with one another by utilizing light clients. Unfortunately there are two things holding this back: IBC is not yet complete, and Eth1.x does not have a stable enough light client ecosystem. In lieu of these missing pieces, Aragon Chain will utilize a bridge.

A bridge is a piece of software that interfaces between two or more blockchains, facilitating the transfer of data amongst them. Bridges interact with _bridge contracts_ (deployed on the respective blockchains) that the bridge software monitors for _deposit events_ which indicate data is ready to be transferred. Bridges can be trusted or trustless. Trusted bridges typically have a set of known validators. These validators are initially voted in by those who originally deployed the contracts. After, they are voted in and out by the current validators. Trustless bridges typically require complex merkle proofs and on chain validation of another chain&#39;s block headers to ensure the security of a transferred asset.

Aragon Chain will utilize a trusted bridge to facilitate the transfer of tokens and to bestow the ability to control the Aragon Agent between Ethereum and Aragon Chain. Currently, an Aragon Agent is an Ethereum account owned by an Aragon organization. [14] Aragon Agents function as high utility multi-signature accounts and enable Aragon organizations to interact with Ethereum contracts or protocols. Aragon Agents also enable communication between different Aragon organizations and enable various other functionalities such as managing names in ENS, trading tokens on 0x or Uniswap, and opening a Maker CDP. Aragon Agent contracts also play a unique role with the bridge. Although the initial intention is to control the Agent, the bridge will allow for full control of any DAO from either chain, should the user have the adequate permissions. DAO&#39;s can fully operate on either chain. When they need to perform a cross-chain action, they can delegate a vote to their Aragon Agent to interface with the bridge. Another example of using the bridge is for a user to place a vote on Aragon Chain from the Ethereum Mainnet. If one were using a mobile wallet that has a built-in browser to visit Dapps, that person could visit the bridge website and cast the vote through the bridge. This functionality will be extremely useful for mobile wallets that do not allow a user to specify a custom URL to a node. This bridge will offer invaluable functionality for the greater Aragon ecosystem, especially during the early days of Aragon Chain, by creating a comfortable stepping stone to and from the new chain.

13. ARA is derived through a curve based on the total amount of ANT. See [here](https://blog.aragon.one/aragon-chain/) for more.
14. See [here](https://blog.aragon.one/aragon-agent-beta-release/) for Aragon Agent.
### Technical Details

The bridge will consist of 3 main components: the ``Listener``, which listens to the bridge events on a source chain; the ``Core``, which will perform validation of the bridge event as well as network message passing; and the ``Transaction Poster``, which will post the respective transaction to the destination chain. Together, these components will allow a user to transfer assets and information from one chain to another, by locking tokens on the chain of origin and unlocking them on the destination chain.

The first version of the bridge will allow bridging of transactions that have been signed by n-of-m bridge validators. Authorities will be whitelisted to relay on chain A after being added to the bridge contract on chain A. Contract addresses will be configurable by bridge nodes to allow for easy upgrading.

### Chain Identification

Since the bridge will be acting between multiple chains, we need a way to identify the chain to which we want to bridge a transaction. The bridge nodes will have a known configuration that will map a chain ID/Key to the chain configuration (full node address, endpoints, etc). This means users will not need to know any information about the destination chain apart from its ID.

### Token Transfers

A token transfer is the operation of moving a token from one chain to another. To enable token transfers, a similar concept to the bridge adapters will be used. The bridge contract is able to accept any type of ERC standard token, using ERC receivers. These receivers implement a generic function that, under the hood, performs all additional logic required to transfer that specific token standard.

### **Core**

The core module performs validation of bridge transfer requests. To validate a transfer request, the bridge needs to confirm that the value that was locked by the user is sufficient to perform the transaction on the destination chain. The destination transaction may be arbitrary data, so it may perform a value transfer from the bridge contract to the user.

The core module needs to be able to decode the transaction data on the destination chain. This can be accomplished by an adapter interface. Every chain that wishes to be connected to the bridge needs to implement an adapter interface.

### **Transaction Poster**

The transaction poster will post transactions to a chain. It needs to be connected to a destination chain node or have some other method of interacting with the chain. It also needs a valid account on this chain with some value in it (for gas fees).

### **Bridge Development**

The bridge to be used by Aragon Chain is currently under active development by ChainSafe Systems. A proof-of-concept (PoC) should be available in January and will allow developers to experiment with cross-chain functionalities. The repository is currently private but will be open-sourced once the first PoC is ready. The bridge will be continually maintenance by ChainSafe Systems. Details on the specification and advanced functionality will also be released once the repository is made public.

# **Test Results**

Preliminary tests were conducted by ChainSafe. We saw that every transaction that was executed successfully made it into Ethermint, meaning the EVM module operated as expected. However, we found that we need to make a modification to Ethermint in order for all of the migration tests to run successfully. [15] The issue arrose when the deploy scripts waited for certain events to fire from the contracts. This caused the deploy script to seemingly fail because filters are not fully supported on the Ethermint JSON-RPC shim. Consequently, the first version of this document will not include a detailed section on testing but will be augmented by a supplementary document which outlines the testing and results and will be released in the future.

15. ChainSafe had to disable a lot of the deploy scripts and hardcode some values.


# Future Features
### Multi-Signature Accounts

The RSK team recently created a RSK improvement proposal (RSKIP) to support multi-key accounts. [16] The proposal makes multi-signature contracts "first class citizens". Aragon Chain will implement a similar approach and have multi-signature contracts baked into the protocol. This will allow for a plethora of optimizations on existing Aragon contracts, for instance many of the Aragon contracts, including Aragon Agents, utilise multi-signatures. Alternatively, the Cosmos keybase, apart of the Cosmos SDK, now supports multi-signatures, and could offer similar benefits to the above RSKIP.
16. See [here](https://github.com/rsksmart/RSKIPs/blob/master/IPs/RSKIP138.md).

### Precompiles

We can try to optimize Aragon Chain through precompiles for any operations or functions that are computationally expensive. For Aragon on the Ethereum mainnet, any precompiles introduced have to work for the wider network. However, for Aragon Chain, there are a lot of exciting optimizations that we can do with precompiles because they can be bespoke to the Aragon use case. For this use case, precompiles are preferable to opcodes because they allow one to maintain compatibility with existing tooling frameworks such as the Solidity compiler. [17] Therefore, they do not require any special updates to existing infrastructure. A possible optimization would be to create a &quot;merklelize&quot; precompile that allows one to create, update, and prove a Merkle tree. The minime contract  has the concept of checkpointing for balances so the total storage size and cost of maintaining the checkpoints could be reduced by leveraging a Merkle tree. [18] Finally, Aragon has a suite of apps that aid in day to day operations of a DAO, most notably the accounting. [19] There could be unique use cases to leverage advanced on-chain functionality in a cost effective manner to help with accounting or payroll. Math operations can become very expensive, or in some cases limiting, due to Solidity&#39;s language constraints. Building precompiles to meet these use cases could become extremely helpful.

Another alternative, mentioned above, is creating additional logic in the Ethermint state transition, external of the EVM module. This can be done under the condition that these additional transitions do not update any EVM state.  For example, a vote can occur on-chain (via a smart contract) and the values could be fed into the Cosmos governance module by reading the state of the contract after the EVM has finished processing.

17. Precompiles live at an address, and are treated like a smart contract so they do not require a change in the actual EVM.
18. See [here](https://github.com/aragon/minime/blob/master/contracts/MiniMeToken.sol#L69) for more on minime contract.
19. Apps are bundles of contracts performing one activity such as finance, payroll and voting. See [here](https://github.com/aragon/aragon-apps) for the Aragon apps repository on Github.
