# NemoForge V1.0
## *The Sovereign Self-Training & Strategy Optimization Suite*

Welcome to **NemoForge V1.0**, the dedicated, production-grade self-training and quantitative strategy optimization suite built for `krakenfondazione`.

NemoForge is designed as a standalone, self-adaptive, and 100% independent ecosystem operating entirely on the high-performance remote VPS. It allows the heavy NVIDIA Nemotron 30B GGUF model to autonomously collect historical market data, backtest strategies, analyze execution failures, and rewrite its own prompt-engineering rules to continuously evolve and outsmart market friction.

---

## 🏛️ CORE ARCHITECTURE & PILLARS

NemoForge replaces static signal execution with a closed-loop autonomous learning engine consisting of four decoupled modules:

1.  **Lacus Backtest Engine (`lacus_engine.py`):**  
    A high-speed, deterministic backtesting simulator that replays historical Kraken OHLC and L2 book data. It models leverage, margin limits, liquidations, and simulates trading friction (custom taker fee rate up to 0.26% and custom slippage rate) to calculate the exact *Fee Drag* before live deployment.
2.  **Meta-Prompt Optimizer (MPO — `prompt_optimizer.py`):**  
    The cognitive core of self-adaptation. It connects to the local Nemotron-30B server, extracts losing trade episodes from the active SQLite database (`episodic_memory`), performs a silent logical post-mortem, and writes mutated, optimized system prompts to prevent those specific errors from reoccurring.
3.  **SentinelProf (`telemetry_profiler.py`):**  
    A passive background telemetric sweep that tracks GPU VRAM usage, inference latencies, order rejection rates (`Insufficient margin`), and slippage leakage during live production runs.
4.  **Bulk Ingestion Scraper (`ingest_history.py` & `run_bulk_ingestion.py`):**  
    A sequential, highly respectful, and rate-limited scraper that downloads deep 1-minute historical candles (up to 365 days) for all active Kraken contracts dynamically, without triggering API rate limits or overloading the VPS.

---

## 📂 REPOSITORY DIRECTORY TREE

```text
NemoForge/
  ├── nemoforge/
  │     ├── __init__.py
  │     ├── lacus_engine.py       # High-speed deterministic paper simulator (Spot/Futures)
  │     ├── prompt_optimizer.py   # Meta-Prompt Optimizer (Post-mortems via Nemotron 30B)
  │     ├── telemetry_profiler.py # SentinelProf (GPU, latency, and margin telemetry)
  │     ├── ingest_history.py     # Paginated and rate-limited 1-minute candle downloader
  │     ├── run_bulk_ingestion.py # Bulk dynamic scannable contract scraper
  │     └── generate_charts.py    # VPS-independent unscaled Matplotlib chart renderer
  │
  ├── nemoforge_cli.py            # Unified command-line interface plancia di controllo
  ├── run_config.json             # Default simulation and live loop parameters
  ├── README.md                   # This file
  ├── ARCHITECTURE.md             # Deep architectural and system specifications
  └── INSTALL.md                  # Quickstart, dependencies, and execution guide
```

---

## 🚀 QUICK START & EXECUTION

NemoForge is controlled entirely through the `nemoforge_cli.py` plancia:

```bash
# 1. Download 365 days of 1-minute candles for a specific contract
python3 nemoforge_cli.py download-history --pair BTC/EUR --timeframe 1m --limit 525600

# 2. Run a Lacus backtest over a downloaded historical dataset
python3 nemoforge_cli.py backtest --file ./data/history/BTC_EUR_1m.csv --symbol PF_XBTUSD

# 3. Trigger the Meta-Prompt Optimizer to audit recent losses and mutate the prompt
python3 nemoforge_cli.py optimize --db /broker/storage/storage-next/db/nemotron.sqlite

# 4. Check real-time VPS telemetry, including NVIDIA Tesla T4 GPU usage and latencies
python3 nemoforge_cli.py status
```

*For detailed setup, shebang troubleshooting, and dependency shepherding, please refer to [INSTALL.md](./INSTALL.md).*
