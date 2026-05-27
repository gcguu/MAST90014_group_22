# MAST90014 Group 22 — Wominjeka Coffee Co Vehicle Routing

A profit-maximising multi-product vehicle routing problem (VRP) for **Wominjeka Coffee Co (WCC)**, a Melbourne-based wholesale coffee distributor that delivers perishable milk and non-perishable beans to a network of cafes from a single depot.

The project formulates, solves, and analyses two Gurobi MIP models (an MTZ baseline and a DFJ lazy-callback variant), compares them to a nearest-neighbour heuristic, and runs sensitivity and perishability experiments on a real Melbourne instance (34 cafes + 1 depot, with OSRM-derived distance and travel-time matrices).

---

## Problem Summary

WCC operates a fleet of refrigerated vans that depart from a single depot (the Peter Hall Building at the University of Melbourne) and must deliver a mix of perishable milk products and non-perishable coffee beans to cafes across inner Melbourne.

The optimiser jointly decides:

1. **Which** cafes are served on a given day
2. **Which** van serves each selected cafe
3. The **route** taken by each van
4. The **quantity** of each product delivered
5. How to **maximise daily profit** while respecting capacity, route-duration, and milk-perishability constraints

Two model formulations are implemented:

- **Model 1** — Profit-maximising multi-product CVRP with perishability (route-duration cap for vans carrying milk). Implemented twice for comparison: once with **MTZ** subtour elimination and once with **DFJ** lazy callbacks. See [docs/math_modelling.md](docs/math_modelling.md).
- **Model 2** — Selective multi-product **VRPTW** extension adding cafe time windows, soft-lateness penalties, departure/arrival times, vehicle-product compatibility, and inventory limits. See [docs/Mathematical model 2.md](docs/Mathematical%20model%202.md).

---

## Repository Layout

```
.
├── data/
│   ├── real/                       OSRM-derived locations + distance/time matrices
│   └── synthetic/                  Research-grounded synthetic cafes, demands, vans, products
├── docs/
│   ├── math_modelling.md           Model 1 (MTZ / DFJ) formulation
│   └── Mathematical model 2.md     Model 2 (VRPTW extension) formulation
├── notebooks/                      All experiments, models, and analysis
├── plots/                          Publication-ready PNG figures
├── results/                        Solver outputs (CSV) and rendered route maps
├── requirements.txt
└── README.md
```

---

## Data

### `data/real/` — Melbourne instance (OSRM-derived)

| File | Description |
|---|---|
| [locations.csv](data/real/locations.csv) | 35 nodes (1 depot + 34 cafes) with `name, latitude, longitude, is_depot` |
| [distance_matrix.csv](data/real/distance_matrix.csv) | 35×35 road distances (km) from the OSRM API |
| [time_matrix.csv](data/real/time_matrix.csv) | 35×35 driving times (minutes) from the OSRM API |

### `data/synthetic/` — Research-grounded demand & fleet parameters

| File | Description |
|---|---|
| [cafes.csv](data/synthetic/cafes.csv) | 34 cafes with `cafe_id, name, lat, lng, size` (small / medium / large) |
| [products.csv](data/synthetic/products.csv) | 7 products: 4 milk types (perishable, 8–10 h shelf-life) + 3 bean varieties |
| [demands.csv](data/synthetic/demands.csv) / [demands_pivot.csv](data/synthetic/demands_pivot.csv) | Daily demand per cafe × product (long and pivot formats) |
| [vans.csv](data/synthetic/vans.csv) | Fleet of 5 refrigerated vans, 500–700 kg capacity, $0.40–$0.50/km fuel |
| [depot.csv](data/synthetic/depot.csv) | Single depot at Peter Hall Building, UniMelb |
| [perishability_params.csv](data/synthetic/perishability_params.csv) | `T_max = 3 h`, loading time 15 min, service time 5 min |
| [cafe_summary.csv](data/synthetic/cafe_summary.csv) | Pre-computed daily weight (kg) and revenue ($) per cafe |

**Headline numbers:** 1,129 L of daily milk demand, 130 kg of bean demand, ~1,291 kg total daily weight, ~$6,464 daily revenue potential. The full assumptions and references are documented in [data/README.md](data/README.md).

---

## Notebooks

All experiments live under [notebooks/](notebooks/). Recommended reading order:

| Notebook | Purpose |
|---|---|
| [data_loading.ipynb](notebooks/data_loading.ipynb) | Loads and merges cafes, demands, vans, products, and perishability parameters; sanity-checks fleet capacity |
| [cafe_distance_matrix.ipynb](notebooks/cafe_distance_matrix.ipynb) | Parses 35 Google Maps links, geocodes cafes, and builds road-based distance/time matrices via OSRM |
| [cafe_location_map.ipynb](notebooks/cafe_location_map.ipynb) | Interactive Folium map of all cafes and the depot |
| [gurobi_mtz.ipynb](notebooks/gurobi_mtz.ipynb) | Model 1 with MTZ subtour elimination (baseline) |
| [gurobi_dfj.ipynb](notebooks/gurobi_dfj.ipynb) | Model 1 with DFJ lazy-callback subtour elimination |
| [gurobi model 2.ipynb](notebooks/gurobi%20model%202.ipynb) | Model 2 — VRPTW extension with time windows and soft lateness |
| [nearest_neighbour.ipynb](notebooks/nearest_neighbour.ipynb) | Greedy nearest-neighbour heuristic benchmark |
| [experiments.ipynb](notebooks/experiments.ipynb) | Solves Gurobi vs. heuristic on four problem sizes; produces [results/experiment_results.csv](results/experiment_results.csv) |
| [sensitivity_analysis.ipynb](notebooks/sensitivity_analysis.ipynb) | 15 scenarios sweeping van capacity, fleet size, T_max, and penalty β |
| [perishability_analysis.ipynb](notebooks/perishability_analysis.ipynb) | Compares solutions with and without milk-perishability constraints |
| [maps.ipynb](notebooks/maps.ipynb) | Side-by-side Folium maps comparing MTZ and DFJ routes |
| [generate_maps_figures_visuals.ipynb](notebooks/generate_maps_figures_visuals.ipynb) | Builds the publication figures in [plots/](plots/) |

---

## Results

### Solver outputs ([results/](results/))

- [experiment_results.csv](results/experiment_results.csv) — Gurobi vs. heuristic on four problem sizes (10/2, 15/3, 34/3, 34/5).
- [sensitivity_results.csv](results/sensitivity_results.csv) — 15 sensitivity scenarios across capacity, fleet size, T_max, and β.
- [perishability_results.csv](results/perishability_results.csv) — With- vs. without-perishability comparison.
- [results/mtz/](results/mtz/) and [results/dfj/](results/dfj/) — `routes.csv`, `deliveries.csv`, and `served_cafes.csv` for each formulation.
- [results/maps/](results/maps/) — Rendered route maps as HTML and PNG (MTZ, DFJ, and side-by-side comparison).

### Figures ([plots/](plots/))

| # | File | What it shows |
|---|---|---|
| 01 | [01_cafe_network_demand_bubble.png](plots/01_cafe_network_demand_bubble.png) | Cafe network sized by daily demand weight |
| 02 | [02_top_cafes_by_demand_weight.png](plots/02_top_cafes_by_demand_weight.png) | Top 12 cafes ranked by demand |
| 03 | [03_product_demand_mix.png](plots/03_product_demand_mix.png) | Product demand mix (milk vs. beans) |
| 04 | [04_osrm_distance_time_distribution.png](plots/04_osrm_distance_time_distribution.png) | OSRM distance/time distribution |
| 05 | [05_gurobi_route_map.png](plots/05_gurobi_route_map.png) | Optimal Gurobi routes on the Melbourne map |
| 06 | [06_van_utilisation_route_duration.png](plots/06_van_utilisation_route_duration.png) | Van utilisation vs. route duration |
| 07 | [07_profit_objective_breakdown.png](plots/07_profit_objective_breakdown.png) | Profit decomposed into margin, transport, fixed cost |
| 08–11 | `08_sensitivity_*.png` | Sensitivity charts for capacity, fleet size, T_max, β |
| — | [perishability_comparison.png](plots/perishability_comparison.png) | With- vs. without-perishability outcomes |
| — | [gurobi_vs_heuristic.png](plots/gurobi_vs_heuristic.png) | Optimiser vs. nearest-neighbour benchmark |
| — | [distance_heatmap.png](plots/distance_heatmap.png), [time_heatmap.png](plots/time_heatmap.png) | OSRM matrix heatmaps |

---

## Setup

Requires **Python 3.10+**, a working **Gurobi** licence (academic licences are free), and an internet connection if you want to regenerate the OSRM matrices.

```bash
pip install -r requirements.txt
jupyter notebook
```

Key dependencies:

- **gurobipy ≥ 11.0** — MIP solver for both model formulations
- **pandas, numpy, scipy** — data handling
- **geopandas, folium, contextily** — geospatial visualisation
- **matplotlib, plotly** — figures
- **ortools, highspy** — optional alternative solvers / benchmarking

See [requirements.txt](requirements.txt) for the full list.

---

## Reproducing the Results

1. Open [notebooks/data_loading.ipynb](notebooks/data_loading.ipynb) and run all cells to verify the input data loads cleanly.
2. Run [notebooks/gurobi_mtz.ipynb](notebooks/gurobi_mtz.ipynb) and [notebooks/gurobi_dfj.ipynb](notebooks/gurobi_dfj.ipynb) to produce the route CSVs in `results/mtz/` and `results/dfj/`.
3. Run [notebooks/experiments.ipynb](notebooks/experiments.ipynb), [notebooks/sensitivity_analysis.ipynb](notebooks/sensitivity_analysis.ipynb), and [notebooks/perishability_analysis.ipynb](notebooks/perishability_analysis.ipynb) to regenerate the CSVs under `results/`.
4. Run [notebooks/generate_maps_figures_visuals.ipynb](notebooks/generate_maps_figures_visuals.ipynb) and [notebooks/maps.ipynb](notebooks/maps.ipynb) to regenerate all figures in `plots/` and route maps in `results/maps/`.

---

## Course Context

MAST90014 — University of Melbourne. Group 22.
