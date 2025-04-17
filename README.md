
# Pantheon Congestion Control Evaluation

This repository contains experiments comparing three TCP congestion control algorithms using [Pantheon](https://github.com/StanfordSNR/pantheon) and [Mahimahi](https://github.com/ravinet/mahimahi). The evaluation covers **Cubic**, **Vegas**, and **SCReAM** under two network profiles.

---

## Experiment Summary

### Protocols Tested
- **Cubic**
- **Vegas**
- **SCReAM**

### Network Profiles
1. **Low-latency, high-bandwidth:** 50 Mbps, 10 ms RTT  
2. **High-latency, low-bandwidth:** 1 Mbps, 200 ms RTT

### Metrics Collected
- Throughput (time-series and average)
- RTT (time-series, average, and 95th percentile)
- Packet Loss (time-series and aggregate)

---

## Setup Instructions

### 1. Install Pantheon and Mahimahi
```bash
git clone https://github.com/StanfordSNR/pantheon.git
cd pantheon
sudo ./tools/install_deps.sh
git submodule update --init --recursive
```

### 2. Install Mahimahi (if not available)
```bash
sudo apt-get install mahimahi
```

Or build from source:
```bash
git clone https://github.com/ravinet/mahimahi
cd mahimahi
./autogen.sh
./configure
make
sudo make install
```

---

## 📁 Directory Structure

```
pantheon/
├── results/
│   ├── low-latency-high-bandwidth/       # Logs for 50Mbps scenario
│   └── high-latency-low-bandwidth/       # Logs for 1Mbps scenario
├── traces/
│   ├── 50mbps_uplink.trace
│   ├── 50mbps_downlink.trace
│   ├── 1mbps_uplink.trace
│   └── 1mbps_downlink.trace
├── analysis/
│   └── rtt_vs_throughput_plot.py         # Custom plot script
```

---

## Running Experiments

### Generate trace files
```bash
# 50 Mbps — 60s, 4 packets/ms
python2 -c "import sys; [sys.stdout.write('1500\n') for _ in range(4167)]" > /traces/50mbps.trac

# 1 Mbps — 1 packet every 12ms
python2 -c "import sys; [sys.stdout.write('1500\n') for _ in range(83)]" > /traces/1mbps.trace
```

### Run 50 Mbps scenario
```bash
python src/experiments/test.py local   --schemes "cubic vegas scream"   --data-dir results/low-latency-high-bandwidth/   --runtime 60   --uplink-trace traces/50mbps.trace   --downlink-trace traces/50mbps.trace   --prepend-mm-cmds "mm-delay 5"   --extra-mm-link-args "--uplink-queue=droptail --downlink-queue=droptail --uplink-queue-args=packets=500 --downlink-queue-args=packets=500"
```

### Run 1 Mbps scenario
```bash
python src/experiments/test.py local   --schemes "cubic vegas scream"   --data-dir results/high-latency-low-bandwidth/   --runtime 60   --uplink-trace traces/1mbps.trace   --downlink-trace traces/1mbps.trace   --prepend-mm-cmds "mm-delay 100"   --extra-mm-link-args "--uplink-queue=droptail --downlink-queue=droptail --uplink-queue-args=packets=500 --downlink-queue-args=packets=500"
```

---

## Analyzing Results

### Generate Pantheon Report
```bash
python ./src/analysis/analyze.py --data-dir results/low-latency-high-bandwidth
python ./src/analysis/analyze.py --data-dir results/high-latency-low-bandwidth
```
---

## Outputs Committed

- Raw logs: `results/*/*.log`
- Metadata: `pantheon_perf.json`, `pantheon_metadata.json`
- Plots: `pantheon_report.pdf`

---

