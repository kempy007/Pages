---
title: Unofficial backup/restore of polkadot rocksdb
parent: 2022
tags: blockchain polkadot hacking
---
# Unofficial backup/restore of polkadot rocksdb

Whilst on a mission to discover faster ways to deploy blockchain infrastructure, I eventually decided to poke around at the db level, the export-blocks feature of polkadot is lovely, however the import-blocks is not so great since it reruns consensus and thus costs more compute and time. 
Why would anyone be motivated to look this deep when all the data is on the p2p network, well it takes a long time, which keeps growing, anywhere from a few days to weeks to get your chain caught up and zero blocks behind the network.
If you look at what polkashots have done is they archive the filesystem, however compression add more time to the job, so they decided to offer a pruned dataset to which you have to run with `unsafe-pruning`. 
I mean if you've done infra for some time you'd probably want to use SAN snapshots, but that won't cut cross cloud.

I'm pretty certain this method will not be supported by parity.


## prerequisites 

`brew install rocksdb snappy`


## rocksdb native backup

`rocksdb_ldb --db=/path/to/chains/network/db/full backup --backup_dir=/tmp/backup/ --stderr_log_level=1`

## rocksdb native restore

`rocksdb_ldb --db=/path/to/chains/network/db/full restore --backup_dir=/tmp/backup/ --stderr_log_level=1`

### References

- http://blog.mitchseymour.com/using-ldb-to-inspect-kafka-streams-state-stores/
- https://review.spdk.io/gerrit/plugins/gitiles/spdk/rocksdb/+/6b2c1d962052cf1b5e6ebd24d74db22bf827ba05/INSTALL.md

#### 30th of June 2022
