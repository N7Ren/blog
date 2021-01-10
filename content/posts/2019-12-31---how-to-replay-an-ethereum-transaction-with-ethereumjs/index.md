---
title: How to replay an Ethereum transaction with EthereumJS
date: "2019-12-31T09:00:00.000Z"
template: post
draft: false
slug: "replay-ethereum-transaction-with-ethereumjs"
category: "ethereum"
tags:
  - "ethereum"
  - "ethereumjs"
  - "nodejs"
  - "crypto"
  - "blockchain"
description: "Execute an Ethereum transaction again to retrieve information at this state"
---

For this task you need *Node.Js*, a blockchain database of the *ropsten* testnet synched without pruning with the *geth* Ethereum client and the <a href="https://github.com/ethereumjs/ethereumjs-vm" target="_blank">EthereumJS-VM</a> library. You can read about how to setup the necessary environment in this blogpost: <a href="/posts/extract-state-and-storage-for-contract-from-ethereum-database/" target="_blank">How to extract state and storage for a contract from the Ethereum database</a>

## Replay a transaction

**EtherumJS-VM** has the ability to run a transaction which can be used to replay a transaction from the blockchain:
`runTx({ block: block, tx: tx, skipNonce: true, skipBalance: true })` 
It takes the *block* and the *transaction* as input. The last two parameters are about checking if the *nonce* and *balance* are correct. Since the transaction was already executed and the result is already stored in the blockchain we don't have to check it again

**Warning**: The following code should only be executed if you have stopped the *geth* sync or made a copy of the *geth* database otherwise your Ethereum database could get corrupted.

The following code shows how to replay a transaction:
```javascript
//Imports left out for brevity
const db = level('/media/ren/2nd/geth/ethereum/geth/chaindata') // open ethereum leveldb stored in path
const bc = new Blockchain({ db: db }) // takes db and makes it parsable

const chain = 'ropsten'
const address = '0x7ac337474ca82e0f324fbbe8493f175e0f681188' // random ropsten contract
const blockNumber = 19693 // block number which has a transaction with address as target

start()

async function start () {
  const block = await getBlock(blockNumber) // promisified getBlock
  const tx = block.transactions[0]

  // get the parent state to start the replay before the current block was formed
  const parentBlock = await getBlock(blockNumber - 1)
  const parentTrie = new Trie(db, parentBlock.header.stateRoot) // open state trie

  // create vm with parent state
  const vm = new VM({
    state: parentTrie,
    blockchain: bc,
    chain: chain
  })

  // replay transaction
  const txResult = await vm.runTx({ block: block, tx: tx, skipNonce: true, skipBalance: true })
  console.log(txResult)
}
```

Here it is important that the state is taken from the parent block (current block - 1) to ensure that the transaction was not already executed. If you would take the state of the current block the state would already have stored the result of the execution.

Finally, the whole source file is available <a href="https://github.com/N7Ren/ethereumjs-experiments/blob/master/src/replay-transaction.js" target="_blank">here</a>.