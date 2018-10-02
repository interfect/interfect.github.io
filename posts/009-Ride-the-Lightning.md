# Ride the Lightning: How to use the Lightning Network on the Bitcoin Mainnet

With the [recent activation of Segregated Witness (SegWit)](https://www.reddit.com/r/Bitcoin/comments/6vnqu9/boom_segwit_activated_this_is_gentlemen/) on the main Bitcoin blockchain, everyone has been so hyped for the [Lightning Network](https://lightning.network/) and its trustless off-chain "payment channels" that the Bitcoin price has hit record highs.

However, actual Lightning Network usage has been a bit thin on the ground. The [peer-to-peer protocol has been specced out](https://github.com/lightningnetwork/lightning-rfc/blob/master/00-introduction.md), and implementations such as [c-lightning](https://github.com/ElementsProject/lightning), [lnd](https://github.com/lightningnetwork/lnd), and [lit](https://github.com/mit-dci/lit) have been written, but users and network nodes are scarce.

This post will explain how to ignore the "do not use these with real money!" warnings and deploy a Lightning Network node on the Bitcoin mainnet.

## Step 0: Get Docker

For this writeup, I'm using [Docker](https://docker.com/) to run all the relevant software, because building things from source is hard and I'm lazy. If you have a recent Ubuntu machine and have Docker already set up, you should be good. Otherwise:

```
# install Docker
sudo apt install docker
# Add yourself to the Docker group for sudo-less Docker commands
sudo usermod -aG docker $USER
# Make sure to log out and in again for group membership to refresh!
```

## Step 1: Get Bitcoin

I already happened to have a Docker-based Bitcoin full node set up, but you probably don't. To set one up, first make a directory for where you want your Bitcoin config and blockchain data to live. I keep mine on a special `scratch` mount:

```
sudo mkdir -p /scratch/bitcoin/mainnet/bitcoind
```

Then, start up one of the excellent Bitcoin builds provided by [@amacneil](https://github.com/amacneil/docker-bitcoin):

```
docker run --rm --name bitcoind_mainnet -v /scratch/bitcoin/mainnet/bitcoind:/data -p 8333:8333 -p 9735:9735 amacneil/bitcoin:latest
```

This command, on the first run, will automatically download the Bitcoin image. On subsequent runs, it will use the local image. We needed to mount our config and chain data storage directory with `-v`, and expose ports 8333 (used by Bitcoin's peer-to-peer network) and 9735 (used by the Lightning Network's peer-to-peer network) on the host. (We're later going to be dumping the Lightning node into the same virtual network host as the Bitcoin node, so we have to forward the Lightning port now.)

At this point, you might want to set up port forwards on your router for these ports, and punch any necessary holes in your firewall.

### Add some Security

By default, the running `bitcoind` is listening on port 8332 with no IP whitelist, a username of `bitcoin`, and a password of `password`. The only security comes from Docker not exposing port 8332 to the outside world. If this makes you feel nervous, edit `/scratch/bitcoin/mainnet/bitcoind/bitcoin.conf` and set a real `rpcpassword` and some safe file permissions, and then stop your `bitcoind` (with a `Ctrl+C` in the window running the command, or a `docker stop bitcoind_mainnet` in another terminal) and restart it.

### Wrap `bitcoin-cli`

Bitcoin Core's `bitcoind` comes with a useful `bitcoin-cli` tool. I use this script, saved as `bitcoin-cli` on my `$PATH`, to give me CLI access to my Dockerized Bitcoin node:

```
#!/usr/bin/env bash
docker run --rm --network container:bitcoind_mainnet -v /scratch/bitcoin/mainnet/bitcoind:/data amacneil/bitcoin:latest bitcoin-cli "$@"
```

This runs another Docker container from the same `amacneil/bitcoin:latest` image, but drops it into the network namespace of the running `bitcoind_mainnet` container, so they share the same `localhost` and the RPC client can talk to the RPC server. It also exposes the same data directory, so that the client can read the RPC password if one is set. It forwards all the scripts arguments to the containerized `bitcoin-cli`, so that it will do whatever you told it to do.

Once that script is saved somewhere and marked executable, try it out with something like `bitcoin-cli getinfo`. You should get a nice informative response like:

```
{
  "version": 140200,
  "protocolversion": 70015,
  "walletversion": 130000,
  "balance": 0.00000000,
  "blocks": 483425,
  "timeoffset": 0,
  "connections": 8,
  "proxy": "",
  "difficulty": 888171856257.3206,
  "testnet": false,
  "keypoololdest": 1503625917,
  "keypoolsize": 100,
  "paytxfee": 0.00000000,
  "relayfee": 0.00001000,
  "errors": ""
}
```

If you see something like that, your Dockerized `bitcoind` is working correctly, and you can move on to...

## Step 2: Get Lightning

Since I was already running `bitcoind`, when I went looking for a Lightning node I wanted one that supported `bitcoind` as a backend. Since [lnd](https://github.com/lightningnetwork/lnd) and [lit](https://github.com/mit-dci/lit) only support the Go-based [btcd](https://github.com/btcsuite/btcd) and rely on its own particular RPC calls and interfaces that `bitcoind` doesn't have, my only real option was [c-lightning](https://github.com/ElementsProject/lightning)'s `lightningd`. I'm not sure that c-lightning is the Right Choice in general for a Lightning node, since lnd seems to be getting more attention, but it works with the software stack I have and is written in a language I speak, so it's what I chose.

First, we need to make a directory for the Lightning Network state files. This holds your node's database of its open payment channels, as well as the filesystem socket that the user interface uses to communicate with the node's backend.

```
sudo mkdir -p /scratch/bitcoin/mainnet/clightning
```

Then, you can run this super long command to download and run `lightningd`:
    
```
docker run --rm --name lightning --network container:bitcoind_mainnet -v /scratch/bitcoin/mainnet/bitcoind:/root/.bitcoin -v /scratch/bitcoin/mainnet/clightning:/root/.lightning --entrypoint /usr/bin/lightningd cdecker/lightningd:master --network=bitcoin --log-level=debug
```

We put this container in the same network namespace as the `bitcoind_mainnet` container, so they share a `localhost` and the Lightning daemon can talk to the Bitcoin daemon. We also expose the Bitcoin directory, so `lightningd` can read the `bitcoin.conf` file and get the RPC password, and the Lightning directory we just set up. We tell Docker to run `/usr/bin/lightningd` inside the container, and we tell `lightningd` that it will be running on the `bitcoin` mainnet (as opposed to `testnet` or `litecoin`), and to log a lot so we can see what it's up to.

The `lightningd` daemon should start up and say weird stuff like `Adding block` and `Ignoring chaintip`. I've no idea what it's talking about, but it means it's talking to `bitcoind` so it makes me happy.

Note that unlike with `bitcoind`, `lightningd` won't respond to a `Ctrl+C`. You can kill its container with `docker kill lightning`, but the Right Way to shut it down is with `lightning-cli`.

### Wrap `lightning-cli`

Like Bitcoin Core's `bitcoin-cli`, c-lightning has a `lightning-cli` command. Here's the script I use to wrap it so I can just type "lightning-cli" and talk to my Dockerized `lightningd`:

```
#!/usr/bin/env bash
docker run --rm -v /scratch/bitcoin/mainnet/clightning:/root/.lightning --entrypoint /usr/bin/lightning-cli cdecker/lightningd:master "$@"
```

You don't need `lightning-cli` to be in the same network namespace as the daemon, because it communicates through a filesystem socket in the `.lightning` directory instead.

Once you have this set up, run `lightning-cli getinfo`. You should see some statistics like these, and in particular the `blockheight` should agree with whatever `bitcoin-cli getinfo` reports:

```
{ "id" : "03bcf9c43644326a8888c6e061ca00da74f6660d5619964b381f4c95893b6ef1b0", "port" : 9735, "testnet" : false, "version" : "v0.5.2-2016-11-21-820-g5db4e11", "blockheight" : 483425 }
```

To stop your `lightningd` safely, use `lightning-cli stop`. To see all the commands, `lightning-cli help`.

## Step 3: Connect Nodes

In theory, now you can use your cool new Lightning Network node to connect to other nodes, open payment channels, and send Lightning Network transactions. In practice, I haven't done any of this yet, because I only have the one node.

If you are on [cjdns](https://github.com/cjdelisle/cjdns), you can connect to my node:

```
lightning-cli connect fc8f:1cb7:fedd:dfd2:fd9c:96a0:e54d:1467 9735 03bcf9c43644326a8888c6e061ca00da74f6660d5619964b381f4c95893b6ef1b0
```

However, I haven't funded it, so it won't actually be able to open any channels.

**UPDATE: This node no longer exists. However, there are many more nodes nowdays. Use one of those.**

If you aren't on cjdns, you can set up two nodes and connect one to the other using the IP and port, and the "id" value from `lightning-cli getinfo`. You can see if you are connected with `lightning-cli getpeers`.

## Step 4: Insert Coins

Getting money into `lightningd` is still hard. According to the [c-lightning docs](https://github.com/ElementsProject/lightning#opening-a-channel-on-the-bitcoin-testnet), you do something like:

1. Get an address for your Lightning node with `lightning-cli newaddr`.
2. Send funds to that address. Get the transaction ID.
3. Grab the raw transaction from the ID with `bitcoin-cli getrawtransaction <txid>`.
4. Inform `lightningd` it has money by passing it the raw transaction with `lightning-cli addfunds <rawtx>`

## Step 5: Open Channel

Once you are connected to a node and have some money in your `lightningd`, you need to open a payment channel to send money across. Make sure both nodes have some funds, and do:

```
lightning-cli fundchannel <node_id> <amount>
```

## Step 6: Create an Invoice

To receive a Lightning Network payment, you need to make an invoice. Use the `lightning-cli invoice` command, with an amount **in millisatoshis**, so 1000 times the amount in satoshis, and a name:

```
lightning-cli invoice 1000 testinvoice
```

You should get back something like:

```
{ "rhash" : "55f3e07f74110488cd8894e9af766ab731ba1b736cb7066cef7624ecaea70b77" }
```

You then need to send your node ID, the `rhash` value, and the amount to pay to the sender. There are cool Lightning Network encoding schemes to do this, but `lightningd` doesn't support any of them.

## Step 7: Pay the Invoice

Now, from another node, pay your invoice. This happens in two parts; the first is the calculation of a route, for which it helps to `sudo apt install jq` to be able to pull the route string out of `lightning-cli`'s JSON. You need to plug in the recipient's node ID and the amount (still in **millisatoshis**) to send along.

```
ROUTE="$(lightning-cli getroute <recipient_id> <amount> 1 | jq -r '.route' -)"
```

Then send the payment, using the `rhash` value from the invoice:

```
lightning-cli sendpay $route <rhash>
```

## Step 8: Profit!

To close your payment channel with your peer, and keep any funds you received:

```
lightning-cli close <node_id>
```

There's no way to know how much money your node has, but if you can figure it out, you can withdraw money **in normal satoshis** to a Bitcoin address:

```
lightning-cli withdraw <amount> <address>
```

## Conclusions

Overall, I'm not really satisfied with c-lightning. Being a command-line tool is fine, but it's missing some basic features, like noticing when it receives funds over normal Bitcoin transactions, or being able to report a total balance.

However, as soon as c-lightning implements some of these missing features, it should start working really well; it is at least easy to set up, and I think `bitcoind` is more popular than `btcd`.

Next, I'm going to look at `lnd` and its cool [Zap wallet](https://github.com/LN-Zap/zap-desktop), which looks much harder to get going, but much more usable once you have it.












