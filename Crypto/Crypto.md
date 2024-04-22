# BlockChain

### Architecture

- wallet: store the priviate key locally, controlled by user
- Blockchain: 
- Front-end: interact in-between wallet and block-chain
  - read data from blockchain & send transaction to the blockchain
  - user does sth in frontend, frontend send messages to wallet, wallet ask user's confirmation, then wallet signs a transaction and give back to frontend, which will be send to the blockchain







# Ethereum

### EVM compatible

Other blockchains that use ETH's technology.

### Layer 2 blockchains

A Layer-2 blockchain represents **a collection of scaling solutions crucial for enhancing the performance and scalability of Layer-1 blockchains, like Ethereum**. These layer 2 protocols operate atop the primary blockchain, significantly reducing congestion, lowering transaction costs, and boosting throughput.

Polygon, Arbitrum, Optimism...

### Smart Contract

Bitcoin only able to process simple transactions, while ETH can process smart contract, i.e., a small program that runs on blockchain.

##### Solidity

A statically-typed curly-braces programming language designed for developing smart contracts that run on Ethereum.