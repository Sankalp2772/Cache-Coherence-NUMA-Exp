# Cache Coherence and NUMA Performance Experiments

A systems lab project exploring cache coherence protocols, false sharing, and NUMA memory access using Python, C, and OpenMP.

## Experiments

### 1. Cache coherence protocol simulation

Simulates cache-line behaviour for MSI, MESI, and MOESI. The simulator compares adjacent/shared cache-line access with padded cache-line access and reports bus transactions, BusRd, upgrades, invalidations, writebacks, cache-to-cache transfers, and silent state transitions.

### 2. False sharing

An OpenMP C benchmark compares adjacent counters with cache-line-padded counters.

Example measurements from the local WSL2 environment:

| Threads | Adjacent median (s) | Padded median (s) |
|---:|---:|---:|
| 1 | 0.0446 | 0.0490 |
| 2 | 0.0416 | 0.0216 |
| 4 | 0.0354 | 0.0189 |

These measurements are environment-dependent and subject to timing noise. Padding reduced median runtime at 2 and 4 threads in these runs; the single-thread result showed no benefit.

### 3. NUMA memory sweep

A C/OpenMP benchmark sweeps a memory buffer and reports a checksum and elapsed time.

Example local-node measurements (512 MB, four passes):

| Threads | Time (s) |
|---:|---:|
| 1 | 0.1683 |
| 2 | 0.0933 |

### 4. Local versus remote NUMA placement

The remote-node comparison was **not completed**. The WSL2 and VMware Ubuntu environments used for this lab exposed only one NUMA node (`numactl --hardware` reported `available: 1 nodes (0)`). Therefore, only node-0 local placement could be tested. No remote-NUMA timing result is claimed.

The host is an Intel Core i5-12450H single-socket laptop CPU. Virtual NUMA domains, if exposed by VMware, would not by themselves establish genuine remote physical-memory access on this host.

## Environment

- Ubuntu Linux (WSL2 and VMware VM used at different stages)
- GCC with OpenMP support
- `numactl` and `hwloc`
- Python 3
- Intel Core i5-12450H host CPU

Install the primary tools on Ubuntu:

```bash
sudo apt update
sudo apt install build-essential numactl hwloc python3 -y
```

## Build and run

Compile the C benchmarks from their source files:

```bash
gcc -O2 -fopenmp false_sharing.c -o false_sharing
gcc -O2 -fopenmp numa_sweep.c -o numa_sweep
gcc -O2 -fopenmp compare_numa.c -o compare_numa
```

Run the Python coherence simulator:

```bash
python3 coherence_sim.py 1000
```

Run false-sharing tests:

```bash
OMP_NUM_THREADS=2 ./false_sharing adjacent
OMP_NUM_THREADS=2 ./false_sharing padded
```

Run available local NUMA-node tests:

```bash
OMP_NUM_THREADS=1 numactl --cpunodebind=0 --membind=0 ./numa_sweep
OMP_NUM_THREADS=2 numactl --cpunodebind=0 --membind=0 ./numa_sweep
```

Run the comparison program:

```bash
./compare_numa
```

Check command-line options against the actual source files before running.

## Measurement notes

- Repeat performance runs (three runs were used) and compare medians.
- Runtime is affected by scheduling, background load, virtualization, and thermal or power management.
- Do not interpret virtual CPU socket count as proof of multiple NUMA memory nodes. Verify with `lscpu` and `numactl --hardware`.

## Repository status

This README records the available results and the remote-NUMA limitation. Add the project source files and any raw output files alongside it.