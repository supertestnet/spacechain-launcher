# Spacechain launcher
An implementation of a spacechain with a mechanism for launching your own

# How to play with it

Go here: https://supertestnet.github.io/spacechain-launcher/

# Video

https://www.youtube.com/watch?v=ajzy_8Nibv8

# Principles of operation

1. Alice wants to make a bitcoin sidechain
2. For its first block she creates some data and hashes it
3. She creates a bitcoin transaction that commits to that hash
4. She uses a p2p messaging network to share the block she created with other people
5. For its subsequent blocks she repeats steps 2-4 but now each block must "point to" its parent
6. To ensure that anyone can compete to make more blocks in case Alice stops (or censors someone), the utxo created in step 3 must use an anyone_can_spend address -- the ACSA -- and subsequent blocks must be committed to by spending *from* the ACSA
7. As long as sidechain "block producers" follow steps 5-6 repeatedly, the "trail of blocks," i.e. their order in the blockchain, is easy to follow
8. If no one ever deviated from this pattern we would not need to do anything more -- as an aside, sighash_anyprevout and op_check_template_verify do that: they make a covenant so that block producers *have to* follow this pattern, anything else is an invalid transaction
9. But we do not have sighash_anyprevout or op_check_template_verify so I do this instead: suppose a rogue miner takes the money from the ACSA and sends it to his wallet
10. That is detectable by anyone syncing the chain because the trail mentioned in step 7 suddenly stops
11. So what they do now is, they rely on this fallback mechanism: block producers can bid to put a new utxo in the ACSA to serve as the new "recovery point" for the trail
12. Since many people can create this "recovery point," nodes must come to consensus on which one to follow
13. The consensus mechanism is a first-seen rule: run through every transaction after the theft searching for the first transaction that puts money in the ACSA, and the resulting utxo is, by consensus, the recovery point
14. Return to step 5 and loop forever
15. Voila, the sidechain never dies and nodes can always come to consensus on its next block

# What works

Creating a spacechain works (yay!).

The governance mechanism seems to work. Spaceblocks are created by hashing some data and committing that hash to bitcoin's blockchain. Client software comes to consensus on the order of transactions by observing the order in which spaceblock hashes are committed. Spaceblocks themselves are distributed out of band through a seemingly peer to peer (but actually not) method: nostr dms.

The script address for anchor outputs seems to work. It's an anyone-can-spend address. Spacechain miners are expected to use fee bidding to compete with one another to move that utxo and, in the witness data of the transaction that moves it, commit the hash of a new spaceblock to bitcoin's blockchain. If someone steals the anchor output, that is detectable because the output won't go right back into the anyone-can-spend address (if it did, it wouldn't be a theft, because you could just keep building on top of it by moving it again), and at that point miners can bid to make a new anyone-can-spend utxo. The first anchor utxo to make it into the blockchain after a theft and commit a new spaceblock wins and is, by consensus, the new anchor utxo that everyone else must build on top of.

Fork resolution seems to work. If you request a block and someone sends you one that does not build on your latest one, your browser will try to resolve the fork by repeatedly requesting the parents, grandparents, and so forth of the contested blocks. If both have a direct lineage back to the genesis block, it is a legitimate fork, and which one to follow is decided first by which one is longer, and, if the chains are the same length, by which chain's anchor transaction comes first at the moment of the fork.

Issuing spacecoins seems to work. Anyone can issue spacecoins by burning sats, though my hope is that no one actually does this on mainnet (see the FAQ below). Every sat burned gets you a new spacecoin. There is a marketplace where users can peg into the spacechain by burning sats. The software seems to reliably detect the burned coins and appropriately display your balance in the built in wallet, but the wallet is often veeeerrrryyyyyy ssssssslllllloooowwwwww. It relies on a rate limited web api from blockchair to search for op_returns on bitcoin's blockchain, and to make it work at all, I had to make it queue the api queries and run through them slowly. Blockchair's service also randomly gives 504 errors to some requests so I have to deal with that too. Oh yeah, and it doesn't index op_returns until the transaction containing them gets confirmed, so if you burn some sats to fund an address, it won't seem like anything happened until the transaction confirms (but even then, give it a few minutes, because the api calls themselves are very slow).

Sending and receiving spacecoins seems to work, with the above caveat that the wallet is incredibly slow to detect these transfers, just like it's slow to detect issuance op_returns. Even *after* a transaction gets confirmed and is visible on the built in spaceblock explorer, the wallet probably won't detect it for several minutes, because it's busy waiting for its turn in a queue of api calls.

The built in block explorer works. I am pretty proud of it, actually. It does almost everything I want it to. I think it's a very cool feature of this software that it comes with a built in block explorer.

# What doesn't work

A lot of stuff involving the built in wallet is super wonky or doesn't work at all. It doesn't have a button for importing an existing wallet, so good luck trying to do that. The wallet tries to broadcast transactions to other miners but they don't currently do anything in response, like mine them. Users can mine their own transactions, but that leads directly to the next problem:

Any sort of competitive mining, such as two people trying to mine a spaceblock in the same bitcoin block, is totally borked. One of the problems is that the mining software assumes you are the only one mining. When you go onto the mining page, it checks what the current blockheight is, and tries to build a block on top of the latest block you have, but it does not detect new blocks that might come in while you are doing that. So if you aren't the only miner, you might waste time trying to mine on top of an old block even after a new block has been mined by someone else. 

The mining software also doesn't reset your transaction data when a new block comes in, so if you don't do it manually (by typing esoteric commands from the source code into your browser console, or by refreshing the page), your browser might try to mine transactions that have already been mined. As a temporary workaround, you can refresh your browser after every new block and that will clear your cached transaction data.

I'm not running an always-on miner so if you click to view the first spacechain (i.e. my spacechain), you'll have trouble finding any blocks when I'm not running tests. Miners are the only ones who serve blocks right now so you can't really get them from somewhere else. I did put the first 10 or so blocks of the first spacechain into the source code, but good luck reading the source code to figure out how to make your browser recognize them as spaceblocks.

In the marketplace, you cannot yet make your own offers. To enable that I want to make a new opcode that makes a spacechain transaction valid if and only if a utxo is created on bitcoin before a certain blockheight and sends a specified amount of sats to a specified destination. That will allow non-interactive atomic swaps between bitcoin and the spacechain, which people can use to easily list spacecoins for sale and then leave the spacechain whenever someone takes their offer.

A bunch of other stuff is broken too but that's a long enough list for now.

# FAQ

## Don't spacechains burn bitcoins? Are they an attack on bitcoin?

Spacechains do burn bitcoins but I don't think they are an attack on bitcoin. I also don't recommend burning bitcoins on mainnet, because you'll probably never get back everything you burned. It's fine on testnet though.

## Why did you make a thing that burns bitcoins if you don't recommend it?

Lack of self control. When I originally heard about spacechains I was like, "Burn bitcoins? That's a dumb idea," and then I started thinking up reasons why it's dumb, and then I wondered, "How would you even *do* that?" so I considered it a few minutes and realized how it might be done, and then I was done for: I had to try it and see if it would work. And it does work, but that doesn't make it a good idea.

## I oppose this because it burns bitcoins

That's fine, you can oppose it. I think a few things are worth pointing out in its defense:

(1) This is only on *testnet.* It's meant as a technical demonstration of one way to do permissionless sidechains without a soft fork. I don't want people to actually burn real bitcoins, as in, on mainnet. But the testnet is precisely for experiments like this. I have no problem with burning testnet bitcoins because stuff like that is precisely what they exist for.

(2) Even if someone does this on mainnet, what will probably happen is this: spacechain creators will do it to issue an initial supply of coins on their spacechain, but after that, there's no reason anyone else has to do it. Other people can just buy some of the intial supply and use those. The option to make your own spacecoins (through burning) is just there to prevent the people who hold the initial supply from charging too much for them. If they ever charge more than 1 sat per spacecoin, you can just make your own spacecoins. Having that option keeps the whole thing permissionless.

(3) You can get rid of the bitcoin burning part by issuing some other token in a trusted manner. For example, tether corp could create a spacechain whose only token is tethers, and they wouldn't have to burn any bitcoins to do that. But this idea is probably even worse than burning bitcoins because it means literally creating an altcoin. (BTW the stacks blockchain is basically a spacechain with their own premined token, so this idea of "issue an altcoin on a spacechain so you don't have to burn any bitcoins" isn't even new. It's just worse than anything anyone ever invented before.)

(4) This is actually the same thing as #3 but one potentially-interesting *type* of token you could issue on a spacechain is a federated bitcoin-pegged token, similar to liquid bitcoin or rootstock bitcoin. This might be cool to do because a lot of bitcoiners, including myself, don't trust the federations that control liquid and rootstock. But what if you made *your own* liquid or *your own* rootstock where *you* are part of the federation in control? If you do that, you're placing more trust in yourself, which seems potentially appealing. At some point I want to add an option to the spacechain launcher to create a federated peg in this manner.

(5) The reason why I *personally* think burning bitcoins is dumb is because I don't see any meaningful value I can get in compensation for it. If a spacechain really offered some feature that I thought was worth $10, I would be willing to burn up to $10 in bitcoin to get that feature, because that just seems like common sense to me. The exact method I use to purchase something doesn't matter to me that much, whether it's sending bitcoins to a burn address or to someone who is selling the thing I want. So if someone comes up with a really compelling use case for spacechains (which I doubt will happen) I might burn a few sats to use it, if that's the cheapest way to use the feature. So I don't think burning bitcoins is *inherently* dumb, I just doubt anyone will come up with anything useful enough to justify it according to my personal preferences.
