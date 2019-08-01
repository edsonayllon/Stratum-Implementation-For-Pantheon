# Pantheon GPU Miner Research

## Description

 Upgrade Pantheon's mining capabilities to include GPU mining.

## Research Criteria

The research should:
1. Identify mining software to be supported
2. Identify Protocols to be supported
3. Implementation instructions of those protocols


## Research Procedure


## Research Documentation

### Current Pantheon Releases

Currently, Pantheon [offers binaries](https://docs.pantheon.pegasys.tech/en/latest/Installation/Install-Binaries/) for Windows, MacOS, and Linux. As such, it would be ideal for a GPU miner integration to include compatibility across these three operating systems.

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

### Mining Software


Ethereum mining software includes the following:

- [ETHminer](https://github.com/ethereum-mining/ethminer)
- [Claymore](https://github.com/Claymore-Dual/Claymore-Dual-Miner)
- [CGMiner]()
- [WinETH]()
- [Minergate]()
