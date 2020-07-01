<h1 align="center">Joining the Cross Chain Ecosystem</h1>

English | [中文](README_CN.md)

## Overview

Poly cross chain ecosystem provides a platform for various different chains to interact and transfer data along with carrying out cross chain transactions. Any chain can freely join the ecosystem. However, the chains that support smart contracts can interact and transfer all kinds of information between chains, while the chains that do not support smart contracts are limited to cross chain asset transfer. There are three factors that determine whether a blockchain can join the Poly cross chain ecosystem: 

- The system implements multiple interfaces either using smart contracts, or as core API that can be used to interact with the chain
- The chain needs to provide its block header format, the serialization and deserialization methods used, and the mode of signature verification in order to allow the relay chain to be able to process and verify block headers.
- The chain also needs to provide its merkle tree generation and verification method for the ledger records or transactions. This allows the relay chain to determine the legitimacy of the transaction records.

## Interface Requirements

To implement cross chain features for any chain, say Ethereum, there are two kinds of contracts that need to be deployed-
1. Block header synchronization contract: This contract maintains the record of block headers of the relay chain on this chain. These block headers serve as means to verify cross chain transactions.
2. Cross chain management contract: Every chain can have no more than one management contract. It creates the cross chain transactions that are transferred to the relay chain. All the service contracts that contain the business logic need to communicate with the management contract.

The interface methods that need to implemented by the respective contracts are as follows:

### Block Header Synchronization Contract

| Interface Method      | Description                                                  |
| --------------------- | ------------------------------------------------------------ |
| **SyncGenesisHeader** | Synchronizes the relay chain's genesis block header (or another block header where a change in block generation cycle occurred), method is invoked one time only when the contract is initialized, stores and processes the genesis block header, fetches the consensus node info of the relay chain, please refer to the [code](https://github.com/ontio/ontology/tree/master/smartcontract/service/native/cross_chain/header_sync) for more details |
| **SyncBlockHeader**   | Consistently synchronizes block cycle change and cross chain transaction block headers from the relay chain, relayer uses this interface method to synchronize block headers, stores and processes block headers, fetches the consensus node info if block generation cycle changes, please refer to the [code](https://github.com/ontio/ontology/tree/master/smartcontract/service/native/cross_chain/header_sync) for more details |

### Cross Chain Management Contract

| Interface Method        | Description                                                  |
| ----------------------- | ------------------------------------------------------------ |
| **CreateCrossChainTx**  | Creates cross chain transactions, invoked by service contracts when a cross chain function is carried out in the logic, transaction includes unique chain ID, transaction is recorded in the merkle tree, please refer to the [code](https://github.com/ontio/ontology/tree/master/smartcontract/service/native/cross_chain/cross_chain_manager) for more details |
| **ProcessCrossChainTx** | Fetches and processes cross chain transactions, invoked by the relayer when fetching transactions and merkle proofs, finds the merkle root of a transaction based on the block height (in the block header), verifies the legitimacy of transaction using the transaction parameters, invokes the service contract on the target chain, please refer to the [code](https://github.com/ontio/ontology/tree/master/smartcontract/service/native/cross_chain/cross_chain_manager) for details |

## Cross Chain Interaction Between Chains

<div align=center><img width="800" height="570" src="resources/ark.png"/></div>

The figure above illustrates the cross chain interaction between chain A to chain B. The user sends a cross chain request from chain A by invoking a dApp's cross chain interface, and on the target chain B the dApp's smart contract executes the necessary logic to produce the final result. Chain A and B implement the two contracts and other necessary interfaces, and anyone can develop an infrastructure for dApps around the cross chain management contract. The contracts deployed on chain A and chain B make up a complete cross chain dApp.

The complete process flow from chain A to chain B is as follows:

- The user invokes the service contract on chain A, which then in turn invokes the cross chain management contract. The management contract transfers the parameters to the target chain and a cross chain transaction is created by management contract which is sent to the target chain based on block generation on chain A;
- Since there is no means of automatic data exchange between two chains, a **relayer** needs to be set up to transfer block header details from chain A to the relay chain's block header synchronization contract. It also fetches the management contract's response event from chain A which encapsulates the parameters passed by the user, and also fetches the merkle proof. Next, it groups this information together and sends it to the cross chain management contract.
- The management contract fetches the block headers from chain A, verifies whether or not the cross chain parameters and the proof are valid, and then transmits the necessary information to chain B in the form of an event;
- Chain B's relayer transfers the relay chain's block headers to chain B's block header synchronization contract. The relevant chain B cross chain transaction parameters and respective merkle proofs are fetched from the ledger records of the relay chain and transmitted to chain B's cross chain management contract;
- The management contract of chain B determines the legitimacy of the cross chain transaction information and then invokes the relevant target contract and completes the cross chain contract invocation;

There are two different merkle proofs that are transferred to the relay chain:
1. The merkle proof that is used to verify the legitimacy of cross chain transactions from chain A
2. The merkle proof that is used to ensure that a transaction has been created and has occurred on the relay chain

These merkle proofs help establish a trust mechanism for the cross chain ecosystem. Any chain can join the cross chain ecosystem by setting up the communication interface with the relay chain.

## Chain Elements to be Submitted to the Cross Chain Ecosystem

1. **Genesis Block of the Chain**

The genesis block's block header needs to be submitted to the consortium chain of the cross chain ecosystem so as to allow to it verify the consequent transactions and block headers on this chain.

> Another high-priority block such as block where the block generation cycle changes can also be submitted in place of the genesis block.

2. **Verification and Analysis Method, Block Header Structure** 

The block header verification and processing method along with the block structure needs to be submitted so that the relay chain can verify the block headers that will be transferred from this chain.

3. **Merkle Tree Structure and Verification Method**

The merkle tree structure and the respective verification needs to be submitted to the relay chain such that cross chain transactions from this chain can be verified.

## Register the Chain on Relay Chain

After setting up the necessary cross chain infrastructure, the chain can be registered on the relay chain. The registration API used is as follows:

```go
type RegisterSideChainParam struct {
	Address      string    
	ChainId      uint64    
	Router       uint64    
	Name         string    
	BlocksToWait uint64    
}
```

|  Parameter   | Description                                                  |
| :----------: | ------------------------------------------------------------ |
|   Address    | The wallet address that will be associated to this chain and will be authorized to modify the registration information |
|   ChainId    | Chain ID of this chain                                       |
|    Router    | The routing protocol of the chain, existing routing protocols - **1:BTC**, **2:ETH**, **3:ONT**, isomorphic chains can select existing protocol, rest must first define the respective protocol and then select |
|     Name     | Name of the blockchain network                               |
| BlocksToWait | No. of blocks to wait for in order to confirm finality                |

As the cross chain ecosystem grows with time and more and more chains join in, different routing protocols will become a part of the system and isomorphic chains will directly be able to use the appropriate routing protocol.

After the registration process is complete and the Cross Chain Council approves the application, the chain officially becomes a part of the cross chain ecosystem.
