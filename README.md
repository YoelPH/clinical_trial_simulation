# clinical_trial_simulation

notebooks for Monte Carlo clinical-trial simulations with:
- **Stochastic recruitment** over calendar time
- **Delayed event/diagnosis detection** via a customizable month-by-month PMF
- **Interim monitoring** (pause/break rules, minimum follow-up, minimum evaluable N)
- **Early stopping** via a **max-events** rule and/or **stop-after-break** rule
- **Post-simulation summaries and plots** (rates, ribbons, stopping-month histogram)

> Note: this repo is notebook-first and intentionally lightweight. Code is exploratory and not packaged.

## Repository layout

- [clinical_trial_simulations.ipynb](clinical_trial_simulations.ipynb)  
  Main end-to-end simulation notebook: PMF definition/visualization, recruitment model, trial simulator with interim analysis + max-events stopping, Monte Carlo runs, and evaluation/plots.

- [interim_analysis_stopping_rules.ipynb](interim_analysis_stopping_rules.ipynb)  
  Supporting notebook for interim monitoring / stopping-rule exploration (uses the lookup table CSV).

- [interim_monitoring_lookup_table.csv](interim_monitoring_lookup_table.csv)  
  Lookup table used for interim monitoring logic in the interim notebook.

- [README.md](README.md)  
  This file.

## What the main notebook does

### 1) Diagnosis-time / detection-time distribution (PMF)
In [clinical_trial_simulations.ipynb](clinical_trial_simulations.ipynb), a discrete PMF over 24 months is built from:
- a baseline Binomial PMF, then
- custom weights for ultrasound/PET months and other manual tweaks, then
- renormalization to a valid PMF.

This PMF is used to sample a **diagnosis time after treatment** for true-positive patients.

### 2) Recruitment model
Monthly recruitment is sampled with a piecewise “rate by period” model implemented by:
- [`rate_sampler`](clinical_trial_simulations.ipynb) (notebook function)

It samples a per-month recruit count from a Normal distribution, rounds, and clips to period-specific maxima.

### 3) Patient-level simulation
The notebook simulates patient trajectories with:
- recruitment month
- true recessive status (Bernoulli with probability `p0`)
- months since recruitment
- diagnosis time (for positives)
- observed status (becomes 1 once months-since-recruitment reaches diagnosis time)

A basic recruiting-only run is provided by:
- [`full_recruiting_run`](clinical_trial_simulations.ipynb)

### 4) Interim monitoring + early stopping
Core trial simulation is:
- [`trial_interim_simulation`](clinical_trial_simulations.ipynb)

It supports:
- **Interim triggers** at specific recruited-N targets (e.g., `[60]`)
- A minimum follow-up window (e.g., 6 months)
- Optional requirement for a minimum number of evaluable patients at interim (waits if not met)
- Interim decision logic via:
  - [`interim_evaluator`](clinical_trial_simulations.ipynb)
- **Max-events** stopping: stop the trial once observed events reach `max_events`
- A “pause/break” mechanic: if interim triggers a pause, time advances and a final check is applied (“stop after break”).

### 5) Monte Carlo evaluation + plots
The notebook runs Monte Carlo simulations (e.g., 10,000) for different `p0` values and summarizes with:
- [`evaluate_trial_simulation_outputs`](clinical_trial_simulations.ipynb)
- [`print_evaluation`](clinical_trial_simulations.ipynb)

Visualization utilities include:
- [`plot_uncertainty_ribbons`](clinical_trial_simulations.ipynb) (median + quantile bands over time)
- [`plot_stopping_month_hist`](clinical_trial_simulations.ipynb) (distribution of max-events stopping month)

## How to run

### Requirements
These notebooks use standard scientific Python packages:
- `numpy`, `pandas`, `matplotlib`, `scipy`


## Key parameters you’ll likely edit

In [clinical_trial_simulations.ipynb](clinical_trial_simulations.ipynb):

- Trial size / timeline:
  - `total_number_pat` (default used in multiple places: often 120)
  - follow-up duration (often 24 months)

- Recruitment:
  - period means/SDs/clips inside [`rate_sampler`](clinical_trial_simulations.ipynb)

- True event rate:
  - `p0` (used as the Bernoulli probability for “true recessive status”)

- Detection distribution:
  - `PMF` construction (weights, exam months, tail behavior)

- Interim design:
  - `interim=[60]` (or multiple targets)
  - `minimum_follow_up`
  - `minimum_patients_at_interim`

- Max-events:
  - `max_events`

## Output interpretation (high level)

- “Pause at interim” corresponds to the **protocol decision** based on *observed* outcomes at interim.
- “Stop after break” is the final stop decision after the pause window.
- “Stopped due to max events” is independent of interim: it can happen whenever observed events reach `max_events`.
- “Stopping month” histogram summarizes *when* max-events stopping occurs (conditional on it occurring).

## Notes / caveats

- This is exploratory notebook code (not a packaged library).
- There are duplicated helper definitions in the notebook (e.g., `sample_custom_pmf` appears more than once).
- Some values are effectively “global defaults” in cells (e.g., total N=120 is hardcoded in the evaluation helper); update them if you change the trial design.

