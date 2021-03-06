---
author: Edson Ayllon
category: research
tags: 
- Stratum
- Ethereum Client
- JSON-RPC
- mining
status: distilled
twitter: https://twitter.com/relativeread
---

## Research 5-2019

# Pantheon External GPU Miner Research Documentation

Research for adding external GPU miner to an Ethereum Client using JSON-RPC or Stratum.

## Contents

<!-- TOC depthFrom:2 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [Contents](#contents)
- [1 | Current Pantheon Releases](#1-current-pantheon-releases)
- [2 | Mining Software](#2-mining-software)
	- [2.1 Qtminer](#21-qtminer)
	- [2.2 Minergate](#22-minergate)
	- [2.3 Claymore](#23-claymore)
	- [2.4 Phoenix Miner](#24-phoenix-miner)
	- [2.5 Ethminer](#25-ethminer)
- [3 | API Protocols](#3-api-protocols)
	- [3.1 Getwork Protocol](#31-getwork-protocol)
	- [3.2 Stratum Protocol](#32-stratum-protocol)
- [4 | Stratum Implementation](#4-stratum-implementation)
	- [4.0 GetWork](#40-getwork)
	- [4.1 Stratum 1.0 (Dwarfpool Stratum Protocol)](#41-stratum-10-dwarfpool-stratum-protocol)
		- [4.1.1 Rationale](#411-rationale)
		- [4.1.2 Initial Connection](#412-initial-connection)
		- [4.1.5 Jobs](#415-jobs)
	- [4.2 Stratum 2.0 (NiceHash Stratum Protocol)](#42-stratum-20-nicehash-stratum-protocol)
		- [4.2.1 Rationale](#421-rationale)
		- [4.2.2 Initial Connection](#422-initial-connection)
		- [4.2.3 Difficulty](#423-difficulty)
		- [4.2.4 ExtraNonce](#424-extranonce)
		- [4.2.5 Jobs](#425-jobs)
- [References](#references)

<!-- /TOC -->

## 1 | Current Pantheon Releases

Pantheon, as of now, is a Command Line Interface (CLI) solution, and such, design a user interface is not within the scope of the project. However, as a CLI application, flags must be included to perform various operations.


Pantheon's current mining solution uses CPU mining. From [Pantheon's documentation](https://docs.pantheon.pegasys.tech/en/latest/Getting-Started/Starting-Pantheon/), the following instructions are supplied which include mining:

> To run a node that mines blocks at a rate suitable for testing purposes:

```
pantheon --network=dev --miner-enabled --miner-coinbase=0xfe3b557e8fb62b89f4916b721be55ceb828dbd73 --rpc-http-cors-origins="all" --host-whitelist="*" --rpc-ws-enabled --rpc-http-enabled --data-path=/tmp/tmpDatdir
```

>Alternatively, use the following configuration file on the command line to start a node with the same options as above:
>
```
network="dev"
miner-enabled=true`
miner-coinbase="0xfe3b557e8fb62b89f4916b721be55ceb828dbd73"
rpc-http-cors-origins=["all"]
host-whitelist=["*"]
rpc-ws-enabled=true
rpc-http-enabled=true
data-path="/tmp/tmpdata-path"
```


Currently, Pantheon [offers binaries](https://docs.pantheon.pegasys.tech/en/latest/Installation/Install-Binaries/) for Windows, MacOS, and Linux. As such, it would be ideal for a GPU miner integration to include compatibility across these three operating systems.

Several MacOS hardware offerings may be insufficient for hardware mining, as many models come with integrated graphics.

The Mac current offerings in Apple.com show [the following graphics options](https://www.apple.com/mac/compare/):

- Mac Mini: Intel UHD Graphics 630
- Macbook Air (Retina): Intel UHD Graphics 617
- Macbook Pro (13 in): Intel Iris Plus Graphics 645 or 655
- Macbook Pro (15 in):
  - Intel UHD Graphics 630 (all configurations)
  - Radeon Pro 555X with 4GB of GDDR5 memory
  - Radeon Pro 560X with 4GB of GDDR5 memory
  - Radeon Pro Vega 16 with 4GB of HBM2 memory
  - Radeon Pro Vega 20 with 4GB of HBM2 memory
- iMac (21.5 in): Intel Iris Plus Graphics 640
- iMac (21.5 in Retina):
  - Radeon Pro 555X with 2GB of GDDR5 memory
  - Radeon Pro 560X with 4GB of GDDR5 memory
  - Radeon Pro Vega 20 with 4GB of HBM2 memory
- iMac (25 in):
  - Radeon Pro 570X with 4GB of GDDR5 memory
  - Radeon Pro 575X with 4GB of GDDR5 memory
  - Radeon Pro 580X with 8GB of GDDR5 memory
  - Radeon Pro Vega 48 with 8GB of HBM2 memory
- iMac Pro:
  - Radeon Pro Vega 56 with 8GB of HBM2 memory
  - Radeon Pro Vega 64 with 16GB of HBM2 memory
  - Radeon Pro Vega 64X with 16GB of HBM2 memory
- Mac Pro:
  - Dual AMD FirePro D500 with 3GB of GDDR5 memory each
  - Dual AMD FirePro D700 with 6GB of GDDR5 memory each
- Mac Pro (New):
  - One Radeon Pro 580X MPX Module with 8GB of GDDR5 memory
  - One or two Radeon Pro Vega II MPX Modules with 32GB of HBM2 memory each
  - One or two Radeon Pro Vega II Duo MPX Modules with 64GB of HBM2 memory each

_Note that the Radeon Pro 580X MPX [is a less powerful model](https://www.techpowerup.com/gpu-specs/radeon-pro-580x.c3398) than the Radeon Rx 580._

Due to increases in DAG file, it's no longer possible to GPU mine with 2GB memory graphics cards. Any Mac with 2GB of GPU memory will be unable to mine.

Mining on Intel integrated graphics cards (iGPUs) [have had reports of](https://bitcointalk.org/index.php?topic=1820187.0) very low hashing power along with possibly permamently damaging the iGPU.

Mining on Macbook Pro GPUs have [had reported hashing rates of 8 Mh/s](https://forum.ethereum.org/discussion/16483/mining-on-the-macbook-pros-amd-radeon-560-gpu). However, mining on Macbooks is not recommended due to how Macbooks handle heat, and the amount of heat generated by mining.

Certain models of the iMac and Mac Pro are sufficent to mine with, namely models with Radeon Pro Vega GPUs. However, considering how expensive these higher Mac offerings can be, how these Macs handle heat should be of concern as it may be possible mining may cause permanent damage to the computer. However, Mac compatibility should be considered for these higher models.It is also possible to mine with an external graphics card(s) (eGPU) on Mac, further creating a case for compatibility.

## 2 | Mining Software


Ethereum mining software includes the following:

- [ETHminer](https://github.com/ethereum-mining/ethminer)
- [Claymore](https://github.com/Claymore-Dual/Claymore-Dual-Miner)
- [Qtminer](https://github.com/etherchain-org/qtminer)
- [Mingergate's Miner](https://minergate.com/downloads)


### 2.1 Qtminer

Qtminer's repo [describes the project as](https://github.com/etherchain-org/qtminer) a Stratum enabled Ethereum miner, however lacks sufficient documentation.

A [good number of users](https://www.reddit.com/r/EtherMining/search/?q=qtminer&restrict_sr=1)
have used Qtminer in the past, however these recorded cases exceed a year ago, while [Claymore](https://www.reddit.com/r/EtherMining/search/?q=claymore&restrict_sr=1) and [Ethminer engagements](https://www.reddit.com/r/EtherMining/search/?q=ethminer&restrict_sr=1) have been more recent.

[Qtminer's Github repo](https://github.com/etherchain-org/qtminer/commits/master) indicates only 4 commits to the project, all in the year 2015, indicating no ongoing or recent development or developer interest in the project.

This miner should not be pursued for this project.

### 2.2 Minergate

Minergate [has support for](https://minergate.com/downloads) both Windows, Linux, and MacOS. Mingergate miner is a miner [built for the MinerGate mining pool](https://minergate.com/). Minergate's website promises hashing speeds slightly more optimized than those from Ethminer and Claymore, displaying hashing power of [14.39 Mh/s, vs 14.13 and 14.31 Mh/s](https://minergate.com/downloads) for Claymore and Ethminer respectively for an AMD Rx580 GPU.

However, Minergate is a GUI program, which may excessive as Pantheon is a Command Line Interface application. Minergate will also not be considered.


### 2.3 Claymore

Of the mining programs most actively used by miners currently, both Claymore and Ethminer seem to be [the most broadly used](https://www.reddit.com/r/EtherMining).

Claymore [supports both](https://github.com/Claymore-Dual/Claymore-Dual-Miner) NVIDIA and AMD GPUs, optimized for OpenCL and CUDA cores.

Claymore charges a [1% developer fee](https://github.com/Claymore-Dual/Claymore-Dual-Miner) for ETH mining, which [reportedly increases](https://www.reddit.com/r/EtherMining/wiki/software/apps) with dual mining other currencies.

Claymore does not seem to be have any software license.

### 2.4 Phoenix Miner

[Phoenix Miner](https://github.com/Phoenix-Miner/PhoenixMiner), like Claymore, is a miner optimized for OpenCL and CUDA cores. Phoenix Miner also contains a 1% developer fee. This fee is collected by mining with the developer's address [for 35 seconds every 90 minutes](https://github.com/Phoenix-Miner/PhoenixMiner), similar to Claymore. During its initial release, Phoenix seemed to do[ slightly better than Claymore in hashrate optimization](https://www.reddit.com/r/EtherMining/comments/7t2sd6/anyone_try_phoenix_miner_apparently_slightly/).


Phoenix Miner does not seem to be have any software license.

### 2.5 Ethminer

[Ethminer](https://github.com/ethereum-mining/ethminer) originates from `cpp-ethereum` which became [Aleth](https://github.com/ethereum/aleth), a C++ Ethereum client. Ethminer started after `cpp-ethereum` discontinued its GPU mining functionality.

Ethminer [supports both](https://github.com/ethereum-mining/ethminer) NVIDIA and AMD GPUs (OpenCL and CUDA cores).

The [Minergate pool reports](https://minergate.com/downloads) slightly faster and slightly slower hashrates of Ethminer compared to Claymore depending on GPU, however, both stay relatively close in hashrates.

Ethminer has had over 14,000 commits, with the last commit [occuring late June, 2019](https://github.com/ethereum-mining/ethminer/commits/master).

Unlike Claymore and Phoenix Miner, there exists [an Ethminer version compatible with Mac](https://github.com/ArtSabintsev/Ethminer-for-macOS), The Mac Ethminer repo contains over 12,000 commits, however, the last commit was recieved in 2018.

Ethminer also [does not seem to include a dev fee](https://github.com/ethereum-mining/ethminer).

Ethminer has compatibility for Windows, Linux and [MacOS](https://github.com/ArtSabintsev/Ethminer-for-macOS).

Ethminer comes in [standalone executables](https://github.com/ethereum-mining/ethminer/releases/tag/v0.18.0) for Linux and Windows. Ethminer can also be [built from source](https://github.com/ethereum-mining/ethminer/blob/master/docs/BUILD.md) for Windows, Linux, and MacOS. Binaries are not included in Pantheon source, so unique builds per operating system can be made through the Pantheon CLI. Because of MacOS compatibility, no dev fee as default, and active development by the Ethminer devs, Ethminer should be the miner considered for Pantheon.

Ethminer is GPL 3.0, which is compatible with Pantheon's Apache 2.0 license.

## 3 | API Protocols

From the [Ethminer documentation](https://github.com/ethereum-mining/ethminer/blob/master/docs/POOL_EXAMPLES_ETH.md), Ethminer connects to mining pools using the following syntax:

```
-P scheme://user[.workername][:password]@hostname:port[/...]
```

Where values in square brackets are optional. The value `scheme` can be any of the following:

- `http` for getwork mode (geth)
- `stratum+tcp` for plain stratum mode
- `stratum1+tcp` for plain stratum eth-proxy compatible mode
- `stratum2+tcp` for plain stratum NiceHash compatible mode

For a Pantheon node on localhost, the hostname and port would be `localhost` and, what appears to be from building from the Pantheon repository, port 30303.

### 3.1 Getwork Protocol

An overview of getwork can be [found on the Bitcoin wiki](https://en.bitcoinwiki.org/wiki/Getwork). Getwork uses JSON-RPC over an HTTP transport. Without arguments, getworkprovides a block header for a miner to find a solution.

The following is a getwork implementation example for Ethereum:

- https://github.com/sammy007/ether-proxy

The Ethereum version of HTTP Getwork pools originally used was initiating a `go-ethereum` (geth) node with RPC enabled. Documentation for enabling gpu mining with geth is [found on the Ethereum Gitbook](https://ethereum.gitbooks.io/frontier-guide/content/gpu.html).






### 3.2 Stratum Protocol

The following sources provide a specification for Stratum


- [EIP 1571](http://eips.ethereum.org/EIPS/eip-1571)
- [Slushpool](https://slushpool.com/help/stratum-protocol/) and [Slushpool Github](https://github.com/slushpool/stratumprotocol/blob/master/stratum-extensions.mediawiki)
- [Nicehash](https://github.com/nicehash/Specifications/blob/master/EthereumStratum_NiceHash_v1.0.0.txt)
- [Sia Mining](https://siamining.com/stratum)
- [Aion Network](https://github.com/aionnetwork/aion_miner/wiki/Aion-Stratum-Protocol)
- [php-proxy-stratum](https://github.com/ctubio/php-proxy-stratum/wiki/Stratum-Mining-Protocol)
- [open-ethereum-pool](https://github.com/sammy007/open-ethereum-pool/blob/master/docs/STRATUM.md)


[Originally created for](https://slushpool.com/help/stratum-protocol/) the Bitcoin client [Electrum](https://electrum.org/#home), the Stratum Protocol was ported to mining to [overcome getwork shortcomings](https://slushpool.com/help/stratum-protocol/) when considering large scale mining systems.

Getwork specification for the SHA-256 mining algorithm may be [suitable for a 4.2 GH/s mining rig](https://slushpool.com/help/stratum-protocol/). This means, for a 42 GH/s mining rig, 10 concurrent requests are required. For larger scale [ASIC](https://en.bitcoin.it/wiki/ASIC) mining equipment, this leads to not enough jobs to meet the capacity of the equipment via the getwork protocol.

With the introduction of mining pools, several mining rigs combining hasing capacity for shared mining rewards, [under getwork, miners had to choose](https://slushpool.com/help/stratum-protocol/) short intervals which led to higher network load and lower staling ratio, or intervals which do not overload the network and servers.[Long polling](https://en.bitcoin.it/wiki/Long_polling), the solution to this problem, [created another problem]() where long polling reconnections were difficult to distinguish from DDoS attacks.

While http getwork may be sufficient for lesser powered mining rigs, since Pantheon is an enterprise level solution, Stratum should be the considered implementation.


The Stratum Protocol [reduces client-server communications](https://github.com/ctubio/php-proxy-stratum/wiki/Stratum-Mining-Protocol) to work under very low bandwidth, reducing server strain. [The server pushes work](https://slushpool.com/help/stratum-protocol/) to the mining client, whereby the miner can use that work until the next push, regardless of hashing rate.

[Stratum uses](https://slushpool.com/help/stratum-protocol/) a plain TCP socket, with payload encoded as JSON-RPC messages.  [The mining client opens](https://slushpool.com/help/stratum-protocol/) a TCP socket and writes requests to the server in JSON messages finished by the newline character `\n`. [Each line received](https://slushpool.com/help/stratum-protocol/) by the client is a JSON-RPC fragment containing the response.


The following are examples of Stratum implementations for Ethereum:
- https://github.com/Atrides/eth-proxy
- https://github.com/sammy007/open-ethereum-pool
- https://github.com/coinfoundry/Miningcore
- https://bitcointalk.org/index.php?topic=1200891.0


The Ethereum implementation of stratum started with creating a TCP proxy for a traditional geth HTTP RPC node. The first mining pool to implement stratum for Ethereum was Dwarfpool, and as such, all pools followed Dwarfpool's stratum specification (spec) at first, while most major mining pools still follow this specification currently. Ethminer calls this spec Stratum 1.

Ethereum's Stratum 1 protocol deviates from Slushpool's stratum specification made for Bitcoin. RPC methods in Stratum 1 differ from the original Slushpool spec, where `mining.submit` from the original stratum spec is labeled `eth_getWork` in Dwarfpool's stratum spec. `eth_getWork` is a method from [Ethereum's JSON-RPC api](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_getwork). The Stratum specification from Slushpool also is not real JSON RPC, but is based on JSON RPC 2.0, omitting JSON RPC version, and appending character `/n` with each message. Stratum 1, or the Dwarfpool spec of stratum seems to be a TCP proxy for the getWork HTTP RPC of geth.

While Dwarfpool's stratum specification deviates from Slushpool's original stratum specification, and is actually a proxy for HTTP getWork, Dwarfpool reports increased mining rewards of 10-20% compared to HTTP getWork.

In response to Dwarfpool's deviation from the original Slushpool stratum specification for Bitcoin, Nicehash developed a new stratum protocol for Ethereum which follows Slushpool Stratum closely, while also providing more detailed documentation on the protocol. Ethminer calls this stratum spec Stratum 2, while Nicehash calls this stratum specification `EthereumStratum/1.0.0`.

A stratum specification building on Nicehash's stratum spec, which the creator named `EthereumStratum/2.0.0`, is documented under [EIP 1517](https://eips.ethereum.org/EIPS/eip-1571). The EthereumStratum/2.0.0 EIP draft was created November 2018.

## 4 | Stratum Implementation

Stratum is a communication standard for pools and miners based on the  JSON encoded remote procedure call version 2.0 ([JSON-RPC 2.0](https://www.jsonrpc.org/specification)). JSON-RPC is an API protocol predating Graphql and REST.

A client sends a request to the server, calling a function/method. If the request contains an ID, the server provides a response to the client.

Stratum uses the following JSON-RPC native message types:
- request
- response
- notification


A JSON-RPC request contains the following components:

- `method`: The name of the method called
- `params`: An Object or Array passed as parameters to the method
- `id`: A value of any type used to match a response to a request

The response to requests contain the following components:

- `result`: The data returned by the method.
- `error`: A specified error code if there was an error.
- `id`: The id of the request it is responding to

In JSON-RPC 1.0, both `result` and `error` would be supplied with each response, with a value of `null` for the field not used. However, in JSON-RPC 2.0, either `result` or `error` is sent, with `null` omitted.

A notification, is a request requiring no response. As so, notifications are formated like requests, but with `id` omitted in JSON-RPC 2.0.

- `method`
- `params`

The JSON-RPC behavior of Stratum is done from server to miner via socket transport. In a standard Transmission Control Protocol (TCP) socket, clients initiate connection, then both sides can initiate transport of payload (application protocol) anytime.


Bitcoin Stratum utilizes [the following methods](https://en.bitcoin.it/wiki/Stratum_mining_protocol):
- `mining.subscribe`: Miner subscribes to work from a server, required before all other communication.
- `mining.notify`: Server pushes new work to the miner.
- `mining.set_difficulty`: Signals the miner to stop submitting shares below the new difficulty.
- `mining.submit`: Miner submits shares

### 4.0 GetWork

GetWork comes from the [Ethereum JSON RPC `eth_getWork` method](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_getwork). For this, an Ethereum HTTP RPC endpoint is needed with `eth_getWork` producing work, and `eth_submitWork` accepting solutions. Most Ethereum mining pools traditionally use `go-ethereum` (geth) for this.

For EthMiner, the following conditions must be met.
- JSON responses must be on one Line
- Work must always be available from `eth_getWork`

If a JSON response spans more than one line (has line breaks), EthMiner will throw an error and not read any work.

If work is available one moment, but the next displays no work, EthMiner will abort a connection, as it queries `eth_getWork` multiple times to fulfill hashing rate. Once work is available, that work should be provided by `eth_getWork` until new work becomes available.  

### 4.1 Stratum 1 (Dwarfpool Stratum Protocol)

#### 4.1.1 Rationale

The Dwarfpool Stratum spec is simply a TCP proxy for HTTP getwork, where new work is pushed to miners via JSON RPC notifications. That simple change produces 10-20% greater rewards per miner than getwork, depending on a rig's hashing capacity. For this version to run, getwork must be operational first. For Ethereum, getwork uses normal API protocols for Ethereum JSON RPC. Traditional mining pools use `go-ethereum` (geth) for getwork, so, to add getwork to Pantheon, see how geth implements HTTP JSON RPC. Given getwork is operation, this stratum spec is the easiest to implement for Ethereum.

Stratum version 1 for Ethereum is broadly supported by miners. The majority of mining pools use this protocol. The motivation for implementing this protocol over Stratum 2, however, is the ease of implementation. Stratum 1 may be simpler to integrate a working version, as it contains less restrictions. This may be sufficient for an initial implementation, and an upgrade can be done further on. However, when Ethereum transitions to Proof of Stake (PoS), if Pantheon also transitions to PoS for enterprise, such an upgrade may become unnecessary.

Documentaion for Stratum 2.0 was left in the case Stratum 2.0 for Ethereum is desired to be pursued further by the Pantheon team.

Specification for Stratum 1 appear on [open-ethereum-pool](https://github.com/sammy007/open-ethereum-pool/blob/master/docs/STRATUM.md)'s Stratum documentation. That specification mimics the functionility found on [`eth-proxy`](https://github.com/Atrides/eth-proxy). That documentation will be used as a reference.

The Dwarfpool version of stratum [uses the JSON-RPC 2.0 specification without altercations for their stratum](https://github.com/Atrides/eth-proxy/blob/master/stratum/protocol.py).

Example from `eth-proxy/stratum/protocol.py`:

```
({'id': request_id, 'method': method, 'params': params, 'jsonrpc':'2.0', 'worker': worker}
```

The Dwarfpool stratum protocol deviates from the original Slushpool stratum protocol in method naming convention.

Comparison of [Dwarfpool's stratum vs Slushpool's original spec](https://slushpool.com/help/stratum-protocol/#!/manual/stratum-protocol):

```
*The eth-proxy method*
Stratum 0: Connecting to us2.ethpool.org:3333
Stratum 0: Connection Established Successfully
Stratum 0: Sending Message - {"id": 1, "jsonrpc": "2.0", "method": "eth_submitLogin", "params": ["ADDRESS", "EMAIL"]}
Stratum 1: Connecting to eth.f2pool.com:8008
Stratum 0: Message Received - {"id":1,"jsonrpc":"2.0","result":true}
Stratum 1: Connection Established Successfully
Stratum 1: Sending Message - {"id": 1, "jsonrpc": "2.0", "method": "eth_submitLogin", "params": ["ADDRESS", "EMAIL"]}
Stratum 1: Message Received - {"jsonrpc":"2.0","id":1,"result":true,"error":null}

---
*Real stratum method*
Stratum 0: Connecting to us2.ethpool.org:3333
Stratum 0: Connection Established Successfully
Stratum 0: Sending Message - {"id": 1, "jsonrpc": "2.0", "method": "mining.subscribe", "params": []}
Stratum 1: Connecting to eth.f2pool.com:8008
Stratum 0: Message Received - {"id":1,"jsonrpc":"2.0","result":true}
Stratum 1: Connection Established Successfully
Stratum 1: Sending Message - {"id": 1, "jsonrpc": "2.0", "method": "mining.subscribe", "params": []}
Stratum 1: No Message Received After Timeout!
Stratum 1: Receive Error - An existing connection was forcibly closed by the remote hostname
```


#### 4.1.2 Initial Connection

Open Ethereum Pool uses JSON-RPC 2.0 for their server.


A request from the miner to the server has the following syntax:

```
{
  "id": 1,
  "jsonrpc": "2.0",
  "method": "eth_submitLogin",
  "params": ["0xb85150eb365e7df0941f0cf08235f987ba91506a"]
}
```

A successful response returns the following:


```
{ "id": 1, "jsonrpc": "2.0", "result": true }
```

Errors are handled as follows:

```
{
  "id": 1,
  "jsonrpc": "2.0",
  "result": null,
  "error": {
    code: -1,
    message: "Invalid login"
  }
}
````



#### 4.1.5 Jobs

Job requests have the following syntax:

```
{ "id": 1, "jsonrpc": "2.0", "method": "eth_getWork" }

```

A successful response returns the following:

```
{
  "id": 1,
  "jsonrpc": "2.0",
  "result": [
      "0x1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef",
      "0x5eed00000000000000000000000000005eed0000000000000000000000000000",
      "0xd1ff1c01710000000000000000000000d1ff1c01710000000000000000000000"
    ]
}
```

Following [Ethereum's JSON-RPC documentation](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_getwork), the values in the results array are as follows:

- `DATA, 32 Bytes` - current block header pow-hash
- `DATA, 32 Bytes` - the seed hash used for the DAG.
- `DATA, 32 Bytes` - the boundary condition ("target"), 2^256 / difficulty.


If no work is available at the time of request, server return the following error:

```
{
  "id": 10,
  "result": null,
  "error":
  {
    code: 0,
    message: "Work not ready"
  }
}
```

The server pushes new jobs to the miner as they become available through a JSON-RPC notification.

```
Server sends job to peers if new job is available:

{
  "jsonrpc": "2.0",
  "result": [
      "0x1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef",
      "0x5eed00000000000000000000000000005eed0000000000000000000000000000",
      "0xd1ff1c01710000000000000000000000d1ff1c01710000000000000000000000"
    ]
}
```

The results array from job notifications are the same response as with job requests, as seen above.

A share submission request looks like the following:

```
{
  "id": 1,
  "jsonrpc": "2.0",
  "method": "eth_submitWork",
  "params": [
    "0xe05d1fd4002d962f",
    "0x6c872e2304cd1e64b553a65387d7383470f22331aff288cbce5748dc430f016a",
    "0x2b20a6c641ed155b893ee750ef90ec3be5d24736d16838b84759385b6724220d"
  ]
}
```

Parameters include:
- `DATA, 8 Bytes` - The nonce found (64 bits)
- `DATA, 32 Bytes` - The header's pow-hash (256 bits)
- `DATA, 32 Bytes` - The mix digest (256 bits)

A response from the server contains the following:

```
{ "id": 1, "jsonrpc": "2.0", "result": true }
{ "id": 1, "jsonrpc": "2.0", "result": false }
```

On Open Ethereum Pool's Stratum server, invalid share submissions return a message, followed by a temporary ban.

```
{ "id": 1, "jsonrpc": "2.0", "result": null, "error": { code: 23, message: "Invalid share" } }
{ "id": 1, "jsonrpc": "2.0", "result": null, "error": { code: 22, message: "Duplicate share" } }
{ "id": 1, "jsonrpc": "2.0", "result": null, "error": { code: -1, message: "High rate of invalid shares" } }
{ "id": 1, "jsonrpc": "2.0", "result": null, "error": { code: 25, message: "Not subscribed" } }
{ "id": 1, "jsonrpc": "2.0", "result": null, "error": { code: -1, message: "Malformed PoW result" } }
```


### 4.2 Stratum 2 (NiceHash Stratum Protocol)


#### 4.2.1 Rationale

Ethminer offers the following options for pool connection:
- `stratum1+tcp`
- `stratum2+tcp`

`stratum1` refers to the Dwarfpool implementation of stratum, the first stratum implementation for Ethereum. The Dwarfpool stratum implementation can be found here: https://github.com/Atrides/eth-proxy.

`stratum2` refers to the Nicehash specification of stratum, "EthereumStratum/1.0.0." The Nicehash stratum specification was created to offer improvements to the Dwarfpool implementation.

An update the Nicehash stratum specification is given in [EIP 1571](http://eips.ethereum.org/EIPS/eip-1571), titled "EthereumStratum/2.0.0."

Due to the detail of the Nicehash specification, along with improvements offered by this stratum protocol, the Nicehash stratum protocol will be explored.

The Nicehash stratum protocol is supported by Ethminer and Claymore. Configuration options to enable mining for Claymore under the Nicehash protocol is found in the [Claymore Readme txt file](https://github.com/Claymore-Dual/Claymore-Dual-Miner/blob/master/files/Readme!!!.txt).

The [Nicehash protocol](https://github.com/nicehash/Specifications/blob/master/EthereumStratum_NiceHash_v1.0.0.txt) offers the following benefits over the Dwarfpool stratum implementation:

1. Unique data per miner/worker
2. Reduction of difficulty redundancy
3. Reduction of data redundancy
4. Increased consistency with the original [Slushpool stratum specification](https://slushpool.com/help/stratum-protocol/)
5. Documentation of specification, as opposed to pure implementation

Naming convention of the Nicehash stratum spec reflects the original Slushpool spec which was made for Bitcoin. The Dwarfpool stratum spec naming convention extends getwork, which uses the Ethereum JSON RPC API.

An example implementation of the Nicehash stratum protocol can be found at [Miningcore's repo](https://github.com/coinfoundry/Miningcore).

[Nicehash stratum specification](https://github.com/nicehash/Specifications/blob/master/EthereumStratum_NiceHash_v1.0.0.txt) is as following. All standard Stratum protocol is employed, except for the following:

#### 4.2.2 Initial Connection

Handshake happens after TCP connection is established.

Miner sends data first:

```
{
  "id": 1,
  "method": "mining.subscribe",
  "params": [
    "MinerName/1.0.0", "EthereumStratum/1.0.0"
  ]
}\n
```
- `mining.subscribe`: Miner subscribes to work from a server, required before all other communication.
- `Parameter 1`: Miner name and version
- `Parameter 2`: Stratum protocol and version ("EthereumStratum/Version" for the Nicehash spec)

Each message from miner to pool must have a unique id for the miner to properly read responses as pool may not process miner's messages in first-in-first-out manner.

Server replies:

```
{
  "id": 1,
  "result": [
    [
      "mining.notify",
      "ae6812eb4cd7735a302a8a9dd95cf71f",
      "EthereumStratum/1.0.0"
    ],
    "080c"
  ],
  "error": null
}\n
```
- `mining.notify`: Pushes new work to the miner.
- `Parameter 2` of `result`: Extranonce (in Hex) set by pool. Extranonce may be max 3 bytes in size.
- `Parameter 3` of `result`: "EthereumStratum/1.0.0". If the pool does not report this parameter, or a different version than is supported is reported, the miner should terminate connection.


Miner shall authorize during initial handshake.

#### 4.2.3 Difficulty

Before first job (work) is provided, pool must set difficulty:

```
{
  "id": null,
  "method": "mining.set_difficulty",
  "params": [
    0.5
  ]
}\n
```
- `mining.set_difficulty`: Signals the miner to stop submitting shares below the new difficulty.
- `Parameter 1`: Difficulty in `double` data-type. Conversion
between difficulty and target is done as with Bitcoin.
Difficulty of 1 is transformed to target in HEX:
`00000000ffff0000000000000000000000000000000000000000000000000000`.

If the pool does not set difficulty before first job, then miner assumes difficulty 1 was set.

When difficulty changes, the miner uses the new difficulty for all subsequent jobs recieved.

#### 4.2.4 ExtraNonce

If miner has subscribed to extranonce notifications, then pool may change
miner's extranonce by sending:

```
{
  "id": null,
  "method": "mining.set_extranonce",
  "params": [
    "af4c"
  ]
}\n
```

- `mining.set_extranonce`: Miner replaces the initial subscription values starting with the next recieved job

New extranonce is valid for all subsequent jobs recieved.

#### 4.2.5 Jobs

Pool informs miners about job (work) by sending:

```
{
  "id": null,
  "method": "mining.notify",
  "params": [
    "bf0488aa",
    "abad8f99f3918bf903c6a909d9bbc0fdfa5a2f4b9cb1196175ec825c6610126c",
    "645cf20198c2f3861e947d4f67e3ab63b7b2e24dcc9095bd9123e7b33371f6cc",
    true
  ]
}\n
```

- `mining.notify`: Pushes new work to the miner.
- `Parameter 1`: job ID (must be HEX number of any
size)
- `Parameter 2`: Seedhash. Sent with every job to support possible multipools which rapidly change currencies.
- `Parameter 3`: Headershash.
- `Parameter 4`: Boolean `cleanjobs`.  If set `true`, the miner must clear queue of jobs and immediatelly work on new provided job, as old jobs will result in stale share error.

Miner uses seedhash to identify DAG, then tries to find share below
target (which is created out of provided difficulty) with headerhash,
extranonce and own minernonce.

Share submission (completed jobs) by miner:

```
{
  "id": 244,
  "method": "mining.submit",
  "params": [
    "username",
    "bf0488aa",
    "6a909d9bbc0f"
  ]
}\n
```

- `mining.submit`: Submits shares
- `Parameter 1`: username
- `Parameter 2`: job ID
- `Parameter 3`: minernonce

In the above example minernonce is 6 bytes, because
provided extranonce was 2 bytes. If the pool provides 3 bytes extranonce,
then minernonce must be 5 bytes.

For every work submit, the pool must respond with standard stratum
response:

```
{
  "id": 244,
  "result": true,
  "error": null
}\n
```

For shares accepted.


```
{
  "id": 244,
  "result": false,
  "error": [
    -1,
    "Job not found",
    NULL
  ]
}\n

```

For shares not accepted.

The mining protocol by Dwarfpool differs from the original stratum spec as it seems method names are borrowed from [Ethereum's RPC API naming convention](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_getwork).



## References

- [Pantheon Documentation](https://docs.pantheon.pegasys.tech/en/latest/)
- [EtherMining Reddit](https://www.reddit.com/r/EtherMining) for community engagement in Ethereum mining.
