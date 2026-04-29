---
title: Client Resource Usage
sidebar_position: 11
sidebar_label: Client Resource Usage
---

# Consensus Clients

| Client | Version | Date | DB Size  |  RAM | Notes |
|--------|---------|----  |----------|------|-------|
| Teku   | 24.8.0  | Sep 2024 | ~84 GiB | ~10 GiB |
| Lighthouse | 4.5.0  | Jan 2024 | ~130 GiB | ~5 GiB |
| Nimbus | 24.1.1 | Jan 2024 | ~170 GiB | ~2 to 3 GiB |
| Prysm | 4.1.1 | Jan 2024 | ~130 GiB | ~5 GiB |
| Lodestar | 1.13.0 | Jan 2024 | ~130 GiB | ~8 GiB |

Notes on disk usage
- Teku, Nimbus, Lodestar, Prysm and Grandine continuously prune
- Lighthouse can be resynced in minutes to bring space usage back down, with `./ethd resync-consensus`
- Lighthouse is working on tree states to continuously prune

# Execution clients

For reference, here are disk, RAM and CPU requirements, as well as mainnet initial synchronization times, for different Ethereum execution clients.

## Disk and RAM requirements

SSD and RAM use is after initial sync, when keeping up with head.

Please pay attention to the Version and Date. These are snapshots in time of client behavior. Initial database size increases over time, and execution clients are always working on improving their storage engines.

DB Size is shown with values for different types of nodes: Full, and different levels of expiry: Post-Merge history only; Post-Cancun history only; rolling expiry; aggressive expiry.
"tbd" means I haven't gathered the data. "n/a" means the client does not support this expiry mode, yet.

| Client | Version | Date | DB Full | DB Post-Merge | DB Post-Prague | DB Rolling | DB Aggressive | RAM | Notes |
|--------|---------|------|---------|---------------|----------------|------------|---------------|-----|-------|
| Geth   | 1.17.2 | April 2026 | ~1.2 TiB | ~830 GiB | ~580 GiB | n/a | n/a | ~ 8 GiB | |
| Nethermind | 1.36.2 | April 2026 | ~1.1 TiB | ~740 GiB | ~468 GiB | ~240 GiB | n/a | ~7 GiB | With HalfPath, can automatic online prune at ~350 GiB free |
| Nethermind | 1.37.1 | April 2026 | tbd | ~1 TiB  | ~500 GiB | ~255 GiB | n/a | tbd GiB | With FlatInTrie FlatDB |
| Nethermind | 1.37.1 | April 2026 | tbd | ~1.1 TiB  | ~550 GiB | ~310 GiB | n/a | tbd GiB | With Flat FlatDB |
| Besu | v26.1.0 | February 2026 | ~1.35 TiB | ~850 GiB | n/a | ~560 GiB | ~290 GiB | ~10 GiB | |
| Reth | 2.0.0 | April 2026 | tbd | ~1 TiB  | ~475 GiB | ~490 GiB | ~248 GiB | ~14 GiB | with receipts, exception "Aggressive" |
| Erigon | 3.3.8 | February 2026 | ~1.0 TiB | ~650 GiB | n/a | ~640 GiB | ~355 GiB | See comment | Erigon will have the OS use all available RAM as a DB cache during post-sync operation, but this RAM is free to be used by other programs as needed. During sync, it may run out of memory on machines with 32 GiB or less |
| Nimbus | 0.1.0-alpha | May 2025 | tbd | ~755 GiB | n/a | n/a | n/a | | With Era1 import |
| Ethrex | 10.0.0-rc.1 | March 2026 | n/a | ~300 GiB | n/a | n/a | n/a | ~16 GiB | |

Notes on disk usage
- All clients other than "Nethermind with HalfPath DB" continously prune state
- Nethermind - HalfPath DB state size can be reduced when it grew too large, by [online prune](NodeTypes.md).

## Initial sync times

Please pay attention to the Version and Date. Newer versions might sync faster, or slower.

These are initial syncs of a node with a stated amount of history expiry. For clients that support it, snap sync was used; otherwise, full sync.

Cache size default in all tests.

| Client | Version | Date | Node Type | Test System | Time Taken |  Notes |
|--------|---------|------|-----------|-------------|------------|--------|
| Geth   | 1.17.2  | April 2026 | post-Prague | Netcup RS G11 | ~ 3 hours | |
| Nethermind | 1.36.0| February 2026 | post-Cancun | Netcup RS G11 | ~ 2 hours | Ready to attest after ~ 1 hour |
| Besu | v26.1.0 | February 2026 | rolling | Netcup RS G11 | ~ 13 hours | |
| Erigon | 3.3.8 | February 2026 | rolling | Netcup RS G11 | ~ 12 hours | |
| Reth  | 2.0.0 | April 2026 | Full | Legacy miniPC | ~ tbd days | full sync, no snapshot |
| Reth  | 2.0.0 | April 2026 | post-Prague | Netcup RS G11 | ~ 2 hours  | with DB snapshot |
| Nimbus | 0.1.0-alpha | May 2025 | Full | OVH Baremetal NVME | ~ 5 1/2 days | With Era1 import |
| Ethrex | 10.0.0-rc.1 | March 2026 | post-merge | Netcup RS G11 | ~ 2 hours | |

## Test Systems

Latency is what matters most to Ethereum clients. Measure it with `sudo ioping -D -c 30 /dev/<ssd-device>` during load. Ideally while running a client, but using an `fio` to generate
synthetic load will also get you a ballpark figure. You'd want to be under 300 us max (microseconds, not milliseconds) for an Ethereum execution client. High latency negatively impacts
attestation performance, and is particularly noticeable during sync committee duties.

Synthetic load can be generated with `fio --randrepeat=1 --ioengine=libaio --direct=1 --gtod_reduce=1 --name=test --filename=test --bs=4k --iodepth=64 --size=150G --readwrite=randrw --rwmixread=75; rm test`. If the test shows it'd take hours to complete, feel free to cut it short once the IOPS display for the test looks steady.

150G was chosen to "break through" any caching strategems the SSD uses for bursty writes. Execution clients write steadily, and the performance of an SSD under heavy write is more important than its performance with bursty writes.

Servers have been configured with [noatime](https://www.howtoforge.com/reducing-disk-io-by-mounting-partitions-with-noatime) and [no swap](https://www.geeksforgeeks.org/how-to-permanently-disable-swap-in-linux/) to improve latency.


| Name                 | RAM    | SSD Size | CPU        | r/w latency | Notes |
|----------------------|--------|----------|------------|-------------|-------|
| [OVH](https://ovhcloud.com/) Baremetal NVMe   | 32 GiB | 1.9 TB  | Intel Hexa | 150us max | Datacenter-class NVMe drive |
| [Netcup](https://netcup.eu) RS G11 | 96 GiB | 3 TB | 20 vCPU on an AMD 84-core | 400us avg / 1.1ms max | Storage is fast enough to attest, but too slow to get best rewards |
| Legacy miniPC | 32 GiB | 2 TB | Intel Quad 6th gen | 230 us avg / 320 us max | Home staker setup with PCIe 3 NVMe and older CPU |

## Getting better latency

Ethereum execution layer clients need decently low latency. Measure latency with `ioping` when the system is under load. NVMe SSD is highly recommended; HDD will not be sufficient.

For cloud providers, here are some results for syncing Geth. In a nutshell, use baremetal instead.
- AWS, gp2 or gp3 with provisioned IOPS delivered sub-par performance during sync committees.
- Linode block storage, make sure to get NVMe-backed storage.
- Netcup RS G11 works, but rewards are not optimal.
- There are reports that Digital Ocean block storage is too slow, as of late 2021.
- Strato V-Server is too slow as of late 2021.

Dedicated servers with NVMe SSD will always have sufficiently low latency. Do avoid hardware RAID though, see below.
OVH Advance line is a well-liked dedicated option; latitude.sh, Linode, Vultr or Strato or any other baremetal provider will work as well.

For own hardware, we've seen three causes of high latency:
- DRAMless or QLC SSD. Choose a ["mainstream" SSD](https://gist.github.com/yorickdowne/f3a3e79a573bf35767cd002cc977b038)
with TLC and DRAM. Enterprise / data center SSDs will always work great; consumer SSDs vary.
- Overheating of the SSD. Check `smartctl -x`. You want the SSD to be at ~ 40-50 degrees Celsius, so it does not
throttle.
- Hardware RAID, no TRIM support. [Flash the controller](https://gist.github.com/yorickdowne/fd36009c19fdbee0337bffc0d5ad8284)
to HBA and use software RAID.
