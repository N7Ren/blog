---
title: How to extract state and storage for a contract from the Ethereum database
date: "2019-12-30T08:00:00.000Z"
template: post
draft: false
slug: "extract-state-and-storage-for-contract-from-ethereum-database/"
category: "ethereum"
tags:
  - "ethereum"
  - "ethereumjs"
  - "nodejs"
  - "crypto"
  - "blockchain"
description: "Retrieve state information about an account from the Ethereum blockchain"
---

For this task we will use *Node.Js*, the offical Ethereum client *geth* and the <a href="https://github.com//ethereumjs/ethereumjs-vm" target="_blank">EthereumJS-VM</a> library. It can open and process the locally stored Ethereum database which is a <a href="https://en.wikipedia.org/wiki/LevelDB" target="_blank">LevelDB</a>. A *LevelDB* is an efficient *key-value* storage created by *Google*.

## Syncing the blockchain with geth Ethereum client

Download the current geth version <a href="https://geth.ethereum.org/downloads/" target="_blank">here</a>. After extracting start it with the following flags: 
`./geth --datadir "/media/ren/2nd/geth/ethereum" --testnet --syncmode "full" --gcmode "archive"`

The first option `--datadir "path_to_ethereum_folder"` specifies where the Ethereum blockchain information should be stored. The second option `--testnet` means that we want to sync the <a href="https://github.com/ethereum/ropsten/blob/master/revival.md">ropsten</a> testnet. This is so we can begin quickly - the sync process would take way longer for the *mainnet*. The third and fourth option `--syncmode "full"` and `--gcmode "archive"` are set to make sure the whole blockchain with all the necessary information is downloaded. Otherwise *geth* would be pruning block and state information to allow a faster sync of the blockchain.

## Extracting state information from the Ethereum blockchain

Now we need to install all necessary libraries:
Run `npm install level ethereumjs-vm` in the terminal or use the `package.json` in my <a href="https://github.com/RemoteRen/ethereumjs-experiments" target="_blank">git repository</a>.

**Warning**: The following code should only be executed if you have stopped the *geth* sync or made a copy of the *geth* database otherwise your Ethereum database could get corrupted.

First we define all the objects:
```javascript
//imports are left out for brevity
const db = level('/media/ren/2nd/geth/ethereum/geth/chaindata') //open ethereum leveldb stored in path
const bc = new Blockchain({ db: db }) //takes db and makes it parsable

const address = '0x7ac337474ca82e0f324fbbe8493f175e0f681188' // random ropsten contract
const blockNumber = 19693 //block number which has a stored state of account
```

Now we are ready to extract the state and storage information for an Ethereum account:
```javascript
start()

async function start () {
  const block = await getBlock(blockNumber) //get block information
  const trie = new Trie(db, block.header.stateRoot) //open state trie at this block
  const state = await getState(trie, address) //get state from state trie for address

  if (state != null) {
    const acc = new Account(state)
    const storageRoot = acc.stateRoot //get current root of storage trie

    console.log('Account fields: ')
    console.log('nonce: ' + ethUtils.bufferToInt(acc.nonce))
    console.log('balance: ' + new BN(acc.balance)) //get balance in wei
    console.log('storageRoot: ' + ethUtils.bufferToHex(storageRoot))
    console.log('codeHash: ' + ethUtils.bufferToHex(acc.codeHash))
    console.log()

    console.log('Storage trie contents for account: ')
    const storageTrie = trie.copy()
    storageTrie.root = storageRoot

    storageTrie.createReadStream()
      .on('data', function (data) {
        console.log('key: ' + ethUtils.bufferToHex(data.key))
        console.log('value: ' + ethUtils.bufferToHex(rlp.decode(data.value)))
      })
      .on('end', function () {
        console.log('Done reading storage trie.')
      })
  }
}
```

The function `getBlock` and `getState` are promisified versions of `blockchain.getBlock(number, callback)` and `trie.get(address, callback)`, respectively. If you want to know how to promisify callback functions you can read this blogpost: <a href="/posts/callback-to-promise-nodejs/" target="_blank">How to convert a callback function to a Promise in Node.js</a>.

Finally, the whole source file is available <a href="https://github.com/N7Ren/ethereumjs-experiments/blob/master/src/read-account-state-and-storage-promisified.js" target="_blank">here</a>.

Sources:
1. <a href="https://github.com/ethereumjs/merkle-patricia-tree/issues/32#issuecomment-363654149" target="_blank">https://github.com/ethereumjs/merkle-patricia-tree/issues/32#issuecomment-363654149</a>
2. <a href="https://medium.com/cybermiles/diving-into-ethereums-world-state-c893102030ed" target="_blank">https://medium.com/cybermiles/diving-into-ethereums-world-state-c893102030ed</a>
3. <a href="https://web.archive.org/web/20181028024327/http://wanderer.github.io/ethereum/nodejs/code/2014/08/12/running-contracts-with-vm/" target="_blank">https://web.archive.org/web/20181028024327/http://wanderer.github.io/ethereum/nodejs/code/2014/08/12/running-contracts-with-vm/<a>
