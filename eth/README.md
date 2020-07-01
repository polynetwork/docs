<h1 align="center">Ethereum Cross Chain Specifications</h1>
<h4 align="center">Version 1.0 </h4>

English | [中文](ethereum_cross_chain_specification_CN.md)

# Introduction

Ethereum is the first public blockchain that supports smart contracts and allows their application in a blockchain environment. Using smart contracts can tremendously increase the potential and scalablity of blockchain based systems and applications. But with the rapid development of the blockchain industry and the overall ecosystem, the need for communication between chains has become a hurdle. Since data and value cannot be freely transferred between chains, this has given rise to isolated islands of data and value. Also, it is very difficult for a single chain to integrate several different functions while also maintaining performance and the integrity of its architecture, which then leads to bottlenecks in the user experience.
Hence, using cross chain technology to link Ethereum to other chains is essential to offer solutions to the above stated issues.

Since Ethereum supports a Turing complete smart contract system, Ethereum is very versatile in terms of the kind of smart contract logic is supports. If Ethereum is linked to other blockchains, we need to consider the different scenarios that would consist of various kinds of data being transferred between chains.

So then how is this cross chain ecosystem implemented? Let us take closer look.

## Framework

<div align=center><img width="280" height="300" src="./pic/design.png"/></div>

As illustrated in the above figure, the cross chain framework consists of the Ethereum chain, the Ethereum relayer, the relay chain, the target chain's relayer, and finally the target chain itself in that order. Simply put, the user might be using a dApp in a cross chain scenario, and an execution result is transferred to the relay chain by Ethereum's relayer. The target chain's relayer then picks this up and transfers it to the target chain which then processes the obtained infromation. This completes the cross chain communication flow.

The components involved in this process:

- [**Relay Chain**](../poly/README.md): The relay chain is one of the crucial components of the cross chain ecosystem. Each different type of node that is deployed and maintained by different individuals or organizations and has its unique governance and trust mechanism. The relay chain is responsible for connecting them, standardizing cross chain data flow and interfaces, and verifying the legitimacy of cross chain data, etc.
- [**Relayer**](https://github.com/polynetworks/docs/blob/master/eth/ethereum_relayer_manual.md): Each chain in the cross chain ecosystem has a relayer associated to it that is reponsible for monitoring the transactions taking place on the relay chain and then transferring them to the respective target chain, and vice versa. Thus they act as the medium of interaction between Ethereum and the outside world. Relayers collect small incentives carrying out the above stated tasks.
- [**Ethereum Chain**](https://github.com/polynetworks/docs/blob/master/eth/ethereum_relayer_manual.md): Ethereum is the base chain from where the cross chain ecosystem originates. Any dApp can utilize the cross chain smart contract mechanism to implement complex functionality for cross chain scenarios.


## Background

### MPT Proof and the Light Client  

- **MPT Proof**: The Merkle Patricia Tree is an optimized data structure that combines the strengths and features of a Merkle tree and prefix tree and helps save storage space. It is primarily used to organize and manage account data on Ethereum and other critical information such as transaction data. The MPT proof is a data structure that is generated based on the MPT tree to provide evidence of storage status in smart contracts. Users can use it to determine the validity of block status at a particular height.
- **Light Client**: The light client is a tool used to store limited, but critical blockchain data. It is a reliable, secure, light method to store block information.


## The Principle

Merkle proofs are a solution. When events and transactions are recorded on the Ethereum blockchain, a corresponding merkle proof. By the virtue of having a merkle proof and the correct block header we can prove that a particular transaction or event did occur on the Ethereum chain with certainty.

Determining the validity of cross chain data is one of the biggest issues that need to be addressed. How do we confirm that a certain event or transaction has taken place on the Ethereum chain? If the correct block header exists on the relay chain, it can directly use the MPT proof of a particular event or transaction to confirm whether it occurred by sending it to the Ethereum chain. This completes the Ethereum to relay chain transfer. Therefore, establishing whether a cross chain transaction took place on Ethereum chiefly involves two steps, i.e. fetching and synchronizing the Ethereum transactions on the relay chain, and then sending the corresponding MPT proof of the cross chain transaction to the relay chain.

Now the component of the cross chain ecosystem that actually carries out this process of synchronizing block headers and MPT proofs is the **relayer**. The relayer monitors the transactions taking place on Ethereum and synchronizes the block headers on the relay chain along with the respective cross chain transactions and events. It also needs to monitor the relay chain and send the block headers and the respective cross chain transactions to Ethereum.

### Synchronizing Ethereum Block Headers on the Relay Chain

To synchronize block headers from Ethereum to the relay chain you first need to specify an initial block header from where the synchronization starts. The initial block header and the consequent block headers will be synchronized on the relay chain.

Ethereum block headers synchronized on relay chain:

<div align=center><img width="350" height="260" src="pic/sync_headers.png"/></div>

Block headers are synchronized one by one to the relay chain. The relay chain relies on the same verification principle that the Ethereum light client uses to ensure the legitimacy of the block headers, including the mining difficulty of the Ethereum chain.

The relay chain system can identify and withstand forks. The synchronization process carries on with the Ethereum main chain.

### Synchronizing Relay Chain Block Headers on Ethereum

Synchronizing relay chain block headers on Ethereum also occurs in a similar manner. An initial block header is selected from where the synchronization process begins, and all the subsequent blocks are synchronized.

Relay chain headers synchronized on Ethereum:

<div align=center><img width="480" height="200" src="pic/poly_hdrs.png"/></div>

However, in this case not all the blocks headers need to be synced on the Ethereum chain. The only blocks that need to be synced on the Ethereum chain are the cross chain block headers to Ethereum and the block headers wherein any changes occurred in the verification nodes of the relay chain. The relay chain, just as a typical blockchain, also has finality characteristics.

### Transactions from Ethereum to the Relay Chain

In a scenario where let's say an Ethereum to BTC transaction needs to be carried out, first the transaction is created by the user and send to the Ethereum chain. This cross chain transaction triggers the `lock` of the **ETH to BTC** service contract, and simultaneously an event needs to be created and recorded on Ethereum along with its merkle proof which will be generated on the Ethereum chain. Once the block header corresponding to the transaction and its respective merkle proof are synchronized on the relay chain, it can verify the existence of an event on Ethereum. 

A relayer is necessary in this condition that listens for cross chain transactions on ETH and transmits them to the relay chain along with the corresponding merkle proof.

Ethereum to relay chain cross chain transaction:

<div align=center><img width="780" height="360" src="pic/eth2poly.png"/></div>

### Transactions from the Relay Chain to Ethereum  

Sending a transaction from the relay chain to Ethereum is much like the opposite and involves sending the block headers and merkle proof.

- The relayer sends the relay chain block headers to Ethereum's block header synchronization contract. This contract records and maintains the relay chain's block headers;
- The relayer listens to the management contract on the relay chain, fetches the Ethereum cross chain events, and sends the events along with the respective merkle proofs to Ethereum. Finally, the management contract invokes a proxy contract that releases the user's locked ETH. For instance, in a transaction from BTC to ETH, the process of sending the cross chain events and merkle proofs is carried out by the `verifyAndExecuteTx` method of the management contract.

## Ethereum Cross Chain Transaction 

The process flow of cross chain transactions from Ethereum to other chains:

<div align=center><img width="700" height="500" src="pic/cross_progress_en.png"/></div>

1. The user sends a transaction that, for instance, transfers 1 ETH from account A on Ethereum to account B on the target chain

2. Ethereum locks the 1 ETH transferred from account A, generates the transaction that is sent to the target chain and its merkle proof

3. The ETH relayer constantly synchronizes Ethereum block headers on the relay chain, and monitors the Ethereum chain for cross chain transactions. Relayer transmits the cross chain transaction and the merkle proof generated by Ethereum to the relay chain

4. The Ethereum block headers and the respective merkle proof on the relay chain allow it to verify the legitimacy and validity of a cross chain transaction

5. If the transaction is valid, the relay chain generates a transaction to be sent to the target chain, along with the transaction's merkle proof

6. The target chain relayer monitors the transactions on the relay chain and looks out for the transactions that need to be transferred to the target chain. It syncs the block header to the target chain and sends the transaction and the corresponding merkle proof along with it.

7. When the target chain confirms the transaction's legitimacy, the target chain executes a final transaction that sends the 1 ETH to account B.
