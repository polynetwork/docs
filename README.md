<h1 align="center">Poly Network</h1>
<h4 align="center">Version 1.0 </h4>

English | [中文](README_CN.md)

Welcome to the Poly Network cross chain technology center!

Block technology has now undergone over 10 years of gradual development. Even though many public chain systems have emerged, most of the existing blockchain architecture systems focus on experimenting with the scalability and performance of a single blockchain, while for a single blockchain, due to the constraints that come into picture due to the architecture system and development direction, it is difficult to meet all the needs. Therefore, we expect that the future blockchain ecology must be a pattern in which multiple blockchains coexist. Different chains with different characteristics can become part of the blockchain infrastructure, but the chains are still limited to themselves, each forming its own island of value. Chains that are different in nature lack quick interoperability and convenient means of value circulation. In order to build a better next-generation internet infrastructure, we have launched a new cross-chain technology, the Poly Network.

Cross-chain technology is a new technical method to allow inter-chain interactions via cross-chain interoperability protocols that are based on the existing single-blockchain architecture design. The cross-chain interoperability protocol was first proposed by Vitalik Buterin in September 2016. He divided the cross-chain interoperability protocol into three modes: Notary schemes, side chains/relays, and hash locking. The notary mechanism refers to the interaction through a trusted third party as part of the process of cross-chain interaction, which is mainly implemented through mechanisms such as "single sign, multi-sign notary mechanism"; the side chain/relay mode is a protocol that is currently being used, for example: Cosmos and Polkadot and other popular cross-chain projects all use this protocol. It refers to a technical methodology that can rely on a trusted third party for cross-chain transaction verification and can conduct cross-chain transaction verification on its own; Hash time lock is the underlying technology of the earliest lightning network. The protocol consists of time lock and hash lock. Time lock refers to the agreement between the two parties that the transaction must be submitted within a certain time to be valid. Hash lock refers to a hash value H, where if the original image R is provided such that Hash (R) = H, the promise is deemed valid, otherwise it is invalid. If the two parties involved in the transaction could not succeed for any reason, time lock can allow the parties to the transaction to get their assets back to avoid losses due to fraud or transaction failure.

Poly Network is based on the side-chain/relay mode and adopts a two-layer architecture. It employs the Poly relay chain as a cross-chain coordinator, multiple homogeneous chains as cross-chain transaction executors, and Relayer as a cross-chain information porter. By resolving issues such as trust, security and transaction issues of chain data, we have realized a safe, easy-to-use and efficient cross-chain system.

## Characteristics

* Light weight cross chain protocol
* Easy, convenient integration
* Support for heterogenous and homogeneous chains
* Supports cross chain transaction consistency and finality
* Supports cross chain asset and data exchange
* Secure, reliable cross chain protocol based on cryptography
* Supports homogeneous chain protocol scalability (BTC/ETH/NEO/Ontology/Cosmos)
* Openly accessible relay chain 

## Documentation and Guides

* [Poly Documentation](poly/README.md)
* [Bitcoin Documentation](btc/README.md)
* [Ethereum Documentation](eth/README.md)
* [NEO Documentation](neo/README.md)
* [Ontology Documentation](ont/README.md)
* [Cosmos Documentation](cosmos/README.md)
* [Chain Configuration Info](config/README.md)

## Contributions

To contribute to the project, please submit pull requests with signed commits. We look forward to the contributions you make!

You can also send your code to the developer mailing list via email. Feel free to join the Poly Network mailing list and developer forum.

Please provide details regarding your submission to Poly. Here's a sample format:

  Header line: explain the commit in one line (use the imperative)

  Body of commit message is a few lines of text, explaining things
  in more detail, possibly giving some background about the issue
  being fixed, etc.

  The body of the commit message can be several paragraphs, and
  please do proper word-wrap and keep columns shorter than about
  74 characters or so. That way "git log" will show things
  nicely even when it's indented.

  Make sure you explain your solution and why you're doing what you're
  doing, as opposed to describing what you're doing. Reviewers and your
  future self can read the patch, but might not understand why a
  particular solution was implemented.

  Reported-by: whoever-reported-it
  Signed-off-by: Your Name <youremail@yourhost.com>

## Open Source Community

## License

Poly Network is licensed under the GNU Lesser General Public License, v3.0. Refer to the LICENSE file in the root directory of the project for details.
