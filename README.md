# NS EOS inference from gravitational-wave populations

Tutorial material for a PhD-school lecture on hierarchical Bayesian inference of neutron-star equation-of-state (EOS) information from gravitational-wave populations.

The tutorial follows an LVK-style population-inference workflow, adapted to binary neutron stars and tidal information. The central object is the joint inference of population hyperparameters and a phenomenological neutron-star mass--tidal-deformability relation,

\[
\Lambda = \Lambda_{\rm EOS}(m; \phi_{\rm EOS}).
\]

The notebooks are designed to make the assumptions explicit: the EOS surrogate, the simulated population, the mock parameter-estimation samples, the selection correction, the treatment of EOS model misspecification, and the numerical stability of the Monte Carlo likelihood.

## Important: run the notebooks sequentially

The notebooks are **not independent**.

Each notebook writes files that are used by later notebooks. For the full tutorial, start from notebook 00 and run the notebooks in order, in the same working directory. Opening notebook 03, 04, or 05 directly in a fresh runtime will not work unless the files produced by the previous notebooks already exist.

Recommended order:

```text
00_eos_table_and_polynomial_fit.ipynb
01_population_simulation.ipynb
02_mock_parameter_estimation.ipynb
03_hierarchical_inference_with_selection.ipynb
04_eos_model_misspecification.ipynb
05_soft_eos_constraint_and_mc_stability.ipynb
```

## Recommended Colab workflow

1. Open notebook 00 using the Colab link below.

2. In the first code cell of the Colab notebook, clone the repository and move into it:

```python
!git clone https://github.com/Mik3M4n/ns-eos-population-tutorial.git
%cd ns-eos-population-tutorial
```

3. Install the tutorial dependencies:

```python
!pip install -r requirements.txt
```

4. Run notebook 00.

5. Open notebook 01 from the Colab file browser. The notebooks are inside:

```text
ns-eos-population-tutorial/notebooks/
```

6. Continue sequentially through the notebooks in the same Colab runtime.

## Recommended local workflow

From the repository root:

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
jupyter lab
```

Then execute the notebooks in order.

## Repository structure

```text
.
├── README.md
├── requirements.txt
├── notebooks/
│   ├── 00_eos_table_and_polynomial_fit.ipynb
│   ├── 01_population_simulation.ipynb
│   ├── 02_mock_parameter_estimation.ipynb
│   ├── 03_hierarchical_inference_with_selection.ipynb
│   ├── 04_eos_model_misspecification.ipynb
│   └── 05_soft_eos_constraint_and_mc_stability.ipynb
├── data/
│   ├── raw/
│   └── processed/
├── results/
└── figures/
```

The notebooks create `data/processed/`, `results/`, and `figures/` as needed.

## Notebooks

### 00 — EOS table and polynomial fit

[Open in Colab](https://colab.research.google.com/github/Mik3M4n/ns-eos-population-tutorial/blob/main/notebooks/00_eos_table_and_polynomial_fit.ipynb)

Downloads and cleans the PCP(BSk24) mass--radius--tidal-deformability table, then fits polynomial surrogates for \(\log \Lambda(m)\).

Main outputs:

- `data/raw/bsk24_eos.mr`
- `data/processed/eos_table__<eos_tag>.npz`
- `data/processed/eos_fit__<eos_tag>.npz`
- `figures/eos_fit__<eos_tag>.png`

The chosen EOS fit is used by all later notebooks.

### 01 — Intrinsic population simulation

[Open in Colab](https://colab.research.google.com/github/Mik3M4n/ns-eos-population-tutorial/blob/main/notebooks/01_population_simulation.ipynb)

Simulates an intrinsic binary-neutron-star population using the EOS surrogate from notebook 00. The notebook generates source-frame masses, redshifts, detector-frame quantities, and the true tidal quantities implied by the injected EOS.

Requires notebook 00.

Main outputs:

- `data/processed/intrinsic_population__<run_tag>.npz`
- `figures/intrinsic_population__<run_tag>.png`

### 02 — Mock detections and mock parameter estimation

[Open in Colab](https://colab.research.google.com/github/Mik3M4n/ns-eos-population-tutorial/blob/main/notebooks/02_mock_parameter_estimation.ipynb)

Applies a simple SNR/detection model, selects a mock detected catalog, generates controlled mock posterior samples for each event, and draws injections for the selection correction.

Requires notebooks 00--01.

Main outputs:

- `data/processed/detected_events__<run_tag>.npz`
- `data/processed/mock_pe_samples__<run_tag>.npz`
- `data/processed/injections__<run_tag>.npz`
- diagnostic figures in `figures/`

### 03 — Hierarchical inference with selection effects

[Open in Colab](https://colab.research.google.com/github/Mik3M4n/ns-eos-population-tutorial/blob/main/notebooks/03_hierarchical_inference_with_selection.ipynb)

Performs the main hierarchical inference. The likelihood combines mock event-level PE samples with a population model, an EOS polynomial model, and a selection correction estimated from injections. The notebook uses a GWTC-style rate-marginalized likelihood and includes a Monte Carlo variance diagnostic/cut.

Requires notebooks 00--02.

Main outputs:

- `results/inferencedata__<run_tag>.nc`
- `results/posterior_samples__<run_tag>.npz`
- `results/summary__<run_tag>.csv`
- `results/config__<run_tag>.json`
- posterior, trace, corner, mass-distribution, EOS, and MC-variance figures in `figures/`

Notebook 04 loads the matched inference result from this notebook, so notebook 03 must be run before notebook 04.

### 04 — EOS model misspecification

[Open in Colab](https://colab.research.google.com/github/Mik3M4n/ns-eos-population-tutorial/blob/main/notebooks/04_eos_model_misspecification.ipynb)

Studies EOS model misspecification by comparing the matched inference from notebook 03 with inferences that use different polynomial orders for the EOS model. It is intended to illustrate how under-flexible or over-flexible EOS models can be diagnosed, and when different models are practically indistinguishable for a given dataset.

Requires notebooks 00--03.

Main outputs:

- additional `results/inferencedata__<run_tag>.nc` files for the misspecified EOS models
- `results/eos_misspec_summary__<run_tag>.csv`
- `results/eos_misspec_model_metrics__<run_tag>.csv`
- comparison figures for EOS posterior curves, function-space bias, mass posterior changes, and tidal residuals

### 05 — Soft EOS constraint and Monte Carlo stability

[Open in Colab](https://colab.research.google.com/github/Mik3M4n/ns-eos-population-tutorial/blob/main/notebooks/05_soft_eos_constraint_and_mc_stability.ipynb)

Replaces the strict deterministic EOS constraint with a soft/log-normal tidal constraint. This is a numerical and modeling-interpretation notebook: it shows how a finite-width EOS constraint can regularize Monte Carlo likelihood estimates and how to diagnose the effect of the softening scale. The softening scale can be fixed or inferred with a prior.

Requires notebooks 00--02. Optional comparisons use outputs from notebooks 03--04 when available.

Main outputs:

- `results/inferencedata__<run_tag>.nc`
- `results/posterior_samples__<run_tag>.npz`
- `results/summary__<run_tag>.csv`
- `results/soft_eos_metrics__<run_tag>.csv`
- optional `results/soft_eos_cross_comparison__<run_tag>.csv`
- posterior, trace, corner, EOS, mass-distribution, MC-variance, and soft-constraint diagnostic figures

## Data

Raw external data go in:

```text
data/raw/
```

Processed files generated by the notebooks go in:

```text
data/processed/
```

The PCP(BSk24) EOS table used in this tutorial comes from CompOSE and is downloaded by notebook 00.

## Results and figures

Sampler outputs, summaries, and configuration files are written to:

```text
results/
```

Diagnostic plots are written to:

```text
figures/
```

File names include tags describing the EOS fit, population simulation, mock PE configuration, injection set, and inference settings. These tags are intentionally verbose so that results from different runs do not overwrite each other.

## Tutorial philosophy

1. Prepare an EOS-derived \(\Lambda(m)\) curve.
2. Build a controllable polynomial surrogate for \(\log \Lambda(m)\).
3. Simulate a controlled BNS population.
4. Generate mock event-level posterior samples.
5. Perform hierarchical inference with selection effects.
6. Diagnose EOS model misspecification.
7. Study soft EOS constraints and Monte Carlo stability.

Every approximation should be explicit. The notebooks are intended for teaching, not as production LVK analysis code.
