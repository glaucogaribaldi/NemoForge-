# SYSTEM ARCHITECTURE: NEMOFORGE V1.0
## *Quantitative & Cognitive System Architecture*

This document provides a deep-dive technical specification of the **NemoForge V1.0** self-training and prompt-engineering suite. It is designed to act as an authoritative system reference for developers and auditing AIs.

---

## 1. HIGH-LEVEL FLOW & DESIGN PHILOSOPHY

NemoForge operates as a **Sovereign Closed-Loop Learning System**. Unlike standard static automated bots that execute hardcoded rules, NemoForge is designed to analyze its own live execution performance dynamically, isolate bottlenecks, and re-train its own cognitive system instructions as outlined below:

```
+-----------------------------------------------------------------------+
|                       KRAKEN EXCHANGE (API)                           |
+-------------------+-------------------------------+-------------------+
                    | (Live prices & balances)      ^ (Simulated orders)
                    v                               |
+-------------------+-------------------------------+-------------------+
|               SOVEREIGN CONTROL PLANE (Zava U50)                      |
|  - OpenClaw Orchestrator                                              |
|  - User Interface & Telegram Messaging Gateways                       |
+-------------------+-------------------------------+-------------------+
                    | (Tailscale / SSH)             ^ (Pre-rendered PNGs)
                    v                               |
+-------------------+-------------------------------+-------------------+
|               AUTONOMOUS EXECUTION PLANE (VPS)                        |
|                                                                       |
|  [NemoForge Ingestion Engine]                                         |
|    - Dynamically queries all active contracts (Spot/Futures)          |
|    - Progressively scrapes 1m deep OHLCv history (con metodo e calma)  |
|                                                                       |
|  [Lacus Backtest Engine]                                              |
|    - Executes mock orders over downloaded history                     |
|    - Computes simulated commission friction and slippage              |
|                                                                       |
|  [Meta-Prompt Optimizer - MPO]                                        |
|    - Queries episodic_memory SQLite DB for failed trades              |
|    - Sends post-mortems to local Nemotron-30B (Port 8080)             |
|    - Mutates and overwrites system prompts autonomously                |
|                                                                       |
|  [SentinelProf Telemetry Engine]                                      |
|    - Profiles NVIDIA Tesla T4 GPU usage & available margin           |
|                                                                       |
+-----------------------------------------------------------------------+
```

---

## 2. REVOLUTIONARY MODULE BREAKDOWNS

### 📊 1. Lacus Backtest Engine (`lacus_engine.py`)
`LacusEngine` is a lightweight, high-performance backtester designed to run locally on the VPS. 
*   **Agnostic Token Replay:** No assets or contracts are hardcoded. It takes any valid pandas DataFrame containing historical 1-minute Kraken candles and replays them chronologically.
*   **Friction & Slippage Modeling:** It implements simulated commissions (typically matching Kraken Starter tiers of 0.26% for Spot and 0.05% for Futures) and custom slippage rates.
*   **Equity & Drawdown Tracking:** It dynamically updates the unified portfolio equity (capital + unrealized PnL) and computes rolling peak equity and drawdowns, providing identical financial arithmetic to the production paper broker.

### 🧠 2. Meta-Prompt Optimizer (`prompt_optimizer.py`)
The `PromptOptimizer` bridges the gap between raw execution logs and AI prompt configuration:
*   **SQLite Post-Mortems:** It queries the SQLite table `episodic_memory` for recent losing trades. It extracts the logged regime, strategist view, mentor advice, trader decision, and final negative PnL.
*   **Cognitive Self-Reflection:** It prompt-engineers the local **NVIDIA Nemotron 30B GGUF** model with the baseline prompt and these 5 post-mortems, instructing it to identify its own cognitive biases (e.g. over-leveraging, ignoring risk mentor, or over-trading).
*   **Autonomous Mutation:** Nemotron rewrites the prompt, injecting specific hard negative constraints (Vetos) into its own system instructions to block those failure patterns.
*   **XML Sanitization:** The optimizer automatically cleans up any reasoning XML tags (`<think>...</think>`) and markdown fences from the response, saving a clean prompt to disk.

### ⏱️ 3. Bulk Ingestion Engine (`ingest_history.py` & `run_bulk_ingestion.py`)
To avoid hitting Kraken API rate limits or overloading the VPS CPU and memory, the ingestion engine implements a highly respectful "metodo e calma" pipeline:
*   **Dynamic Discovery:** It fetches all active Futures contracts on Kraken dynamically at runtime.
*   **Rate-Limited Chunking:** It fetches daily slices (max 720 bars per call), sleeping for **3.0 seconds** between pages, and **10.0 seconds** between symbols.
*   **Memory Efficiency:** It periodically flushes candles to disk to prevent RAM bloat and automatically cleans up duplicate rows on completion.

---

## 3. DESIGN GUARANTEES & SAFEGUARDS

*   **100% Paper-Only Safety:** The suite runs on read-only Kraken credentials for starting balance synchronization, and uses isolated local paper broker workspaces for simulated order execution. There is physically no code path capable of placing real exchange orders.
*   **Zero Caching Issues:** To prevent chat applications (like Telegram) from sending cached versions of previously rendered charts, the plotting engine `generate_charts.py` writes output files suffixed with unique run IDs or timestamps.
*   **Complete VPS Independence:** All heavy dependencies (`matplotlib`, `pandas`, `ccxt`, `numpy`) are installed inside the VPS's own virtualenv, and plotting is run directly on the VPS. Zava U50 is purely a lightweight administrative Control Plane.
