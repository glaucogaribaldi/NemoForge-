# INSTALLATION & QUICK START GUIDE: NEMOFORGE V1.0
## *Deployment, Environment Setup, and CLI Execution*

This guide provides step-by-step instructions on how to install dependencies, configure environment paths, and run the **NemoForge V1.0** suite on both the remote VPS and the local Control Plane (Zava U50).

---

## 1. PRE-REQUISITES & DEPENDENCIES

NemoForge requires **Python 3.10+** (designed and tested on Python 3.13 and Python 3.14). 

### Required Python Packages:
*   `ccxt` (Exchange API connectors)
*   `pandas` (Historical data parsing and outer-joins)
*   `numpy` (Numerical mathematical arrays)
*   `matplotlib` (Independent unscaled chart rendering)
*   `requests` (API transport for remote Nemotron-30B)

---

## 2. STANDALONE VPS INSTALLATION

To run NemoForge directly on the VPS with 100% independence, follow these commands over SSH:

```bash
# 1. SSH into the VPS
ssh tre@100.73.54.72

# 2. Navigate to the storage-next workspace directory
cd /broker/storage/storage-next

# 3. Activate the virtual environment
source ./venv/bin/activate

# 4. Install the required plotting and CCXT dependencies inside the venv
python3 -m pip install pandas numpy matplotlib ccxt requests

# 5. Verify that the CLI is fully accessible and executable
export PYTHONPATH=/broker/storage/storage-next
python3 ./nemoforge_cli.py --help
```

---

## 3. COMMAND LINE INTERFACE (CLI) PLANCIA DI CONTROLLO

NemoForge V1.0 is operated through the unified command plancia `nemoforge_cli.py`.

### A. Dynamic Historical Data Ingestion
To scrape 365 days of 1-minute historical candles from Kraken Futures sequentially and calmly (rate-limited sleep of 3 seconds per page, 10 seconds per contract):
```bash
# Starts the bulk downloader in the background to avoid SSH termination
nohup python3 -c "import sys; sys.path.append('.'); import nemoforge.run_bulk_ingestion as rbi; rbi.run_bulk()" > ./logs/bulk_ingest.out 2>&1 &
```

### B. Running a Lacus Simulation (Backtest)
To run a deterministic backtest using a downloaded CSV historical candle dataset:
```bash
python3 nemoforge_cli.py backtest --file ./data/history/BTC_EUR_1m.csv --symbol PF_XBTUSD --capital 50000.0 --fee 0.0026 --slippage 0.001
```

### C. Executing Meta-Prompt Optimization (MPO)
To run the prompt optimizer as part of the self-training routine, asking Nemotron-30B (Port 8080) to evaluate past failure logs and rewrite its system prompt:
```bash
python3 nemoforge_cli.py optimize --db ./db/nemotron.sqlite
```

### D. Profiling VPS Resources & GPU Telemetry
To monitor real-time VPS CPU/GPU load (NVIDIA Tesla T4), VRAM, available margin, and API latencies:
```bash
python3 nemoforge_cli.py status
```

---

## 4. CRITICAL TROUBLESHOOTING & DESIGN GUARANTEES

*   **ModuleNotFoundError: No module named 'nemoforge'**  
    This error occurs when the parent workspace directory is not added to Python's search path. To resolve this, always set the environment variable:  
    `export PYTHONPATH=/broker/storage/storage-next` (or your local workspace root).
*   **Failed to make directory '/run/screen': Permission denied**  
    Many Debian/Ubuntu configurations restrict screen creation for non-sudo users. To launch NemoForge processes in the background reliably without screen, always use `nohup <command> &` or python's `subprocess.Popen` with `start_new_session=True`.
