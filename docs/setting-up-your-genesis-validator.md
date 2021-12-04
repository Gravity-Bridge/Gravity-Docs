# Gravity Genesis Bridge Validator Setup

## System setup

While Gravity Bridge full nodes can be run on any platform it is recommended you run your validator on Linux. The most up to date version of any particular distribution is recommended.

### Recomended specs

recommended specs are 8gb ram and a quad core CPU. Storage specs will change over time as the chain grows, start with at least 125gb of SSD space. Remember if you are not using a Copy on write filesystem you will need enough storage to keep a backup of the entire blockchain during upgrades.

### Optional: Copy on Write + Compressing filesystem

This entire section is optional, we do not recommend this if you are new to Linux, using a cloud server, or unfamiliar with disk formatting.

Linux supports two major copy on write (COW) filesystems with transparent compression. ZFS and BTRFS. If you have the experience to do so we recommend formatting the disk that will be storing your blockchain data with one of these two.

Modern compression algorithms like [ZSTD](https://github.com/facebook/zstd) will reduce the disk space required for blockchain storage by 25% or more. Furthermore since your disk has to read and write 25% less data disk io actually improves. The tradeoff is minimal additional CPU usage and is by far worth it for a disk IO bound application like a Cosmos full node.

Furthermore both of these are Copy on Write (COW) systems. When you 'copy' a folder instead of actually duplicating all that data on disk a virtual copy is made and two folders are shown. When you edit any files in the first folder a copy is then made only of the file you edited or 'wrote' to. This means you can instantly make a backup of a 200gb blockchain data folder with only seconds of downtime and without needing enough disk space to store two entire copies of the blockchain.

#### BTRFS

For BTRFS simply format at data partiton and select 'BTRFS' as the filesystem. You may need to install the `btrfs-progs` package for your distribution. Edit the mount options to include `-o compress=zstd` and reboot.

#### ZFS

While BTRFS is very easy to get started with and provides our two key advantages (compression and copy on write) it very quickly requires expert advice as you get into more advanced multi-disk configurations. If you want to have reundant disks we _strongly recommend_ ZFS over BTRFS. [ZFS guide](https://openzfs.github.io/openzfs-docs/Getting%20Started/index.html)

### Increase open file limits

/// flesh out this section including the actual commands to edit limits.conf and
/// an example systemd file from the community. Note to the reader that they can do this step
/// OR use the systemd file with it's own raised limits

Ususally they say to add something like this to limits.conf

```text
*    soft nofile 64000
*    hard nofile 64000
```

But if you're running as SU I think, then you also have to add

```text
root soft nofile 64000
root hard nofile 64000
```

I'd stick that in there somewhere. Maybe this is common knowledge.

## Validator setup

Now that we have completed the system preparation lets start setting up the blockchain

# Firewall Settings

This is optional but greatly helps with the health of the network. 

## Open the inbound P2P port
```
sudo ufw allow from any to any port 26656 proto tcp
```

