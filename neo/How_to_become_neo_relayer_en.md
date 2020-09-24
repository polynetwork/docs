# neo-relayer

English | [中文](./How_to_become_neo_relayer_CN.md)

## Structure

On one hand, neo-relayer monitors Neo network continuously, scans each Neo block and synchronizes those key block headers which contains different "NextConsensus" field to the relay chain. At the same time, neo-relayer can recognize cross-chain transactions, and provides the "StateRoot" of the block containing those transactions together with the "MerkleProof" of those transactions to the relay chain.  
On the other hand, neo-relayer also monitors the relay chain, scans each relay chain block and synchronizes those key block headers which has changed next consensus addresses to Neo chain. Moreover, the relay chain creates cross-chain transactions to Neo chain when it receives cross-chain transactions from other chains. Then neo-relayer relays the "StateRoot" of the block containing those transactions together with the "MerkleProof" of those transactions to Neo chain.  
The entire Neo cross-chain ecosystem can continue to function normally even with just one active neo-relayer.

## Setup

For more information about how to configure, deploy and run a neo-relayer, refer to [neo-relayer]([./How_to_become_neo_relayer_en.md](https://github.com/polynetwork/neo-relayer/blob/master/README.md)).
