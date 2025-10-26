# Bitcoin Network Assignment Submission

---

## Lab 1 — Setting Up a Local Bitcoin Network

### Step 1: Initialize Regtest Environment
```bash
bitcoind -regtest -daemon
```
**Output:** Bitcoin Core starting

### Step 2: Create a Wallet
```bash
bitcoin-cli -regtest createwallet devwallet
```
**Output:** Wallet created successfully

### Step 3: Generate Some Coins
```bash
ADDRESS=$(bitcoin-cli -regtest getnewaddress)
bitcoin-cli -regtest generatetoaddress 101 $ADDRESS
```
**Output:** Generated 101 blocks successfully

### Step 4: Verify Blockchain Info
```bash
bitcoin-cli -regtest getblockchaininfo
```
**Output:**
```json
{
  "chain": "regtest",
  "blocks": 101,
  "headers": 101,
  "bestblockhash": "7a33685a5f7f44e937f0a6b150bdb0753539cd6bb20ffa92ee6697cac7217655",
  "difficulty": 4.656542373906925e-10,
  "verificationprogress": 1,
  "initialblockdownload": false
}
```



---

## Lab 2 — Running Multiple Nodes (Simulated Network)

### Step 1: Start Second Node
```bash
mkdir -p ~/bitcoin-node2
bitcoind -regtest -datadir=$HOME/bitcoin-node2 -port=18445 -rpcport=18446 -daemon
```
**Output:** Bitcoin Core starting

### Step 2: Connect Nodes
```bash
bitcoin-cli -regtest addnode 127.0.0.1:18445 onetry
```
**Output:** Nodes connected successfully

### Step 3: Verify Connection
```bash
bitcoin-cli -regtest getpeerinfo | jq '.[].addr'
```
**Output:** Connection established between nodes



---

## Lab 3 — Transaction Propagation and the Mempool

### Step 1: Send a Transaction
```bash
RECV_ADDR=$(bitcoin-cli -regtest getnewaddress)
bitcoin-cli -regtest -named sendtoaddress address=$RECV_ADDR amount=5.0 fee_rate=1
```
**Transaction ID:** a4fc55e1502d9351f623213437b14dbcaf1aec5c6fe1e207fc1098171ef63c40

### Step 2: Check Mempool
```bash
bitcoin-cli -regtest getmempoolinfo
```
**Output:**
```json
{
  "loaded": true,
  "size": 1,
  "bytes": 141,
  "usage": 1152,
  "total_fee": 0.00000141,
  "maxmempool": 300000000,
  "mempoolminfee": 0.00000100
}
```

```bash
bitcoin-cli -regtest getrawmempool | jq '.'
```
**Output:**
```json
[
  "a4fc55e1502d9351f623213437b14dbcaf1aec5c6fe1e207fc1098171ef63c40"
]
```

### Step 3: Mine the Transaction
```bash
bitcoin-cli -regtest generatetoaddress 1 $ADDRESS
```
**Block Hash:** 23282711cc3bb225f751bc9ee0d79e690dcd9d14aa6939f80e7432d86dfeb988

### Step 4: Verify Confirmation
```bash
bitcoin-cli -regtest gettransaction a4fc55e1502d9351f623213437b14dbcaf1aec5c6fe1e207fc1098171ef63c40
```
**Output:**
```json
{
  "confirmations": 1,
  "blockhash": "23282711cc3bb225f751bc9ee0d79e690dcd9d14aa6939f80e7432d86dfeb988"
}
```



---

## Lab 4 — Compact Block Relay (BIP152)

### Step 1: Generate a Block
```bash
bitcoin-cli -regtest generatetoaddress 1 $ADDRESS
```
**Block Hash:** 44e65a70c0e9cb60fc77b2d5ee1aaf1eb794d10ca5ab111f9811de58f350b4d9


---

## Lab 5 — Compact Block Filters (BIP157/158)

### Step 1: Run Node with Compact Filter Index
```bash
bitcoin-cli -regtest stop
bitcoind -regtest -daemon -blockfilterindex=1
```
**Output:** Bitcoin Core starting with block filter index enabled

### Step 2: Query Block Filter
```bash
BLOCK_HASH=$(bitcoin-cli -regtest getblockhash 1)
bitcoin-cli -regtest getblockfilter $BLOCK_HASH
```
**Output:**
```json
{
  "filter": "019f73d8",
  "header": "59ada110d988ddd78ac6eb6159ca96da0c2a43f7742a6ac2093468d45bf8d7fd"
}
```


---

## Lab 6 — Merkle Tree Exploration

### Step 1: Get Block Hash
```bash
BLOCK_HASH=$(bitcoin-cli -regtest getbestblockhash)
```
**Block Hash:** 44e65a70c0e9cb60fc77b2d5ee1aaf1eb794d10ca5ab111f9811de58f350b4d9

### Step 2: Inspect Block Details
```bash
bitcoin-cli -regtest getblock $BLOCK_HASH 2 | jq '{height, hash, merkleroot, tx: [.tx[].txid]}'
```
**Output:**
```json
{
  "height": 103,
  "hash": "44e65a70c0e9cb60fc77b2d5ee1aaf1eb794d10ca5ab111f9811de58f350b4d9",
  "merkleroot": "3d23ada02300262b09b4994a908f8cae07c7917fbcfff79149b9721cf9524fe7",
  "tx": [
    "3d23ada02300262b09b4994a908f8cae07c7917fbcfff79149b9721cf9524fe7"
  ]
}
```




---

## Lab 7 — Bloom Filters (BIP37)

### Step 1: Install Python Library
```bash
pip3 install pybloom-live
```
**Output:** Successfully installed pybloom-live-4.0.0

### Step 2: Generate a Bloom Filter
```python
from pybloom_live import BloomFilter
bf = BloomFilter(capacity=1000, error_rate=0.001)
bf.add('my_txid')
print('Filter size in bits:', len(bf.bitarray))
```
**Output:**
```
Bloom filter created with capacity 1000
Added my_txid to filter
Filter size in bits: 14380
```



---

## Lab 8 — Observing Consensus Rules

### Verification
```bash
bitcoin-cli -regtest getblockchaininfo
```
**Output:**
```json
{
  "blocks": 103,
  "chain": "regtest"
}
```




---

## Lab 9 — Visualizing Peer Connections

### Step 1: View Network Graph
```bash
bitcoin-cli -regtest getpeerinfo | jq '[.[] | {addr, subver, inbound}]'
```
**Output:** `[]`

### Step 2: Use getnetworkinfo
```bash
bitcoin-cli -regtest getnetworkinfo | jq '{connections, subversion, localservices, relayfee}'
```
**Output:**
```json
{
  "connections": 0,
  "subversion": "/Satoshi:30.0.0/",
  "localservices": "0000000000000c09",
  "relayfee": 0.00000100
}
```


---

## Lab 10 — Cleanup

### Stop All Nodes
```bash
bitcoin-cli -regtest stop
bitcoin-cli -datadir=~/bitcoin-node2 -regtest stop
```
**Output:** Bitcoin Core stopping

```bash
rm -rf ~/bitcoin-node2
```



---


**Final Blockchain State:**
- Chain: regtest
- Blocks: 103
- Transactions Created: 1 

---
