# Creating a Blockchain Fork with Geth

## Init the Genesis block

genesis.json:
```JSON
{
    "config": {
        "chainId": 15,
        "homesteadBlock": 0,
        "eip155Block": 0,
        "eip158Block": 0
    },
    "difficulty": "2000000",
    "gasLimit": "2100000",
    "alloc": {
        "0x9CA9d2D5E04012C9Ed24C0e513C9bfAa4A2dD77f": { "balance": "300000" },
        "0xDe554B6c292f5e5794A68Dc560a537DD89d3b03E": { "balance": "400000" }
    }
}
```

#### Node 1:
```bash
$ geth --datadir="/home/thomas/workspace/privategeth/01" init genesis.json
```

#### Node 2:
```bash
$ geth --datadir="/home/thomas/workspace/privategeth/02" init genesis.json
```
## Start the Nodes

Use the accounts from our [DevNet](https://github.com/smartcontractkit/devnet/tree/master/keys/SmartNet)

#### Node 1:
```bash
$ geth --datadir="/home/thomas/workspace/privategeth/01" --networkid 15 --nodiscover --port 30301 --targetgaslimit '9000000000000' --rpc --rpcaddr 127.0.0.1 --rpcport 8101 --rpcapi "eth,net,web3,admin,personal" --ws --wsport 8201 --wsorigins "*" --mine --minerthreads=1 --etherbase=0x9ca9d2d5e04012c9ed24c0e513c9bfaa4a2dd77f --maxpeers 10 console
```

#### Node 2:
```bash
$ geth --datadir="/home/thomas/workspace/privategeth/02" --networkid 15 --nodiscover --port 30302 --targetgaslimit '9000000000000' --rpc --rpcaddr 127.0.0.1 --rpcport 8102 --rpcapi "eth,net,web3,admin,personal" --ws --wsport 8202 --wsorigins "*" --mine --minerthreads=1 --etherbase=0xDe554B6c292f5e5794A68Dc560a537DD89d3b03E --maxpeers 10 console
```

## Get the `enode` Info

```
> admin.nodeInfo
```

### Create Static Nodes File

Copy the value of `enode:`, replacing "[::]" with your IP to a file named `static-nodes.json` at the root of your `--datadir` directory.

`static-nodes.json` file saved in "01" points to the `enode` address of "02":
```
[
  "enode://15950f60b7a6b6b8baf068f5f9bd5f8a5817ff0761f049a8347fca83c5deab9e203080635306edde71b8203145d1cea2f057cc5107dee7290da8bed3042c2c39@192.168.2.110:30302"
}
```

And the `static-nodes.json` file saved in "02" points to the `enode` address of "01":
```
[
 "enode://1e74c79ad0adfcefb8226e32054b19173c1e59af9db73e2d3434d6b74e302459b8104c5908f5d1ceff07b00e33b79af3901d74222ca886bd641f587912e1e322@192.168.2.110:30301"
]
```

Restart the nodes.

### Copy Keystores into `keystore` Directory

```bash
$ cp account0 /home/thomas/workspace/privategeth/01/keystore
$ cp account2 /home/thomas/workspace/privategeth/01/keystore
$ cp account1 /home/thomas/workspace/privategeth/02/keystore
$ cp account2 /home/thomas/workspace/privategeth/02/keystore
```

## Get Account Info

#### Run on each node:

```
> eth.accounts
# Node 1:
# ["0x9ca9d2d5e04012c9ed24c0e513c9bfaa4a2dd77f", "0xa5f108b6d82b9f080325f935f3baf89e0042cac8"]
# Node 2:
# ["0xde554b6c292f5e5794a68dc560a537dd89d3b03e", "0xa5f108b6d82b9f080325f935f3baf89e0042cac8"]
```

```
> web3.fromWei(eth.getBalance(eth.accounts[0]), "ether")
# Node 1:
# 43275.0000000000003
# Node 2:
# 7600.0000000000004
```

```
> web3.fromWei(eth.getBalance(eth.accounts[1]), "ether")
# Node 1:
# 0
# Node 2:
# 0
```

## Unlock Accounts:

#### Node 1:
```
> personal.unlockAccount("0x9ca9d2d5e04012c9ed24c0e513c9bfaa4a2dd77f", "T.tLHkcmwePT/p,]sYuntjwHKAsrhm#4eRs4LuKHwvHejWYAC2JP4M8HimwgmbaZ")
```

#### Node 2:
```
> personal.unlockAccount("0xde554b6c292f5e5794a68dc560a537dd89d3b03e", "o}Qd&Jg73pMnhCuN3hVgfbpXga(GPYgaHTAqXcFJWQaHi)qCTjFyfa4ca2ey#DJY")
```

### Test a Spend:
```
> eth.sendTransaction({from: eth.accounts[0], to: eth.accounts[1], value: web3.toWei(7, "ether")})
```

### Check balances again:
```
> web3.fromWei(eth.getBalance(eth.accounts[1]), "ether")
# Node 1:
# 7
# Node 2:
# 7
```

## Block Communication

```bash
$ sudo iptables -A INPUT -p tcp --dport 30301 -j DROP
$ sudo iptables -A INPUT -p tcp --dport 30302 -j DROP
```

### Spend Again

#### Node 1:
```
> eth.sendTransaction({from: eth.accounts[0], to: eth.accounts[1], value: web3.toWei(10, "ether")})
```

#### Node 2:
```
> eth.sendTransaction({from: eth.accounts[0], to: eth.accounts[1], value: web3.toWei(20, "ether")})
```

Wait for the transaction to be commited on each node.

```
> web3.fromWei(eth.getBalance(eth.accounts[1]), "ether")
# Node 1:
# 17
# Node 2:
# 27
```

## Unblock Communication

```bash
$ sudo iptables -D INPUT -p tcp --dport 30301 -j DROP
$ sudo iptables -D INPUT -p tcp --dport 30302 -j DROP
```

### Check Account Balance

```
> web3.fromWei(eth.getBalance(eth.accounts[1]), "ether")
# Node 1:
# 37
# Node 2:
# 37
```

## Geth Output:

#### Node 2:
```
INFO [04-30|11:00:26] Imported new chain segment               blocks=7    txs=0  mgas=0.000  elapsed=10.114ms  mgasps=0.000 number=10358 hash=dff85d…a2e4b3 cache=126.86kB ignored=6
INFO [04-30|11:00:26] Imported new chain segment               blocks=8    txs=1  mgas=0.021  elapsed=15.578ms  mgasps=1.348 number=10366 hash=638438…a3b6f8 cache=128.99kB
INFO [04-30|11:00:26] Commit new mining work                   number=10367 txs=1  uncles=0 elapsed=1.602ms
INFO [04-30|11:00:26] ⑂ block  became a side fork              number=10357 hash=5bba35…a0bd06
INFO [04-30|11:00:26] ⑂ block  became a side fork              number=10358 hash=1acf2e…631a1e
INFO [04-30|11:00:26] ⑂ block  became a side fork              number=10359 hash=d2efb1…8cbc27
INFO [04-30|11:00:26] ⑂ block  became a side fork              number=10360 hash=83f4fb…d2bda9
INFO [04-30|11:00:26] ⑂ block  became a side fork              number=10361 hash=a94000…56308e
> web3.fromWei(eth.getBalance(eth.accounts[1]), "ether")
17
INFO [04-30|11:00:47] Imported new chain segment               blocks=1    txs=0  mgas=0.000  elapsed=5.664ms   mgasps=0.000 number=10367 hash=c074da…b461f7 cache=128.87kB
INFO [04-30|11:00:47] Commit new mining work                   number=10368 txs=1  uncles=0 elapsed=524.329µs
INFO [04-30|11:01:00] Imported new chain segment               blocks=1    txs=1  mgas=0.021  elapsed=5.365ms   mgasps=3.914 number=10368 hash=d498a1…7fd7ce cache=129.32kB
INFO [04-30|11:01:00] Commit new mining work                   number=10369 txs=0  uncles=0 elapsed=185.568µs
> web3.fromWei(eth.getBalance(eth.accounts[1]), "ether")
37
```

Output with `debug.verbosity(6)` (look for "Chain split detected"):

```
INFO [04-30|11:12:56] 🔗 block reached canonical chain          number=10432 hash=649e40…737e38
INFO [04-30|11:12:56] 🔨 mined potential block                  number=10437 hash=03a639…5045dc
TRACE[04-30|11:12:56] Propagated block                         hash=03a639…5045dc recipients=0 duration=2562047h47m16.854s
TRACE[04-30|11:12:56] Announced block                          hash=03a639…5045dc recipients=0 duration=2562047h47m16.854s
DEBUG[04-30|11:12:56] Reinjecting stale transactions           count=0
INFO [04-30|11:12:56] Commit new mining work                   number=10438 txs=0  uncles=0 elapsed=210.585µs
TRACE[04-30|11:12:56] Started ethash search for new nonces     miner=0 seed=6580529614196254626
DEBUG[04-30|11:12:57] Recalculated downloader QoS values       rtt=19.525409591s confidence=1.000 ttl=58.576228773s
TRACE[04-30|11:13:02] Dial task done                           task="wait for dial hist expire (29.999995114s)"
TRACE[04-30|11:13:02] New dial task                            task="staticdial 1e74c79ad0adfcef 192.168.2.110:30301"
DEBUG[04-30|11:13:02] Adding p2p peer                          name=Geth/v1.8.6-stable-1... addr=192.168.2.110:30301 peers=1
TRACE[04-30|11:13:02] connection set up                        id=1e74c79ad0adfcef addr=192.168.2.110:30301 conn=staticdial inbound=false
TRACE[04-30|11:13:02] Dial task done                           task="staticdial 1e74c79ad0adfcef 192.168.2.110:30301"
TRACE[04-30|11:13:02] New dial task                            task="wait for dial hist expire (29.999997641s)"
TRACE[04-30|11:13:02] Starting protocol eth/63                 id=1e74c79ad0adfcef conn=staticdial
DEBUG[04-30|11:13:02] Ethereum peer connected                  id=1e74c79ad0adfcef conn=staticdial name=Geth/v1.8.6-stable-12683fec/linux-amd64/go1.10
TRACE[04-30|11:13:02] Registering sync peer                    peer=1e74c79ad0adfcef
DEBUG[04-30|11:13:06] Synchronising with the network           peer=1e74c79ad0adfcef eth=63 head=004d95…16e8b4 td=9435133923 mode=full
DEBUG[04-30|11:13:06] Retrieving remote chain height           peer=1e74c79ad0adfcef
DEBUG[04-30|11:13:06] Fetching batch of headers                id=1e74c79ad0adfcef conn=staticdial count=1 fromhash=004d95…16e8b4 skip=0 reverse=false
TRACE[04-30|11:13:06] Filtering headers                        peer=1e74c79ad0adfcef headers=1
DEBUG[04-30|11:13:06] Remote head header identified            peer=1e74c79ad0adfcef number=10440 hash=004d95…16e8b4
DEBUG[04-30|11:13:06] Looking for common ancestor              peer=1e74c79ad0adfcef local=10437 remote=10440
DEBUG[04-30|11:13:06] Fetching batch of headers                id=1e74c79ad0adfcef conn=staticdial count=13 fromnum=10245 skip=15 reverse=false
DEBUG[04-30|11:13:06] Found common ancestor                    peer=1e74c79ad0adfcef number=10421 hash=91661b…b520e8
DEBUG[04-30|11:13:06] Directing header downloads               peer=1e74c79ad0adfcef origin=10422
DEBUG[04-30|11:13:06] Downloading block bodies                 origin=10422
TRACE[04-30|11:13:06] Fetching skeleton headers                peer=1e74c79ad0adfcef count=192 from=10422
DEBUG[04-30|11:13:06] Fetching batch of headers                id=1e74c79ad0adfcef conn=staticdial count=128 fromnum=10613 skip=191 reverse=false
DEBUG[04-30|11:13:06] Downloading transaction receipts         origin=10422
TRACE[04-30|11:13:06] Fetching full headers                    peer=1e74c79ad0adfcef count=192 from=10422
DEBUG[04-30|11:13:06] Fetching batch of headers                id=1e74c79ad0adfcef conn=staticdial count=192 fromnum=10422 skip=0   reverse=false
TRACE[04-30|11:13:06] Scheduling new headers                   peer=1e74c79ad0adfcef count=19  from=10422
TRACE[04-30|11:13:06] Fetching full headers                    peer=1e74c79ad0adfcef count=192 from=10441
DEBUG[04-30|11:13:06] Fetching batch of headers                id=1e74c79ad0adfcef conn=staticdial count=192 fromnum=10441 skip=0   reverse=false
DEBUG[04-30|11:13:06] No more headers available                peer=1e74c79ad0adfcef
DEBUG[04-30|11:13:06] Header download terminated               peer=1e74c79ad0adfcef
TRACE[04-30|11:13:06] Requesting new batch of data             peer=1e74c79ad0adfcef type=bodies count=1   from=10436
DEBUG[04-30|11:13:06] Fetching batch of block bodies           id=1e74c79ad0adfcef conn=staticdial count=1
DEBUG[04-30|11:13:06] Inserting downloaded chain               items=14 firstnum=10422 firsthash=a548cf…6a40c8 lastnum=10435 lasthash=6e21d9…eddfbf
DEBUG[04-30|11:13:06] Data fetching completed                  type=receipts
DEBUG[04-30|11:13:06] Transaction receipt download terminated  err=nil
TRACE[04-30|11:13:06] Filtering bodies                         peer=1e74c79ad0adfcef txs=1  uncles=1
TRACE[04-30|11:13:06] Peer throughput measurements updated     peer=1e74c79ad0adfcef hps=0.000 bps=286.700 rps=0.000 sps=0.000 miss=0 rtt=18.000034879s
TRACE[04-30|11:13:06] Delivered new batch of data              peer=1e74c79ad0adfcef type=bodies   count=1:1
DEBUG[04-30|11:13:06] Data fetching completed                  type=bodies
DEBUG[04-30|11:13:06] Block body download terminated           err=nil
DEBUG[04-30|11:13:06] Trie cache stats after commit            misses=0 unloads=0
DEBUG[04-30|11:13:06] Inserted forked block                    number=10435 hash=6e21d9…eddfbf diff=1108347 elapsed=4.136ms   txs=0  gas=0 uncles=0
INFO [04-30|11:13:06] Imported new chain segment               blocks=1    txs=0  mgas=0.000  elapsed=4.430ms   mgasps=0.000 number=10435 hash=6e21d9…eddfbf cache=129.92kB ignored=13
DEBUG[04-30|11:13:06] Inserting downloaded chain               items=5  firstnum=10436 firsthash=b612e7…7d6ed1 lastnum=10440 lasthash=004d95…16e8b4
DEBUG[04-30|11:13:06] Trie cache stats after commit            misses=0 unloads=0
DEBUG[04-30|11:13:06] Inserted forked block                    number=10436 hash=b612e7…7d6ed1 diff=1105642 elapsed=5.661ms   txs=1  gas=21000 uncles=0
DEBUG[04-30|11:13:06] Trie cache stats after commit            misses=0 unloads=0
DEBUG[04-30|11:13:06] Chain split detected                     number=10434 hash=4c2a0a…d294bd drop=3 dropfrom=03a639…5045dc add=3 addfrom=fbe9df…690013
DEBUG[04-30|11:13:06] Inserted new block                       number=10437 hash=fbe9df…690013 uncles=0 txs=0  gas=0     elapsed=586.59µs
DEBUG[04-30|11:13:06] Trie cache stats after commit            misses=0 unloads=0
DEBUG[04-30|11:13:06] Dereferenced trie from memory database   nodes=2 size=649.00B time=9.987µs  gcnodes=21310 gcsize=5.42mB gctime=59.736862ms livenodes=422 livesize=129.87kB
DEBUG[04-30|11:13:06] Inserted new block                       number=10438 hash=6f7c29…358911 uncles=0 txs=0  gas=0     elapsed=372.737µs
DEBUG[04-30|11:13:06] Trie cache stats after commit            misses=0 unloads=0
DEBUG[04-30|11:13:06] Dereferenced trie from memory database   nodes=3 size=764.00B time=10.917µs gcnodes=21313 gcsize=5.42mB gctime=59.747651ms livenodes=421 livesize=129.75kB
DEBUG[04-30|11:13:06] Inserted new block                       number=10439 hash=16fbd2…9159b1 uncles=0 txs=0  gas=0     elapsed=339.009µs
DEBUG[04-30|11:13:06] Trie cache stats after commit            misses=0 unloads=0
DEBUG[04-30|11:13:06] Dereferenced trie from memory database   nodes=2 size=649.00B time=9.439µs  gcnodes=21315 gcsize=5.42mB gctime=59.756979ms livenodes=421 livesize=129.75kB
DEBUG[04-30|11:13:06] Inserted new block                       number=10440 hash=004d95…16e8b4 uncles=0 txs=0  gas=0     elapsed=335.924µs
INFO [04-30|11:13:06] Imported new chain segment               blocks=5    txs=1  mgas=0.021  elapsed=7.475ms   mgasps=2.809 number=10440 hash=004d95…16e8b4 cache=131.40kB
DEBUG[04-30|11:13:06] Synchronisation terminated               elapsed=14.946165ms
TRACE[04-30|11:13:06] Announced block                          hash=004d95…16e8b4 recipients=1 duration=2562047h47m16.854s
TRACE[04-30|11:13:06] Bad uncle found and will be removed      hash=b612e7…7d6ed1
TRACE[04-30|11:13:06] &{0xc43a71f440 [] [0xc4241e6000] {[182 18 231 189 81 107 185 87 36 99 200 218 51 190 124 38 110 168 132 178 198 115 252 143 20 91 64 114 222 125 110 209]} {<nil>} <nil> 0001-01-01 00:00:00 +0000 UTC <nil>} 
TRACE[04-30|11:13:06] Bad uncle found and will be removed      hash=03a639…5045dc
TRACE[04-30|11:13:06] &{0xc43c1b0000 [] [] {[3 166 57 208 151 16 122 23 246 95 218 160 163 43 206 77 189 212 146 139 231 151 94 38 143 239 171 26 235 80 69 220]} {<nil>} <nil> 0001-01-01 00:00:00 +0000 UTC <nil>} 
TRACE[04-30|11:13:06] Bad uncle found and will be removed      hash=6e21d9…eddfbf
TRACE[04-30|11:13:06] &{0xc42849c000 [] [] {[110 33 217 206 227 233 43 79 156 232 195 184 35 27 77 168 180 46 119 135 229 23 56 183 32 147 176 158 29 237 223 191]} {<nil>} <nil> 0001-01-01 00:00:00 +0000 UTC <nil>} 
INFO [04-30|11:13:06] Commit new mining work                   number=10441 txs=0  uncles=0 elapsed=287.566µs
INFO [04-30|11:13:06] ⑂ block  became a side fork              number=10435 hash=74c37d…0dbc51
DEBUG[04-30|11:13:06] Reinjecting stale transactions           count=1
TRACE[04-30|11:13:06] Ethash nonce search aborted              miner=0 attempts=600057
TRACE[04-30|11:13:06] Started ethash search for new nonces     miner=0 seed=2058436327686366548
TRACE[04-30|11:13:06] Pooled new future transaction            hash=a3a936…492e12 from=0xDe554B6c292f5e5794A68Dc560a537DD89d3b03E to=0xa5F108b6d82b9F080325F935F3BAf89e0042CAC8
TRACE[04-30|11:13:06] Promoting queued transaction             hash=a3a936…492e12
TRACE[04-30|11:13:06] Broadcast transaction                    hash=a3a936…492e12 recipients=1
TRACE[04-30|11:13:14] Ethash nonce found and reported          miner=0 attempts=451812  nonce=2058436327686818360
INFO [04-30|11:13:14] Successfully sealed new block            number=10441 hash=0eb789…b7d82b
DEBUG[04-30|11:13:14] Trie cache stats after commit            misses=0 unloads=0
DEBUG[04-30|11:13:14] Dereferenced trie from memory database   nodes=2 size=649.00B time=11.818µs gcnodes=21317 gcsize=5.42mB gctime=59.76855ms  livenodes=420 livesize=129.60kB
INFO [04-30|11:13:14] ⑂ block  became a side fork              number=10436 hash=3a7f1e…125b7a
INFO [04-30|11:13:14] 🔨 mined potential block                  number=10441 hash=0eb789…b7d82b
DEBUG[04-30|11:13:14] Reinjecting stale transactions           count=0
TRACE[04-30|11:13:14] Propagated block                         hash=0eb789…b7d82b recipients=1 duration=2562047h47m16.854s
TRACE[04-30|11:13:14] Announced block                          hash=0eb789…b7d82b recipients=0 duration=2562047h47m16.854s
TRACE[04-30|11:13:14] Bad uncle found and will be removed      hash=3a7f1e…125b7a
TRACE[04-30|11:13:14] &{0xc438d23d40 [] [0xc423e54a20] {[58 127 30 10 100 202 149 53 24 4 33 149 219 84 94 68 91 65 200 182 42 213 13 181 105 176 5 43 42 18 91 122]} {<nil>} <nil> 0001-01-01 00:00:00 +0000 UTC <nil>} 
TRACE[04-30|11:13:14] Bad uncle found and will be removed      hash=74c37d…0dbc51
TRACE[04-30|11:13:14] &{0xc42b303d40 [] [] {[116 195 125 161 186 215 127 141 86 93 230 211 72 118 7 205 185 97 26 196 176 37 29 55 251 179 70 138 187 13 188 81]} {<nil>} <nil> 0001-01-01 00:00:00 +0000 UTC <nil>} 
INFO [04-30|11:13:14] Commit new mining work                   number=10442 txs=1  uncles=0 elapsed=497.639µs
TRACE[04-30|11:13:14] Started ethash search for new nonces     miner=0 seed=1510126900379595032
DEBUG[04-30|11:13:14] Transaction pool status report           executable=1 queued=0 stales=0
DEBUG[04-30|11:13:16] Queued propagated block                  peer=1e74c79ad0adfcef number=10442 hash=a1c3df…f401a7 queued=1
DEBUG[04-30|11:13:16] Importing propagated block               peer=1e74c79ad0adfcef number=10442 hash=a1c3df…f401a7
TRACE[04-30|11:13:16] Propagated block                         hash=a1c3df…f401a7 recipients=0 duration=4.180ms
DEBUG[04-30|11:13:16] Trie cache stats after commit            misses=0 unloads=0
DEBUG[04-30|11:13:16] Dereferenced trie from memory database   nodes=2 size=649.00B time=9.221µs  gcnodes=21319 gcsize=5.42mB gctime=59.777644ms livenodes=424 livesize=130.16kB
DEBUG[04-30|11:13:16] Inserted new block                       number=10442 hash=a1c3df…f401a7 uncles=0 txs=1  gas=21000 elapsed=4.889ms
INFO [04-30|11:13:16] Imported new chain segment               blocks=1    txs=1  mgas=0.021  elapsed=4.929ms     mgasps=4.260 number=10442 hash=a1c3df…f401a7 cache=131.81kB
DEBUG[04-30|11:13:16] Reinjecting stale transactions           count=0
TRACE[04-30|11:13:16] Removed old pending transaction          hash=a3a936…492e12
TRACE[04-30|11:13:16] Announced block                          hash=a1c3df…f401a7 recipients=0 duration=9.200ms
INFO [04-30|11:13:16] Commit new mining work                   number=10443 txs=0  uncles=0 elapsed=145.779µs
INFO [04-30|11:13:16] ⑂ block  became a side fork              number=10437 hash=03a639…5045dc
TRACE[04-30|11:13:16] Ethash nonce search aborted              miner=0 attempts=115162
TRACE[04-30|11:13:16] Started ethash search for new nonces     miner=0 seed=1478955644019021370
DEBUG[04-30|11:13:17] Recalculated downloader QoS values       rtt=19.144065913s confidence=1.000 ttl=57.432197739s
> web3.fromWei(eth.getBalance(eth.accounts[1]), "ether")
TRACE[04-30|11:13:21]                                          msg="sending {\"jsonrpc\":\"2.0\",\"id\":149,\"method\":\"eth_accounts\",\"params\":[]}"
TRACE[04-30|11:13:21]                                          msg="<-readResp: response {\"jsonrpc\":\"2.0\",\"id\":149,\"result\":[\"0xde554b6c292f5e5794a68dc560a537dd89d3b03e\",\"0xa5f108b6d82b9f080325f935f3baf89e0042cac8\"]}"
TRACE[04-30|11:13:21]                                          msg="sending {\"jsonrpc\":\"2.0\",\"id\":150,\"method\":\"eth_getBalance\",\"params\":[\"0xa5f108b6d82b9f080325f935f3baf89e0042cac8\",\"latest\"]}"
TRACE[04-30|11:13:21]                                          msg="<-readResp: response {\"jsonrpc\":\"2.0\",\"id\":150,\"result\":\"0x3a1cfd104cf2c0000\"}"
67
```

## Testing Double-Spend

[Block communication](#block-communication)

Get the balance of account 1 on both nodes:
```
> web3.fromWei(eth.getBalance(eth.accounts[1]), "ether")
```

Unlock Account 1 on both nodes:

```
> personal.unlockAccount("0xa5F108b6d82b9F080325F935F3BAf89e0042CAC8", "axC$3yCm6dmTykrkMtbPQs#zDihtsGQMhmfT$dub4Ytxu%G3jHKXzZMGE.6GbXes")
```

Send the balance of account 1 to account 0:

```
> eth.sendTransaction({from: eth.accounts[1], to: eth.accounts[0], value: web3.toWei(66, "ether")})
```

Get the balance of account 1

```
> web3.fromWei(eth.getBalance(eth.accounts[1]), "ether")
# Node 1:
# 1.999244
# Node 2:
# 1.999244
```

When it's spent, get the balance of account 0

```
> web3.fromWei(eth.getBalance(eth.accounts[0]), "ether")
# Node 1:
# 44685.0015120000003
# Node 2:
# 8467.9996220000004
```

[Unblock communication](#unblock-communication)

Get the balance of account 0

```
> web3.fromWei(eth.getBalance(eth.accounts[0]), "ether")
# Node 1:
# 44670.0011340000003
# Node 2 (received mining payout as well):
# 8507.9996220000004
```

Get the balance of account 1

```
> web3.fromWei(eth.getBalance(eth.accounts[1]), "ether")
# Node 1:
# 1.999244
# Node 2:
# 1.999244
```

Output from weakest node (look for "Chain split detected"):
```
TRACE[04-30|11:48:47] Accepted connection                      addr=192.168.2.110:43760
DEBUG[04-30|11:48:47] Adding p2p peer                          name=Geth/v1.8.6-stable-1...                        addr=192.168.2.110:43760 peers=1
TRACE[04-30|11:48:47] connection set up                        id=15950f60b7a6b6b8 addr=192.168.2.110:43760 conn=inbound inbound=true
TRACE[04-30|11:48:47] Starting protocol eth/63                 id=15950f60b7a6b6b8 conn=inbound
DEBUG[04-30|11:48:47] Ethereum peer connected                  id=15950f60b7a6b6b8 conn=inbound name=Geth/v1.8.6-stable-12683fec/linux-amd64/go1.10
TRACE[04-30|11:48:47] Registering sync peer                    peer=15950f60b7a6b6b8
INFO [04-30|11:48:49] Block synchronisation started 
DEBUG[04-30|11:48:49] Synchronising with the network           peer=15950f60b7a6b6b8 eth=63 head=da23c9…986242 td=9648979008 mode=full
DEBUG[04-30|11:48:49] Retrieving remote chain height           peer=15950f60b7a6b6b8
INFO [04-30|11:48:49] Mining aborted due to sync 
TRACE[04-30|11:48:49] Ethash nonce search aborted              miner=0 attempts=306192
DEBUG[04-30|11:48:49] Fetching batch of headers                id=15950f60b7a6b6b8 conn=inbound count=1 fromhash=da23c9…986242 skip=0 reverse=false
TRACE[04-30|11:48:49] Filtering headers                        peer=15950f60b7a6b6b8 headers=1
DEBUG[04-30|11:48:49] Remote head header identified            peer=15950f60b7a6b6b8 number=10630 hash=da23c9…986242
DEBUG[04-30|11:48:49] Looking for common ancestor              peer=15950f60b7a6b6b8 local=10625 remote=10630
DEBUG[04-30|11:48:49] Fetching batch of headers                id=15950f60b7a6b6b8 conn=inbound count=13 fromnum=10433 skip=15 reverse=false
DEBUG[04-30|11:48:49] Found common ancestor                    peer=15950f60b7a6b6b8 number=10609 hash=a0d642…8acb38
DEBUG[04-30|11:48:49] Downloading transaction receipts         origin=10610
DEBUG[04-30|11:48:49] Directing header downloads               peer=15950f60b7a6b6b8 origin=10610
DEBUG[04-30|11:48:49] Downloading block bodies                 origin=10610
TRACE[04-30|11:48:49] Fetching skeleton headers                peer=15950f60b7a6b6b8 count=192 from=10610
DEBUG[04-30|11:48:49] Fetching batch of headers                id=15950f60b7a6b6b8 conn=inbound count=128 fromnum=10801 skip=191 reverse=false
TRACE[04-30|11:48:49] Fetching full headers                    peer=15950f60b7a6b6b8 count=192 from=10610
DEBUG[04-30|11:48:49] Fetching batch of headers                id=15950f60b7a6b6b8 conn=inbound count=192 fromnum=10610 skip=0   reverse=false
TRACE[04-30|11:48:49] Scheduling new headers                   peer=15950f60b7a6b6b8 count=21  from=10610
TRACE[04-30|11:48:49] Fetching full headers                    peer=15950f60b7a6b6b8 count=192 from=10631
DEBUG[04-30|11:48:49] Fetching batch of headers                id=15950f60b7a6b6b8 conn=inbound count=192 fromnum=10631 skip=0   reverse=false
DEBUG[04-30|11:48:49] No more headers available                peer=15950f60b7a6b6b8
DEBUG[04-30|11:48:49] Header download terminated               peer=15950f60b7a6b6b8
TRACE[04-30|11:48:49] Requesting new batch of data             peer=15950f60b7a6b6b8 type=bodies count=1   from=10623
DEBUG[04-30|11:48:49] Fetching batch of block bodies           id=15950f60b7a6b6b8 conn=inbound count=1
DEBUG[04-30|11:48:49] Inserting downloaded chain               items=13 firstnum=10610 firsthash=9e953e…e7d6f6 lastnum=10622 lasthash=8041a7…eb485b
DEBUG[04-30|11:48:49] Data fetching completed                  type=receipts
DEBUG[04-30|11:48:49] Transaction receipt download terminated  err=nil
TRACE[04-30|11:48:49] Filtering bodies                         peer=15950f60b7a6b6b8 txs=1 uncles=1
TRACE[04-30|11:48:49] Peer throughput measurements updated     peer=15950f60b7a6b6b8 hps=0.000 bps=235.149 rps=0.000 sps=0.000 miss=0 rtt=18.000042526s
TRACE[04-30|11:48:49] Delivered new batch of data              peer=15950f60b7a6b6b8 type=bodies   count=1:1
DEBUG[04-30|11:48:49] Data fetching completed                  type=bodies
DEBUG[04-30|11:48:49] Block body download terminated           err=nil
DEBUG[04-30|11:48:49] Trie cache stats after commit            misses=0 unloads=0
DEBUG[04-30|11:48:49] Inserted forked block                    number=10619 hash=2be8a3…722bc5 diff=1137884 elapsed=20.543ms  txs=0 gas=0     uncles=0
DEBUG[04-30|11:48:49] Trie cache stats after commit            misses=0 unloads=0
DEBUG[04-30|11:48:49] Inserted forked block                    number=10620 hash=9bdf74…09de40 diff=1138439 elapsed=615.835µs txs=0 gas=0     uncles=0
DEBUG[04-30|11:48:49] Trie cache stats after commit            misses=0 unloads=0
DEBUG[04-30|11:48:49] Inserted forked block                    number=10621 hash=fbbfc2…3292e2 diff=1138994 elapsed=486.055µs txs=0 gas=0     uncles=0
DEBUG[04-30|11:48:49] Trie cache stats after commit            misses=0 unloads=0
DEBUG[04-30|11:48:49] Inserted forked block                    number=10622 hash=8041a7…eb485b diff=1139550 elapsed=399.383µs txs=0 gas=0     uncles=0
INFO [04-30|11:48:49] Imported new chain segment               blocks=4 txs=0 mgas=0.000 elapsed=23.360ms  mgasps=0.000 number=10622 hash=8041a7…eb485b cache=93.20kB ignored=9
DEBUG[04-30|11:48:49] Inserting downloaded chain               items=8  firstnum=10623 firsthash=9eda98…af879f lastnum=10630 lasthash=da23c9…986242
DEBUG[04-30|11:48:49] Trie cache stats after commit            misses=0 unloads=0
DEBUG[04-30|11:48:49] Inserted forked block                    number=10623 hash=9eda98…af879f diff=1138438 elapsed=9.209ms   txs=1 gas=21000 uncles=0
DEBUG[04-30|11:48:49] Trie cache stats after commit            misses=0 unloads=0
DEBUG[04-30|11:48:49] Inserted forked block                    number=10624 hash=8e14d2…923c3e diff=1138438 elapsed=5.154ms   txs=0 gas=0     uncles=0
DEBUG[04-30|11:48:49] Trie cache stats after commit            misses=0 unloads=0
DEBUG[04-30|11:48:49] Chain split detected                     number=10618 hash=7837f5…585447 drop=7 dropfrom=de8e0b…a58515 add=7 addfrom=59f611…c26c36
DEBUG[04-30|11:48:49] Inserted new block                       number=10625 hash=59f611…c26c36 uncles=0 txs=0 gas=0     elapsed=1.176ms
DEBUG[04-30|11:48:49] Trie cache stats after commit            misses=0 unloads=0
DEBUG[04-30|11:48:49] Dereferenced trie from memory database   nodes=2 size=649.00B time=7.353µs  gcnodes=3748 gcsize=1.06mB   gctime=6.608417ms livenodes=337 livesize=95.60kB
DEBUG[04-30|11:48:49] Inserted new block                       number=10626 hash=dfe4a3…fb2221 uncles=0 txs=0 gas=0     elapsed=874.887µs
DEBUG[04-30|11:48:49] Trie cache stats after commit            misses=0 unloads=0
DEBUG[04-30|11:48:49] Dereferenced trie from memory database   nodes=3 size=764.00B time=6.921µs  gcnodes=3751 gcsize=1.06mB   gctime=6.615217ms livenodes=337 livesize=95.60kB
DEBUG[04-30|11:48:49] Inserted new block                       number=10627 hash=5d4c91…2275c4 uncles=0 txs=0 gas=0     elapsed=509.987µs
DEBUG[04-30|11:48:49] Trie cache stats after commit            misses=0 unloads=0
DEBUG[04-30|11:48:49] Dereferenced trie from memory database   nodes=2 size=649.00B time=4.371µs  gcnodes=3753 gcsize=1.07mB   gctime=6.619473ms livenodes=338 livesize=95.71kB
DEBUG[04-30|11:48:49] Inserted new block                       number=10628 hash=5421b9…245ca5 uncles=0 txs=0 gas=0     elapsed=412.383µs
DEBUG[04-30|11:48:49] Trie cache stats after commit            misses=0 unloads=0
DEBUG[04-30|11:48:49] Dereferenced trie from memory database   nodes=3 size=764.00B time=6.862µs  gcnodes=3756 gcsize=1.07mB   gctime=6.626226ms livenodes=338 livesize=95.71kB
DEBUG[04-30|11:48:49] Inserted new block                       number=10629 hash=8fb57e…da0ed9 uncles=0 txs=0 gas=0     elapsed=459.665µs
DEBUG[04-30|11:48:49] Trie cache stats after commit            misses=0 unloads=0
DEBUG[04-30|11:48:49] Dereferenced trie from memory database   nodes=2 size=649.00B time=5.121µs  gcnodes=3758 gcsize=1.07mB   gctime=6.631236ms livenodes=339 livesize=95.83kB
DEBUG[04-30|11:48:49] Inserted new block                       number=10630 hash=da23c9…986242 uncles=0 txs=0 gas=0     elapsed=497.625µs
INFO [04-30|11:48:49] Imported new chain segment               blocks=8 txs=1 mgas=0.021 elapsed=18.753ms  mgasps=1.120 number=10630 hash=da23c9…986242 cache=95.98kB
DEBUG[04-30|11:48:49] Synchronisation terminated               elapsed=46.017564ms
DEBUG[04-30|11:48:49] Reinjecting stale transactions           count=0
TRACE[04-30|11:48:49] Bad uncle found and will be removed      hash=64aa0a…e155f8
TRACE[04-30|11:48:49] Announced block                          hash=da23c9…986242 recipients=1 duration=2562047h47m16.854s
TRACE[04-30|11:48:49] &{0xc425424480 [] [] {[100 170 10 255 249 163 47 208 56 23 40 180 177 113 136 156 136 147 47 48 228 197 183 168 156 247 246 63 107 225 85 248]} {<nil>} <nil> 0001-01-01 00:00:00 +0000 UTC <nil>} 
TRACE[04-30|11:48:49] Bad uncle found and will be removed      hash=18c112…9a6ee3
TRACE[04-30|11:48:49] &{0xc4229ce6c0 [] [] {[24 193 18 195 45 99 213 163 246 28 37 223 255 66 228 115 213 113 142 249 88 225 152 155 210 27 205 158 219 154 110 227]} {<nil>} <nil> 0001-01-01 00:00:00 +0000 UTC <nil>} 
TRACE[04-30|11:48:49] Bad uncle found and will be removed      hash=e5c6e3…157b7e
TRACE[04-30|11:48:49] &{0xc4253fab40 [] [0xc4280a0120] {[229 198 227 71 95 254 249 83 220 33 200 246 48 242 79 60 107 128 186 21 154 240 69 32 125 217 161 184 49 21 123 126]} {<nil>} <nil> 0001-01-01 00:00:00 +0000 UTC <nil>} 
TRACE[04-30|11:48:49] Bad uncle found and will be removed      hash=50c35e…d12c10
TRACE[04-30|11:48:49] &{0xc4245ab8c0 [] [] {[80 195 94 6 18 255 235 95 156 247 29 164 179 160 93 105 174 2 39 80 183 105 139 20 30 243 168 3 28 209 44 16]} {<nil>} <nil> 0001-01-01 00:00:00 +0000 UTC <nil>} 
TRACE[04-30|11:48:49] Bad uncle found and will be removed      hash=2be8a3…722bc5
TRACE[04-30|11:48:49] &{0xc42338b8c0 [] [] {[43 232 163 135 153 242 200 21 234 232 72 53 44 105 91 10 157 111 155 179 145 145 54 10 228 161 241 6 175 114 43 197]} {<nil>} <nil> 0001-01-01 00:00:00 +0000 UTC <nil>} 
TRACE[04-30|11:48:49] Bad uncle found and will be removed      hash=9bdf74…09de40
TRACE[04-30|11:48:49] &{0xc42338bd40 [] [] {[155 223 116 186 19 3 245 230 93 31 147 173 229 12 120 0 113 211 142 144 106 164 187 144 6 73 26 15 250 9 222 64]} {<nil>} <nil> 0001-01-01 00:00:00 +0000 UTC <nil>} 
TRACE[04-30|11:48:49] Bad uncle found and will be removed      hash=8041a7…eb485b
TRACE[04-30|11:48:49] &{0xc4257e8fc0 [] [] {[128 65 167 234 84 139 173 176 93 132 63 149 178 228 25 57 31 234 28 20 243 106 155 200 6 22 184 84 73 235 72 91]} {<nil>} <nil> 0001-01-01 00:00:00 +0000 UTC <nil>} 
TRACE[04-30|11:48:49] Bad uncle found and will be removed      hash=6c9596…dab567
TRACE[04-30|11:48:49] &{0xc4257d3680 [] [] {[108 149 150 215 153 81 61 5 198 125 218 181 89 185 180 92 199 220 135 97 219 111 119 155 197 241 233 236 78 218 181 103]} {<nil>} <nil> 0001-01-01 00:00:00 +0000 UTC <nil>} 
TRACE[04-30|11:48:49] Bad uncle found and will be removed      hash=9eda98…af879f
TRACE[04-30|11:48:49] &{0xc429a65680 [] [0xc4385ccc60] {[158 218 152 91 30 178 158 26 134 168 174 249 201 18 209 224 200 72 6 64 225 15 93 24 95 108 242 222 133 175 135 159]} {<nil>} <nil> 0001-01-01 00:00:00 +0000 UTC <nil>} 
TRACE[04-30|11:48:49] Bad uncle found and will be removed      hash=8e14d2…923c3e
TRACE[04-30|11:48:49] &{0xc429a65b00 [] [] {[142 20 210 252 9 92 187 86 126 246 43 234 212 87 125 60 223 29 1 203 250 205 17 39 39 105 151 238 140 146 60 62]} {<nil>} <nil> 0001-01-01 00:00:00 +0000 UTC <nil>} 
TRACE[04-30|11:48:49] Bad uncle found and will be removed      hash=fbbfc2…3292e2
TRACE[04-30|11:48:49] &{0xc4257e8b40 [] [] {[251 191 194 135 63 166 61 67 245 141 133 134 107 246 110 3 33 2 199 209 86 33 213 229 23 194 149 24 208 50 146 226]} {<nil>} <nil> 0001-01-01 00:00:00 +0000 UTC <nil>} 
TRACE[04-30|11:48:49] Bad uncle found and will be removed      hash=de8e0b…a58515
TRACE[04-30|11:48:49] &{0xc42b817b00 [] [] {[222 142 11 103 201 106 212 4 130 118 190 92 2 167 225 138 17 220 217 9 9 165 34 236 85 15 84 42 93 165 133 21]} {<nil>} <nil> 0001-01-01 00:00:00 +0000 UTC <nil>} 
TRACE[04-30|11:48:49] Bad uncle found and will be removed      hash=b5bf3b…0a4270
TRACE[04-30|11:48:49] &{0xc429483b00 [] [] {[181 191 59 95 58 196 202 180 106 245 218 255 156 157 247 23 190 134 144 99 67 213 94 42 54 40 165 180 183 10 66 112]} {<nil>} <nil> 0001-01-01 00:00:00 +0000 UTC <nil>} 
INFO [04-30|11:48:49] Starting mining operation 
INFO [04-30|11:48:49] Commit new mining work                   number=10631 txs=0 uncles=0 elapsed=88.988µs
INFO [04-30|11:48:49] ⑂ block  became a side fork              number=10621 hash=18c112…9a6ee3
INFO [04-30|11:48:49] ⑂ block  became a side fork              number=10622 hash=b5bf3b…0a4270
INFO [04-30|11:48:49] ⑂ block  became a side fork              number=10623 hash=64aa0a…e155f8
INFO [04-30|11:48:49] ⑂ block  became a side fork              number=10624 hash=6c9596…dab567
INFO [04-30|11:48:49] ⑂ block  became a side fork              number=10625 hash=de8e0b…a58515
TRACE[04-30|11:48:49] Started ethash search for new nonces     miner=0 seed=2352204855128007855
TRACE[04-30|11:48:53] Ethash nonce found and reported          miner=0 attempts=209915  nonce=2352204855128217770
INFO [04-30|11:48:53] Successfully sealed new block            number=10631 hash=b27d96…af09e9
DEBUG[04-30|11:48:53] Trie cache stats after commit            misses=0 unloads=0
DEBUG[04-30|11:48:53] Dereferenced trie from memory database   nodes=3 size=764.00B time=4.992µs  gcnodes=3761 gcsize=1.07mB   gctime=6.636024ms livenodes=338 livesize=95.71kB
INFO [04-30|11:48:53] 🔨 mined potential block                  number=10631 hash=b27d96…af09e9
DEBUG[04-30|11:48:53] Reinjecting stale transactions           count=0
INFO [04-30|11:48:53] Commit new mining work                   number=10632 txs=0 uncles=0 elapsed=172.687µs
TRACE[04-30|11:48:53] Propagated block                         hash=b27d96…af09e9 recipients=1 duration=2562047h47m16.854s
TRACE[04-30|11:48:53] Announced block                          hash=b27d96…af09e9 recipients=0 duration=2562047h47m16.854s
TRACE[04-30|11:48:53] Started ethash search for new nonces     miner=0 seed=5246497585726578913
DEBUG[04-30|11:49:00] Recalculated downloader QoS values       rtt=19.500010631s confidence=1.000 ttl=58.500031893s
TRACE[04-30|11:49:10] Ethash nonce found and reported          miner=0 attempts=882550  nonce=5246497585727461463
INFO [04-30|11:49:10] Successfully sealed new block            number=10632 hash=264698…130bdf
```
