# Spacechain launcher
An implementation of a spacechain with a mechanism for launching your own

# How to play with it

Go here: https://supertestnet.github.io/spacechain-launcher/

# What works

Creating a spacechain works (yay!).

The governance mechanism seems to work. Spaceblocks are created by hashing some data and committing that hash to bitcoin's blockchain. Client software comes to consensus on the order of transactions by observing the order in which spaceblock hashes are committed. Spaceblocks themselves are distributed out of band through a seemingly peer to peer (but actually not) method: nostr dms.

The script address for anchor outputs seems to work. It's an anyone-can-spend address. Spacechain miners are expected to use fee bidding to compete with one another to move that utxo and, in moving it, commit the hash of a new spaceblock to bitcoin's blockchain. If someone steals the anchor output, that is detectable because the output won't go right back into the anyone-can-spend address (if it did, it wouldn't be a theft, because you could just keep building on top of it by moving it again), and at that point miners can bid to make a new anyone-can-spend utxo. The first anchor utxo to make it into the blockchain and commit a new spaceblock wins and is, by consensus, the new anchor utxo that everyone else must build on top of.

Fork resolution seems to work. If you request a block and someone sends you one that does not build on your latest one, your browser will try to resolve the fork by repeatedly requesting the parents, grandparents, and so forth of the contested blocks. If both have a direct lineage back to the genesis block, it is a legitimate fork, and which one to follow is decided first by which one is longer, and, if the chains are the same length, by which chain's anchor transaction comes first at the moment of the fork.

Issuing spacecoins seems to work, though it's currently disabled unless you pass a special url parameter (see below). You can issue spacecoins by burning sats. Every sat burned gets you a new spacecoin. There is a marketplace where users can peg into the spacechain by burning sats. The software seems to reliably detect the burned coins and appropriately display your balance in the built in wallet.

# What doesn't work

A lot of stuff involving the built in wallet is super wonky or doesn't work at all. It doesn't have a button for importing an existing wallet, so good luck trying to do that. The wallet tries to broadcast transactions to other miners but they don't currently do anything in response, like mine them. Users can mine their own transactions, but that leads directly to the next problem:

Any sort of competitive mining, or even just having two people try to mine a spaceblock in the same bitcoin block, is totally borked. One of the problems is that my mining function relies on data from mempool.space to see the status of the anchor utxo. But mempool.space follows a "first seen" rule for transactions with RBF disabled, meaning a miner can pay a low fee with rbf disabled, then pay a high fee, and no one else will know he paid that high fee.

The mining software also doesn't reset your transaction data when a new block comes in, so if you don't do it manually (by typing esoteric commands from the source code into your browser console), your browser might try to mine transactions that have already been mined.

I'm not running an always-on miner so you'll have trouble finding any blocks when I'm not running tests. Miners are the only ones who serve blocks right now so you can't really get them from somewhere else. I did put the first 10 or so blocks in the source code.

Due to the fact that the wallet is in such a terrible state, I disabled it unless you pass this url parameter: `&testing_wallet=true`. This also disables the spacecoin issuance mechanism since it relies on you having a wallet to deposit the newly issued spacecoins into.

A bunch of other stuff is broken too but that's good for now.

# Don't spacechains involve burning bitcoins? And isn't that an attack on bitcoin?

Spacechains do burn bitcoins but I don't think it is an attack on bitcoin. I also don't recommend it, because you'll probably never get back all that you burnt.

# Why did you make a thing that burns bitcoins if you don't recommend it?

Lack of self control. When I originally heard about spacechains I was like, "Burn bitcoins? That's a dumb idea," and then I started thinking up reasons why it's dumb, and then I wondered, "How would you even *do* that?" so I considered it a few minutes and realized how it might be done, and then I was stuck. I had to try it and see if it would work. And it does work, but that doesn't make it a good idea.

# I oppose this because it burns bitcoins

That's fine, you can oppose it. I think a few things are worth pointing out in its defense:

(1) Not *everyone* has to burn bitcoins to use a spacechain. The spacechain creator will probably do that to issue an initial supply of coins, but after that's done, other people can just use those issued coins by buying them from the spacechain creator or other holders.

(2) You can get rid of the bitcoin burning part by issuing some other token in a trusted manner. For example, tether corp could create a spacechain whose only token is tethers, and they wouldn't have to burn any bitcoins to do that.

(3) The reason why I *personally* think burning bitcoins is dumb is because I don't see any meaningful value I can get in compensation for it. If a spacechain really offered some feature that I thought was worth $10, I would be willing to burn up to $10 in bitcoin to get that feature, because in that case I'm really just purchasing something, and whether I send $10 to a seller or burn them in a burn transaction, it makes no difference to me. So if someone comes up with a really compelling use case for spacechains (which I doubt will happen) I might burn a few sats to use it. I don't think it's *inherently* dumb, I just doubt anyone will come up with anything useful enough to justify it according to my personal preferences.
