[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/Hp77wqaS)
# assignment-5
# Bitcoin Network

# Hands-On Lab Guide — Bitcoin Network Deep Dive (Developers)

**Overview**

This lab complements the slide deck and speaker notes. It focuses on building and observing a local
Bitcoin network, understanding transaction propagation, mempool behavior, and experimenting with
compact filters.

You will use Bitcoin Core in regtest mode for safety and speed — no need to connect to the real
Bitcoin network.

# Lab 1 — Setting Up a Local Bitcoin Network

Step 1: Initialize Regtest Environment

bitcoind -regtest -daemon
Launches a local Bitcoin node in regression test mode.

Step 2: Create a Wallet

bitcoin-cli -regtest createwallet devwallet
Creates a wallet named devwallet for testing.

Step 3: Generate Some Coins

ADDRESS=$(bitcoin-cli -regtest getnewaddress)
bitcoin-cli -regtest generatetoaddress 101 $ADDRESS

Mines 101 blocks to your address — gives you spendable test BTC.
Expected output: Block hashes printed in terminal.

Step 4: Verify Blockchain Info

bitcoin-cli -regtest getblockchaininfo
Confirms height, difficulty, and network details.


# Lab 2 — Running Multiple Nodes (Simulated Network)
You can run multiple nodes on different ports to simulate propagation.

Step 1: Start Second Node

mkdir -p ~/bitcoin-node2

bitcoind -regtest -datadir=~/bitcoin-node2 -port=18445 -rpcport=18446 -daemon

Step 2: Connect Nodes

bitcoin-cli -regtest addnode 127.0.0.1:18445 onetry

Step 3: Verify Connection

bitcoin-cli -regtest getpeerinfo | jq '.[].addr'

You should see 127.0.0.1:18445 listed.

# Lab 3 — Transaction Propagation and the Mempool

Step 1: Send a Transaction

RECV_ADDR=$(bitcoin-cli -regtest getnewaddress)

bitcoin-cli -regtest sendtoaddress $RECV_ADDR 5.0

Step 2: Check Mempool

bitcoin-cli -regtest getmempoolinfo

bitcoin-cli -regtest getrawmempool | jq '.'

Displays current transactions waiting to be mined.

Step 3: Mine the Transaction

bitcoin-cli -regtest generatetoaddress 1 $ADDRESS

Confirms the transaction in a new block.

Step 4: Verify Confirmation

TXID=<your_txid>

bitcoin-cli -regtest gettransaction $TXID

Expected: confirmations: 1

# Lab 4 — Compact Block Relay (BIP152)

**Objective**

Observe reduced bandwidth during block propagation.
Note: For demonstration, run bitcoind with -printtoconsole and enable -debug=net .

Step 1: Enable Compact Blocks

Compact blocks are automatically supported in Bitcoin Core since 0.13.0. 

To view negotiation:

tail -f ~/.bitcoin/regtest/debug.log | grep compact

You’ll see messages like sendcmpct and cmpctblock .

Step 2: Mine a Block and Observe

bitcoin-cli -regtest generatetoaddress 1 $ADDRESS

Watch for compact block announcements in debug logs.

# Lab 5 — Compact Block Filters (BIP157/158)

Step 1: Run a Node with Compact Filter Index

bitcoind -regtest -daemon -blockfilterindex=1

Step 2: Query Block Filter

BLOCK_HASH=$(bitcoin-cli -regtest getblockhash 1)

bitcoin-cli -regtest getblockfilter $BLOCK_HASH

Returns filter header and filter data (Golomb-Rice encoded bitstream).


Step 3: Decode Filter (Optional)

Install neutrino or btcd client to test SPV-like behavior locally.

# Lab 6 — Merkle Tree Exploration

Step 1: Get Block Hash

BLOCK_HASH=$(bitcoin-cli -regtest getbestblockhash)

Step 2: Inspect Block Details

bitcoin-cli -regtest getblock $BLOCK_HASH true

Note the merkleroot field.

Step 3: Verify Merkle Root Manually (Python example)

import hashlib
def double_sha256(b):
return hashlib.sha256(hashlib.sha256(b).digest()).digest()
txids = [bytes.fromhex(txid)[::-1] for txid in ["<txid1>", "<txid2>"]]
root = double_sha256(txids[0] + txids[1])[::-1].hex()
print(root)

Matches the block’s merkleroot .

# Lab 7 — Bloom Filters (BIP37)

Step 1: Use bitcoin-cli RPC (legacy)

bitcoin-cli -regtest setnetworkactive false

Add peers manually with support for BIP37 (use bitcoinj or older btcd client).

Step 2: Generate a Bloom Filter

from pybloom_live import BloomFilter
bf = BloomFilter(capacity=1000, error_rate=0.001)
bf.add('my_txid')
print(bf.bitarray)

Demonstrate how probabilistic matching works.

Discussion: Emphasize deprecation due to privacy leaks (clients revealed interests).

# Lab 8 — Observing Consensus Rules

Step 1: Corrupt a Block (for demo only)

cp ~/.bitcoin/regtest/blocks/blk00000.dat ~/tmp/

**Manually edit a byte — breaks validation**

Step 2: Restart Node

bitcoind -regtest -daemon

Node rejects the corrupted block → error: bad-blk in logs.

Lesson: Consensus rules are strict; invalid data is rejected network-wide.

# Lab 9 — Visualizing Peer Connections

Step 1: View Network Graph

bitcoin-cli -regtest getpeerinfo | jq '[.[] | {addr, subver, inbound}]'

Step 2: Use bitcoin-cli getnetworkinfo

Displays peer count, local services, relay fees, and protocol version.

Tip: Each peer connection is a TCP link exchanging compact messages — similar to sockets in general
networking.

# Lab 10 — Cleanup

Stop all nodes:

bitcoin-cli -regtest stop

bitcoin-cli -datadir=~/bitcoin-node2 -regtest stop

Remove temporary files (optional):

rm -rf ~/bitcoin-node2 ~/.bitcoin/regtest


Conclusion
By completing these labs, developers learn: - How Bitcoin nodes communicate and propagate data. -
How mempool, blocks, and filters interact. - How BIPs (152, 157, 158) optimize bandwidth and privacy. -
How consensus ensures integrity and immutability.

Next step: Integrate these insights into Bitcoin Core RPC automation or monitoring scripts to extend
infrastructure-level understanding.
