## OSINT CTF Challenge

Part of SMTP 2025

Written by fredora

## Place

Description

```
During a trip I visited a small but lovely European town. What is the name of the town?

Format: town name, all lowercase, wrapped in SMT2025{ ... }
Example: SMT2025{middlesbrough}
```

![place](/smt-2025/images/place.png)

This is the image provided for the challenge. Doing a reverse image search will return an overview by Google that it's located in Regensberg, Switzerland. 

![place](/smt-2025/images/regensberg-1.png)

If you hate how AI is everywhere and wish it's not pushed down your throat this much (same tbh), don't worry, you can still solve this challenge without AI. On the Visual Match tab, you can find multiple images referencing the village of Regensberg.

![place](/smt-2025/images/regensberg-2.png)

Flag: SMT2025{regensberg}

## Online

Description

```
My friend found an interesting website online, but he hides the domain from me! Can you help me find it?

Format: domain wrapped in SMT2025{ ... }
Example: SMT2025{google.com}
```

Given this image:

![website](/smt-2025/images/website.jpg)

This is a perfect opportunity to flex your Google Dorking muscles. My strategy here is to find a website that contains any unique string inside the table (I use Product Name and the t CO23/t thing), and its title being "LCA Food Database". One of the queries can be something like this:

```
"Product Name" "t CO2e/t" intitle:"LCA Food Database"
```

Found the website that way, so flag: SMT2025{thebigclimatedatabase.com}

## The Beauty Of Arrakis

Description

```
A mysterious Ethereum whale has moved tens of thousands of ETH over the years. One day, they transferred a huge amount of ETH in a single transaction. Find the transaction hash!

Whale Wallet:
0x742d35Cc6634C0532925a3b844Bc454e4438f44e


Flag format: SMT2025{transactionhash}
```

This is where the challenges went 180 from your usual OSINT challenges to investigating cryptocurrency transaction. I search for a simple blockchain explorer and paste in the wallet address to find transaction history. This is how I found out that the wallet is mostly used for ETH (Ether) transactions and I should probably be using a dedicated scanner for the ETH blockchain for better and easier analysis.

![wallet](/smt-2025/images/eth-1.png)

I'm using [etherscan.io](https://etherscan.io/) to analyze the ETH wallet. They were NOT kidding about the whale status of this address. It's been used for years and it has thousands of transactions.

![wallet](/smt-2025/images/eth-2.png)

To solve this challenge in an efficient manner, you need to understand the key things you're asked to do: 

```
1. Wallet did a massive transfer (outgoing transcation) one day
2. Find the transcation hash of that transfer
```

We can filter for incoming/outgoing transaction, but it's still hard to scrape information because the wallet owner did a lot of incoming and outgoing transaction, both in massive amount of money. However, I noticed that the values of outgoing transactions are very small in comparison with incoming transactions. (I switched to [oklink.com](https://oklink.com/) because it has more intuitive filtering technique for a newbie in blockchain analysis like me):

![wallet](/smt-2025/images/eth-3.png)

100,000 is just a lucky guess because it's a huge wallet so I suppose most of its transaction will exceed that. That's how I filter for "huge transfers." Filtering for only `From <address value>` will filter for outgoing transaction, which will filter the transaction history even further, making it easier for us to search manually.

![wallet](/smt-2025/images/eth-4.png)

We got from 2K transactions, to 52, and then to only 29. At this point I found this particular [transaction.](https://www.oklink.com/ethereum/tx/0xc868d5bfd0ecf0492fecdb9bc897da0d20d8a3b19bc7843b1e1151349a6a5407)

Flag: SMT2025{0xc868d5bfd0ecf0492fecdb9bc897da0d20d8a3b19bc7843b1e1151349a6a5407}

## 007 - OREO

Description

```
Your agent in the field, known only as Mr. Oreo, has gone dark.

Before disappearing, he left behind a clue, a digital breadcrumb hidden in plain sight. According to a tip received by our intelligence team, Mr. Oreo minted an NFT on the Sepolia testnet (Ethereum) earlier this July. Somewhere in that token lies the flag, a vital piece of information that could compromise an entire operation.

Mr. Oreo's wallet:
0xEDFfD5AEc7f8f1b9E112DD12C04507359A578d47

No timestamp, no direct message, just this wallet and a trail of on-chain activity. Your mission is to investigate the wallet, identify any NFTs minted in July 2025, and recover the hidden flag.

Mr. Oreo trusted that only the sharpest eyes would find his message. Don’t let him down.
```

This might be intimidating for first timers, but the description really gave a lot of helpful hints. Let's break down the key things that help us:

```
1. Mr Oreo minted an NFT on Sepolio test net chain
2. The NFT was minted somewhere in July
```

Using etherscan to scan the Sepolio test net, supplying the address, and filtering for NFT we got this information about the wallet:

![nft](/smt-2025/images/nft-1.png)

I searched everywhere, from the contract to its source code looking for a hint of a flag, but turns out the answer is on the transaction detail. 

![nft](/smt-2025/images/nft-2.png)

Clicking on more detail will reveal an ipfs link that has the Token URI:

![nft](/smt-2025/images/nft-3.png)

There were 3 minting processes on the transaction, all three with different URI's. The second transaction will return this interesting json:

```
{
  "name": "Mr. Oreo 007",
  "description": "Mr. Oreo — silent but clever.\nHey hacker, my CID: bafkreibwjrsqy36vy23typkpaaj4cf74dfwohhkgfqtadxljkffzf74ppm",
  "image": "ipfs://bafybeibdyrgco64euwzygkyikx4k7ucxgj3volgwi3qfn3vfumvvxocssy",
  "attributes": [
    {
      "trait_type": "Alias",
      "value": "Agent Oreo"
    },
    {
      "trait_type": "Mood",
      "value": "Infiltrative"
    },
    {
      "trait_type": "Background",
      "value": "Moonlight Shadows"
    },
    {
      "trait_type": "Gear",
      "value": "Night Vision"
    }
  ]
}

i visited this using ipfs.io/ipfs/<CID>
```

If you visit the CID, you will find:

```
{
    "Description": "Congratulations! Your flag: SMT2025{blockchain_is_cool}"
}
```

P.S Funny challenge, I solved this 10 minutes BEFORE it's due but whatever. 