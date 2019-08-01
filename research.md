# Pantheon GPU Miner Research Documentation

## Current Pantheon Releases

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

## Mining Software


Ethereum mining software includes the following:

- [ETHminer](https://github.com/ethereum-mining/ethminer)
- [Claymore](https://github.com/Claymore-Dual/Claymore-Dual-Miner)
- [Qtminer](https://github.com/etherchain-org/qtminer)
- [Mingergate's Miner](https://minergate.com/downloads)

### Claymore

Claymore [supports both](https://github.com/Claymore-Dual/Claymore-Dual-Miner) NVIDIA and AMD GPUs (OpenCL and CUDA).

### ETHminer

Ethminer [supports both](https://github.com/ethereum-mining/ethminer) NVIDIA and AMD GPUs (OpenCL and CUDA).


### Qtminer

Qtminer's repo [describes the project as](https://github.com/etherchain-org/qtminer) a Stratum enabled Ethereum miner, however lacks sufficient documentation.

A [good number of users](https://www.reddit.com/r/EtherMining/search/?q=qtminer&restrict_sr=1)
have used Qtminer in the past, however these recorded cases exceed a year ago, while [Claymore](https://www.reddit.com/r/EtherMining/search/?q=claymore&restrict_sr=1) and [Ethminer engagements](https://www.reddit.com/r/EtherMining/search/?q=ethminer&restrict_sr=1) have been more recent.

[Qtminer's Github repo](https://github.com/etherchain-org/qtminer/commits/master) indicates only 4 commits to the project, all in the year 2015, indicating no ongoing or recent development or developer interest in the project.

This miner should not be pursued for this project.

### Minergate

Minergate [has support for](https://minergate.com/downloads) both Windows, Linux, and MacOS.

## API protocols

### Stratum



## References

- [Pantheon Documentation](https://docs.pantheon.pegasys.tech/en/latest/)
- [EtherMining Reddit](https://www.reddit.com/r/EtherMining) for community engagement in Ethereum mining.