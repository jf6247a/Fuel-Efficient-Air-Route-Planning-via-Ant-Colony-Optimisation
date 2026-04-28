# Fuel-Efficient Distributed ACO 

> COMP1682 Final Year Project  
> Fuel-efficient air route planning via Ant Colony Optimisation (ACO) architecture.

---

## Table of Contents

- [Project Summary](#project-summary)
- [What This Project Adds to BlueSky](#what-this-project-adds-to-bluesky)
- [Core Idea](#core-idea)
- [Features](#features)
- [Repository Layout](#repository-layout)
- [How the Distributed ACO Works](#how-the-distributed-aco-works)
- [Data Sources](#data-sources)
- [Requirements](#requirements)
- [Installation](#installation)
  - [Windows / PyCharm Setup](#windows--pycharm-setup)
  - [Terminal-Only Setup](#terminal-only-setup)
- [Quick Start](#quick-start)
  - [Quick Start: Benchmark Only](#quick-start-benchmark-only)
  - [Quick Start: Full BlueSky Simulator](#quick-start-full-bluesky-simulator)
- [BlueSky Commands](#bluesky-commands)
- [Dashboard / UI Guide](#dashboard--ui-guide)
- [Generating New Data](#generating-new-data)
- [Running Tests](#running-tests)
- [Outputs and Where Files Are Saved](#outputs-and-where-files-are-saved)
- [Known Issues and Workarounds](#known-issues-and-workarounds)
- [Troubleshooting](#troubleshooting)
- [Suggested Demo Flow](#suggested-demo-flow)
- [Project Scope and Limitations](#project-scope-and-limitations)
- [Acknowledgements](#acknowledgements)

---

## Project Summary

This project extends **BlueSky**, the open air traffic simulator, with a research prototype for **fuel-aware route planning** using a **distributed Ant Colony Optimisation (ACO)** model.

The system compares three route-planning strategies:

1. **Shortest-path Dijkstra**  
   A weak baseline based on geometric route length.
2. **Wind-aware / fuel-aware Dijkstra**  
   A stronger deterministic baseline using the project’s fuel-cost model.
3. **Distributed ACO**  
   A swarm-based planner in which airport or region-level colonies propose routes, a master coordinator updates global pheromone knowledge, and that updated knowledge is fed back to the colonies.

The project was designed as a **final-year research prototype**, not as an operational airline dispatch system. It prioritises:

- explainability
- reproducibility
- BlueSky integration
- benchmark export
- parameter tuning
- honest evaluation

---

## What This Project Adds to BlueSky

This repository includes the standard BlueSky simulator plus new project-specific components:

- **`bluesky/plugins/fyp_aco_core.py`**  
  Core graph model, aircraft profiles, fuel model, deterministic baselines, distributed ACO implementation, benchmark runner, chart generator, and graph-building utilities.

- **`bluesky/plugins/fypaco.py`**  
  BlueSky simulation plugin exposing stack commands such as `ACOLOAD`, `ACOROUTE`, `ACOCREATE`, and `ACOBENCH`.

- **`bluesky/plugins/fypacogui.py`**  
  Qt dashboard for loading networks, changing learning parameters, planning routes, running benchmarks, and viewing result tables and graphs.

- **`project_data/fyp_aco_network.json`**  
  Default controlled airport network.

- **`scripts/run_fyp_benchmark.py`**  
  Standalone benchmark runner for regenerating CSV results and figures without opening the full simulator.

- **`tests/test_fyp_aco.py`**  
  Automated tests for core route-planning behaviour and benchmark output.

---

## Core Idea

The project tries to model route planning more like a distributed air traffic system and less like one giant all-knowing optimiser.

### Conceptual flow

1. Each **airport or region** is represented by a **local colony**.
2. Each colony explores candidate routes using local pheromone information plus global pheromone information.
3. The **master coordinator** collects the best local results.
4. The master updates the **global pheromone memory**.
5. The updated global pheromone state is fed back to all colonies.
6. The process repeats for multiple iterations.

This architecture mirrors the project proposal and contextual report more closely than a single centralised optimiser.

---

## Features

### Routing features
- shortest-path routing
- wind-aware deterministic routing
- distributed ACO routing
- aircraft-type-aware fuel multipliers
- congestion penalties
- altitude/climb penalties
- route metrics export

### BlueSky integration features
- load a saved network JSON
- build a network from BlueSky airport/nav data
- create flights from planned routes
- use live BlueSky traffic positions to influence congestion
- generate benchmark CSVs and charts from inside BlueSky

### UI features
- load network
- build network from BlueSky navdb
- choose origin, destination, strategy and aircraft type
- change ACO learning parameters
- toggle live data use
- run benchmarks
- inspect tables and graphs

### Research / evaluation features
- benchmark CSV generation
- convergence history export
- automatic chart creation
- reproducible standalone benchmark script
- automated tests

---

## Repository Layout

```text
bluesky-master/
├── BlueSky.py
├── README.md                     # Upstream BlueSky README
├── README_FYP_ACO.md            # Project-specific README (older version)
├── bluesky/
│   └── plugins/
│       ├── fyp_aco_core.py      # Main ACO logic
│       ├── fypaco.py            # BlueSky sim plugin bridge
│       └── fypacogui.py         # Qt dashboard plugin
├── project_data/
│   ├── fyp_aco_network.json     # Default network
│   ├── fyp_aco_results.csv      # Benchmark output
│   └── history/                 # Convergence histories and per-run data
├── report_assets/
│   ├── fuel_summary.png
│   ├── route_comparison.png
│   └── other dissertation figures
├── scenario/
│   └── fyp_aco_demo.scn         # Demo scenario
├── scripts/
│   └── run_fyp_benchmark.py     # Standalone benchmark entry point
└── tests/
    └── test_fyp_aco.py          # Automated tests
```

---

## How the Distributed ACO Works

The main implementation is in:

```text
bluesky/plugins/fyp_aco_core.py
```

### Main building blocks

- **`AirspaceGraph`**  
  Stores airports (nodes), route legs (edges), and lookup structures.

- **`AircraftProfile`**  
  Stores fuel-factor and performance-related values for aircraft such as A320, A321, B738, E190 and B744.

- **`DistributedACOPlanner`**  
  Runs the ACO search process.

- **`AirportColony`**  
  Represents local pheromone behaviour for a region.

- **`MasterCoordinator`**  
  Aggregates the best results from local colonies and updates the global pheromone state.

### Fuel-aware cost model

The route cost is based on several edge-level factors:

- distance
- wind factor
- congestion factor
- altitude penalty
- aircraft fuel multiplier

This is a **fuel proxy**, not a certified real-world fuel-burn model.

### Strategy behaviour

- **Shortest path** prioritises distance only.
- **Wind-aware Dijkstra** uses deterministic weighted route cost.
- **Distributed ACO** uses pheromone learning plus route attractiveness.

---

## Data Sources

This project uses a combination of:

1. **Controlled experimental data**  
   The default route network in `project_data/fyp_aco_network.json`.

2. **Public or simulator-derived data**  
   BlueSky airport/navigation data when building a graph from navdb.

3. **Simulator-live data**  
   BlueSky traffic positions when `ACOLIVE ON` is enabled.

### Important honesty note
This project does **not** use a full real-time commercial flight operations feed or a complete live aviation weather system.

A safe description is:

> The prototype combines a controlled experimental network with publicly available and simulator-derived data sources. It does not rely on full real-time operational aviation datasets, which was an intentional design choice made to preserve transparency, reproducibility and feasibility.

---

## Requirements

### Recommended Python version
- **Python 3.11** is the safest choice.

### Core Python packages
- `numpy`
- `scipy`
- `pandas`
- `matplotlib`
- `networkx`
- `pytest`
- `pyzmq`
- `msgpack`
- `pygame` or `pygame-ce`

### GUI packages
- `PyQt6`
- `PyQt6-WebEngine`

### Optional but useful
- `rtree`
- `python-docx` (only needed for report-generation scripts, not core routing)

---

## Installation

## Windows / PyCharm Setup

### 1. Extract the project zip
Extract the enhanced project and open:

```text
fyp_bluesky_project_enhanced\bluesky-master
```

### 2. Open the project in PyCharm
Open the **`bluesky-master`** folder, not a subfolder.

### 3. Create a virtual environment
Use a local interpreter in:

```text
bluesky-master\.venv
```

### 4. Install dependencies
Open the PyCharm terminal and run:

```powershell
python -m pip install --upgrade pip setuptools wheel
python -m pip install pyzmq msgpack numpy scipy matplotlib pandas pygame networkx pytest PyQt6 PyQt6-WebEngine
```

### 5. Start BlueSky
From the project root:

```powershell
python BlueSky.py
```

---

## Terminal-Only Setup

If you only want the benchmark script without the full BlueSky window:

```powershell
py -m venv .venv
.\.venv\Scripts\activate
python -m pip install --upgrade pip setuptools wheel
python -m pip install pyzmq msgpack numpy scipy matplotlib pandas pygame networkx pytest
python scripts\run_fyp_benchmark.py
```

---

## Quick Start

## Quick Start: Benchmark Only

This is the fastest reliable way to use the project.

```powershell
python scripts\run_fyp_benchmark.py
```

This will regenerate:

- `project_data/fyp_aco_results.csv`
- `project_data/history/*.csv`
- `project_data/history/*_convergence.png`
- `report_assets/fuel_summary.png`
- `report_assets/route_comparison.png`

---

## Quick Start: Full BlueSky Simulator

### 1. Launch BlueSky
```powershell
python BlueSky.py
```

### 2. Load the project plugins
In the BlueSky command stack:

```text
PLUGINS LOAD FYPACO
PLUGINS LOAD FYPACOGUI
```

### 3. Load the default network
```text
ACOLOAD project_data/fyp_aco_network.json
```

### 4. Set ACO parameters
```text
ACOCFG 36 40 1.15 3.0 0.25 200 0.2 42
```

### 5. Plan a route
```text
ACOROUTE ACO EGLL LIRF A320
```

### 6. Create a flight from that route
```text
ACOCREATE FYP100 A320 ACO EGLL LIRF
```

### 7. Run the benchmark
```text
ACOBENCH project_data/fyp_aco_results.csv A320
```

---

## BlueSky Commands

The simulation plugin exposes the following commands.

### `ACOLOAD path`
Load a project network JSON.

**Example**
```text
ACOLOAD project_data/fyp_aco_network.json
```

---

### `ACOBUILDNET airport_csv max_range_nm output_path`
Build a network from BlueSky navdb airport data.

**Important:** pass the airport list as **one quoted string**.

**Example**
```text
ACOBUILDNET "EGLL,EHAM,EDDF,LFPG,EIDW,LIRF,LEMD,LSZH,LOWW" 750 project_data/generated_network.json
```

Then load it with:

```text
ACOLOAD project_data/generated_network.json
```

---

### `ACOLIVE ON|OFF`
Toggle live BlueSky traffic coupling for congestion penalties.

**Example**
```text
ACOLIVE ON
```

---

### `ACOACTYPE ACTYPE`
Set the default aircraft profile.

**Example**
```text
ACOACTYPE B738
```

Supported examples include:
- `A320`
- `A321`
- `B738`
- `E190`
- `B744`

---

### `ACOCFG ants iterations alpha beta evaporation deposit_weight elite_fraction seed`
Update ACO learning parameters.

**Example**
```text
ACOCFG 36 40 1.15 3.0 0.25 200 0.2 42
```

Parameter meanings:

- **ants** = number of route-searching agents per iteration
- **iterations** = number of learning rounds
- **alpha** = trust in pheromone memory
- **beta** = trust in route attractiveness / heuristic cost
- **evaporation** = forgetting speed for old pheromone
- **deposit_weight** = reward size for good routes
- **elite_fraction** = fraction of best ants used most strongly for updates
- **seed** = random seed for repeatability

---

### `ACOROUTE strategy origin destination aircraft_type`
Plan a route without creating traffic.

**Example**
```text
ACOROUTE ACO EGLL EHAM A320
```

Possible strategies:
- `ACO`
- `WIND`
- `SHORTEST`

---

### `ACOCREATE acid actype strategy origin destination [alt] [spd]`
Create an aircraft in BlueSky and inject the chosen route.

**Example**
```text
ACOCREATE FYP100 A320 ACO EGLL LIRF
```

Optional values:
- altitude defaults to `35000`
- speed defaults to `450`

---

### `ACOBENCH output_csv aircraft_type`
Run the benchmark set and export results.

**Example**
```text
ACOBENCH project_data/fyp_aco_results.csv A320
```

---

### `ACOGRAPH csv_path charts_dir`
Regenerate graphs from an existing CSV.

**Example**
```text
ACOGRAPH project_data/fyp_aco_results.csv report_assets
```

---

### `ACOSUMMARY`
Return a short internal project summary.

---

## Dashboard / UI Guide

The GUI plugin file is:

```text
bluesky/plugins/fypacogui.py
```

Load it with:

```text
PLUGINS LOAD FYPACOGUI
```

### Main UI areas

#### Control tab
- network JSON path
- airport list for navdb network build
- origin and destination selectors
- route strategy selector
- aircraft type selector
- live traffic toggle
- ACO parameter controls
- action buttons

#### Graphs tab
- fuel comparison graph
- convergence graph

#### Data tab
- benchmark results table

### Main buttons
- **Load network** — uses the current network JSON path
- **Build from BlueSky navdb** — creates a graph from airport codes
- **Apply parameters** — updates ACO settings and aircraft/live-data state
- **Plan route** — runs route planning
- **Run benchmark** — generates fresh CSVs and charts
- **Refresh data** — refreshes visible UI data

---

## Generating New Data

There are three main ways to generate new data.

### 1. Rerun the standard benchmark
```powershell
python scripts\run_fyp_benchmark.py
```

### 2. Change parameters and rerun
For example:

```text
ACOACTYPE B738
ACOCFG 50 60 1.2 2.8 0.30 220 0.25 77
ACOBENCH project_data/fyp_aco_results_b738.csv B738
```

### 3. Build a new network and benchmark it
```text
ACOBUILDNET "EGLL,EHAM,EDDF,LFPG,EIDW,LIRF,LEMD,LSZH,LOWW" 750 project_data/generated_network.json
ACOLOAD project_data/generated_network.json
ACOBENCH project_data/generated_results.csv A320
```

### Example “fresh data” workflow
```text
PLUGINS LOAD FYPACO
ACOBUILDNET "EGLL,EHAM,EDDF,LFPG,EIDW,LIRF,LEMD,LSZH,LOWW,LEBL,ESSA,EKCH,EPWA" 900 project_data/generated_network_big.json
ACOLOAD project_data/generated_network_big.json
ACOACTYPE B738
ACOLIVE ON
ACOCFG 60 80 1.5 2.5 0.35 250 0.3 99
ACOBENCH project_data/fyp_aco_results_big.csv B738
```

---

## Running Tests

From the project root:

```powershell
pytest -q tests\test_fyp_aco.py
```

These tests are intended to verify that:
- the fuel-aware baseline is working
- the ACO route planner returns valid routes
- benchmark export works
- graph/history generation works

---

## Outputs and Where Files Are Saved

### Networks
- `project_data/fyp_aco_network.json`
- `project_data/generated_network.json`
- other custom JSON networks you create

### Benchmark CSVs
- `project_data/fyp_aco_results.csv`
- any custom benchmark CSV path you provide

### History files
- `project_data/history/`

### Report figures / charts
- `report_assets/fuel_summary.png`
- `report_assets/route_comparison.png`
- convergence plots written during benchmark runs

---

## Known Issues and Workarounds

### 1. Stack commands say they take 0 arguments
**Symptom**
- `ACOLOAD takes 0 argument`
- `ACOROUTE takes 0 argument`

**Cause**
The current plugin build includes:

```python
from __future__ import annotations
```

inside `bluesky/plugins/fypaco.py`, which can interfere with BlueSky’s stack-command parser.

**Workaround**
Remove that line from:

```text
bluesky/plugins/fypaco.py
```

Then restart BlueSky.

If needed, remove it from:

```text
bluesky/plugins/fypacogui.py
```

as well.

---

### 2. The dashboard says `FYPACO is not loaded yet` even when the plugin loaded
**Symptom**
The console says:

```text
Successfully loaded plugin FYPACO
Successfully loaded plugin FYPACOGUI
```

but the dashboard still shows:

```text
FYPACO is not loaded yet.
```

**Cause**
BlueSky sim plugins and gui plugins run in different processes. The current GUI implementation tries to access the sim plugin too directly.

**Workaround**
Treat the console/plugin load messages as the real source of truth. Use stack commands first. A fuller fix would rewire the GUI to communicate through BlueSky stack commands instead of direct Python plugin references.

---

### 3. `ACOBUILDNET` complains that it cannot convert airport codes to float
**Symptom**
Example error:

```text
could not convert string to float: 'EHAM'
```

**Cause**
The airport list was not passed as one quoted string.

**Correct usage**
```text
ACOBUILDNET "EGLL,EHAM,EDDF,LFPG,EIDW,LIRF,LEMD,LSZH,LOWW" 750 project_data/generated_network.json
```

---

### 4. BlueSky starts and then exits saying it needs packages like `zmq`, `msgpack`, or `PyQt6`
**Fix**
Install the missing packages:

```powershell
python -m pip install pyzmq msgpack PyQt6 PyQt6-WebEngine
```

---

### 5. The interpreter path looks wrong in PyCharm
If you see a path like:

```text
...python.exe\Scripts\python.exe
```

reset the project interpreter. The correct path should normally end at:

```text
bluesky-master\.venv\Scripts\python.exe
```

---

## Troubleshooting

### BlueSky window does not open
- make sure `PyQt6` is installed
- run from the project root
- check the Run console for the next missing dependency

### `import bluesky` is underlined in PyCharm
- ensure you opened the **repo root** (`bluesky-master`)
- do not run random internal files directly
- run `python BlueSky.py` from the root instead

### The benchmark script works but the simulator does not
That usually means the core Python packages are fine, but GUI/runtime BlueSky extras are still missing.

### The UI opens but buttons do nothing
Use stack commands first. The current dashboard/backend link is a known limitation in this build.

### The report-generation script fails with `/mnt/data/...` paths
That script was created in a build environment and uses a hard-coded container path. It is not needed to run the route-planning project.

---

## Suggested Demo Flow

If you want to show the project to someone quickly:

### Minimal demo
```text
PLUGINS LOAD FYPACO
ACOLOAD project_data/fyp_aco_network.json
ACOROUTE ACO EGLL LIRF A320
```

### Better demo
```text
PLUGINS LOAD FYPACO
PLUGINS LOAD FYPACOGUI
ACOLOAD project_data/fyp_aco_network.json
ACOCFG 36 40 1.15 3.0 0.25 200 0.2 42
ACOROUTE ACO EGLL LIRF A320
ACOCREATE FYP100 A320 ACO EGLL LIRF
ACOBENCH project_data/fyp_aco_results.csv A320
```

### Data-generation demo
```text
PLUGINS LOAD FYPACO
ACOBUILDNET "EGLL,EHAM,EDDF,LFPG,EIDW,LIRF,LEMD,LSZH,LOWW" 750 project_data/generated_network.json
ACOLOAD project_data/generated_network.json
ACOLIVE ON
ACOACTYPE B738
ACOCFG 50 60 1.2 2.8 0.30 220 0.25 77
ACOBENCH project_data/generated_results.csv B738
```

---

## Project Scope and Limitations

This project is a **research prototype**.

It does **not** currently provide:
- full operational airline dispatch integration
- full real-time weather ingestion across the whole system
- certified aircraft fuel-burn modelling
- full asynchronous networked colony execution
- a perfectly wired GUI/backend architecture in the current build

What it **does** provide is:
- a coherent distributed ACO architecture
- BlueSky integration
- deterministic baselines for comparison
- simulator-derived network building
- chart and CSV export
- testable and explainable route-planning logic

That is the intended academic value of the project.

---

## Acknowledgements

This project builds on the BlueSky open-source simulator from TU Delft and extends it for COMP1682 final-year research on distributed swarm-based route planning.

The project-specific work focuses on:
- distributed ACO route learning
- fuel-aware cost modelling
- BlueSky simulator integration
- benchmarking and dissertation support

---

If you are using this repository for the dissertation, the safest workflow is:

1. get the benchmark script working first  
2. get BlueSky launching second  
3. use stack commands before relying on the dashboard  
4. generate CSVs and figures for the report  

That order saves a lot of time and a surprising amount of emotional damage.
