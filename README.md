# Spacechain launcher
An implementation of a spacechain with a mechanism for launching your own

# What works

The governance mechanism seems to work. Spaceblocks are created by hashing some data and committing that hash to bitcoin's blockchain. Client software comes to consensus on the order of transactions by observing the order in which spaceblock hashes are committed. Spaceblocks themselves are distributed out of band through a seemingly peer to peer (but actually not) method: nostr dms.

The script address for anchor outputs seems to work. It's an anyone-can-spend address. Spacechain miners are expected to use fee bidding to compete with one another to move that utxo and, in moving it, commit the hash of a new spaceblock to bitcoin's blockchain. If someone steals the anchor output, that is detectable because the output won't go right back into the anyone-can-spend address (if it did, it wouldn't be a theft, because you could just keep building on top of it by moving it again), and at that point miners can bid to make a new anyone-can-spend utxo. The first anchor utxo to make it into the blockchain and commit a new spaceblock wins and is, by consensus, the new anchor utxo that everyone else must build on top of.

Fork resolution seems to work. If you request a block and someone sends you one that does not build on your latest one, your browser will try to resolve the fork by repeatedly requesting the parents, grandparents, and so forth of the contested blocks. If both have a direct lineage back to the genesis block, it is a legitimate fork, and which one to follow is decided first by which one is longer, and, if the chains are the same length, by which chain's anchor transaction comes first at the moment of the fork.

Issuing spacecoins seems to work. You can issue spacecoins by burning sats. Every sat burned gets you a new spacecoin. There is a marketplace where users can peg into the spacechain by burning sats. The software seems to reliably detect the burned coins and appropriately display your balance in the built in wallet.

# What doesn't work

A lot of stuff involving the built in wallet is super wonky or doesn't work at all. Sometimes it thinks you still have spacecoins that you've actually spent. It doesn't have a button for importing an existing wallet, so good luck trying to do that. The wallet tries to broadcast transactions to other miners but they don't currently do anything in response, like mine them.

Any sort of competitive mining will probably be totally borked til I fix it. One of its problems is that it relies on data from mempool.space to see the status of the anchor utxo. But mempool.space follows a "first seen" rule for transactions with RBF disabled, meaning a miner can pay a low fee with rbf, then pay a high fee, and no one else will know he paid that high fee.

The mining software also doesn't reset your transaction data when a new block comes in, so if you don't do it manually (by typing esoteric commands from the source code into your browser console), your browser might try to mine transactions that have already been mined.

A bunch of other stuff is broken too but that's good for now.
