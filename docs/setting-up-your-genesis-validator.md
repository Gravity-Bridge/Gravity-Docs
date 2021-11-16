# Gravity Genesis Bridge Validator Setup

## Index

// fill out an index with page references

## System setup

While Gravity Bridge full nodes can be run on any platform it is recomended you run your validator on Linux. The most up to date version of any particular distribution is recomended.

### Recomended specs

Recomended specs are 8gb ram and a quad core CPU. Storage specs will change over time as the chain grows, start with at least 125gb of SSD space. Remeber if you are not using a Copy on write filesystem you will need enough storage to keep a backup of the entire blockchain during upgrades.

### Optional: Copy on Write + Compressing filesystem

This entire section is optional, we do not recomend this if you are new to Linux, using a cloud server, or unfamilar with disk formatting.

Linux supports two major copy on write (COW) filesystems with transparent compression. ZFS and BTRFS. If you have the experience to do so we recomend formatting the disk that will be storing your blockchain data with one of these two.

Modern compression algorithms like [ZSTD](https://github.com/facebook/zstd) will reduce the disk space required for blockchain storage by 25% or more. Furthermore since your disk has to read and write 25% less data disk io actually improves. The tradeoff is minimal additional CPU usage and is by far worth it for a disk IO bound application like a Cosmos full node.

Furthermore both of these are Copy on Write (COW) systems. When you 'copy' a folder instead of actually duplicating all that data on disk a virtual copy is made and two folders are shown. When you edit any files in the first folder a copy is then made only of the file you edited or 'wrote' to. This means you can instantly make a backup of a 200gb blockchain data folder with only seconds of downtime and without needing enough disk space to store two entire copies of the blockchain.

#### BTRFS

For BTRFS simply format at data partiton and select 'BTRFS' as the filesystem. You may need to install the `btrfs-progs` package for your distribution. Edit the mount options to include `-o compress=zstd` and reboot.

#### ZFS

While BTRFS is very easy to get started with and provides our two key advantages (compression and copy on write) it very quickly requires expert advice as you get into more advanced multi-disk configurations. If you want to have reundant disks we _strongly recomend_ ZFS over BTRFS. [ZFS guide](https://openzfs.github.io/openzfs-docs/Getting%20Started/index.html)

### Increase open file limits

/// flesh out this section including the actual commands to edit limits.conf and
/// an example systemd file from the community. Note to the reader that they can do this step
/// OR use the systemd file with it's own raised limits

Ususally they say to add something like this to limits.conf

```
*    soft nofile 64000
*    hard nofile 64000
```

But if you're running as SU I think, then you also have to add

root soft nofile 64000  
root hard nofile 64000

I'd stick that in there somewhere. Maybe this is common knowledge.

## Validator setup

Now that we have completed the system preparation lets start setting up the blockchain

### Download the Gravity bridge binary

```
wget https://updates.althea.net/cgb-releases/gravity
chmod +x gravity
mv gravity /usr/bin/gravity
gravity init <your moniker here>
gravity keys add <your validator key name>
gravity keys add <your orchestrator key name>
```

### Create a GenTX

wget https://updates.althea.net/cgb-releases/cgb-testnet1-genesis.json
wget https://updates.althea.net/cgb-releases/gen_eth_key
chmod +x gen_eth_key
./gen_eth_key
mv cgb-testnet1-genesis.json ~/.gravity/config/genesis.json
gravity gentx --moniker <your moniker> <your validator key name> 50000000000000ugraviton <orchestrator eth address> <orchestrator address> --chain-id=cgbtestnet1

### Submit your gentx

In order to submit your gentx open up a pull request to this repo placing your gentx into the 'gentxs' folder
our automated test will check it for validity and after that it will be merged.
