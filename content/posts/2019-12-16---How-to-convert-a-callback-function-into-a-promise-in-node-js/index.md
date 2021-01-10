---
title: How to convert a callback function to a Promise in Node.js
date: "2019-12-16T09:00:00.000Z"
template: post
draft: false
slug: "callback-to-promise-nodejs"
category: "nodejs"
tags:
  - "nodejs"
description: "Turn a callback function into a Promise to avoid \"Callback Hell\""
---

A snippet I often need when I work with *Node.js* is turning a callback function into a Promise. I do this so I can use the `async/await` syntax and avoid the <a href="http://callbackhell.com/" target="_blank">Callback Hell</a>.

For example when I want to get a block from the Ethereum blockchain with <a href="https://github.com/ethereumjs" target="_blank">EthereumJS</a> I use the following code:
```javascript
const level = require('level')
const Blockchain = require('ethereumjs-blockchain').default

const db = level('/media/ren/2nd/geth/ethereum/geth/chaindata') // path to geth database
const bc = new Blockchain({ db: db })

bc.getBlock(
  blockNumber, (err, block) => {
    if (err) {
      console.log(err)
    } else {
      console.log(new BN(block.header.number));
      //Further processing
    }
  }
)
```

Because I want to use the `getBlock` function more than once and as a *Promise* I convert into one:
```javascript
const bc = new Blockchain({ db: db })

function getBlock (blocknr) {
  return new Promise((resolve, reject) => {
    bc.getBlock(
      blocknr, (err, block) => {
        if (err) {
          reject(err)
        } else {
          resolve(block)
        }
      })
  })
}
```

Now I can use it in combination with the `async/await` syntax like this:
```javascript
const bc = new Blockchain({ db: db })

start()

async function start() {
 try {
    const previousBlock = await getBlock(17999)
    console.log(new BN(block.header.number))

    const currentBlock = await getBlock(18000)
    console.log(new BN(block.header.number))
    //further processing
  } catch (err) {
    console.log(err)
  }
 }
```

This makes for a more readable and less error prone program.

For completion sake I'll mention another possibility to do this:
```javascript
const promisify = require('util.promisify')
const myfunc = util.promisify(myfunction);
```
This approach however is dependent on a including the *util.promisify* function and I like to keep the dependencies of an application to a minimum.

Source: <a href="https://softwareengineering.stackexchange.com/questions/279898/how-do-i-make-a-javascript-promise-return-something-other-than-a-promise/350041" target="_blank">https://softwareengineering.stackexchange.com/questions/279898/how-do-i-make-a-javascript-promise-return-something-other-than-a-promise/350041</a>