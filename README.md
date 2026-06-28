# NS EOS inference from gravitational-wave populations

Tutorial material for the PhD-school "Thematic school GWsNS-2026: Gravitational Waves from Neutron Stars".
Lecture on hierarchical Bayesian inference of neutron-star equation-of-state (EOS) information from gravitational-wave populations.

The tutorial follows an LVK-style population-inference workflow, adapted to binary neutron stars and tidal information. The central object is the joint inference of population hyperparameters and a phenomenological neutron-star mass--tidal-deformability relation,


Λ = Λ(m; Φ_EOS).

## Important: run the notebooks sequentially

The notebooks are **not independent**.

Each notebook writes files that are used by later notebooks. For the full tutorial, start from notebook 00 and run the notebooks in order, in the same working directory.

The intended order is:

```text
00_eos_table_and_polynomial_fit.ipynb
01_population_simulation.ipynb
02_mock_parameter_estimation.ipynb
03_hierarchical_inference_with_selection.ipynb
04_eos_model_misspecification.ipynb
05_soft_eos_constraint_and_mc_stability.ipynb
```

The recommended way to run the tutorial is locally. Google Colab can be used, requires extra care because the notebooks depend on files produced by previous notebooks.

## Recommended local workflow

From a terminal:

```bash
git clone https://github.com/Mik3M4n/ns-eos-population-tutorial.git
cd ns-eos-population-tutorial

conda create -n ns-eos-pop python=3.11 -y
conda activate ns-eos-pop

pip install -r requirements.txt
jupyter lab
```

Then open the notebooks from the `notebooks/` directory and execute them in order.

All notebooks should be run with the repository root as the working directory. The notebooks write intermediate products to `data/processed/` and figures/results to the appropriate output directories.

## Google Colab workflow

Colab is usable for an interactive run, but it is **not the recommended workflow**. It can be slow, even with a GPU. 

Do **not** run `pip install -r requirements.txt` in Colab. Colab already provides its own notebook environment and many packages. Installing the full local requirements can overwrite Colab's notebook-server dependencies and break the runtime.

### Notebook 00 in Colab

Open notebook 00 from GitHub/Colab. In the first cell, run:

```python
from google.colab import drive
drive.mount('/content/drive')

%cd /content/drive/MyDrive
!git clone https://github.com/Mik3M4n/ns-eos-population-tutorial.git || true
%cd ns-eos-population-tutorial
```

Then execute notebook 00.

This clones the repository into Google Drive so that files written by notebook 00 remain available to notebooks 01--05.

### Notebooks 01 and 02 in Colab

Open the next notebook using:

```text
File -> Open notebook -> GitHub
```

Then choose the notebook from the repository.

At the top of notebooks 01 and 02, run:

```python
from google.colab import drive
drive.mount('/content/drive')

%cd /content/drive/MyDrive
%cd ns-eos-population-tutorial
```

Then execute the notebook.

### Notebooks 03, 04, and 05 in Colab

Open each notebook from GitHub in the same way:

```text
File -> Open notebook -> GitHub
```

At the top of notebooks 03, 04, and 05, run:

```python
from google.colab import drive
drive.mount('/content/drive')

%cd /content/drive/MyDrive
%cd ns-eos-population-tutorial

!pip install -q numpyro corner
```

Then execute the notebook.

The extra install is needed because these notebooks use NumPyro/JAX sampling and corner plots, which may not be available in the default Colab runtime.

## Structure

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
└── figures/
```

## Notebooks

### 00 — EOS table and polynomial fit

[Open in Colab](https://colab.research.google.com/github/Mik3M4n/ns-eos-population-tutorial/blob/main/notebooks/00_eos_table_and_polynomial_fit.ipynb)

Load the PCP(BSk24) mass-radius-tidal-deformability table and fit a polynomial surrogate for \(\log\Lambda\) as a function of neutron-star mass.

This notebook defines the EOS tag, the mass interval, the polynomial order, and the output naming convention used downstream.

Produces files used by notebook 01.

### 01 — Population simulation

[Open in Colab](https://colab.research.google.com/github/Mik3M4n/ns-eos-population-tutorial/blob/main/notebooks/01_population_simulation.ipynb)

Simulate the intrinsic binary-neutron-star population using the EOS surrogate prepared in notebook 00.

The simulation includes the mass population, redshift/distance distribution, source-frame and detector-frame masses, and the corresponding EOS-predicted tidal quantities.

Requires files produced by notebook 00. Produces files used by notebook 02.

### 02 — Mock parameter estimation

[Open in Colab](https://colab.research.google.com/github/Mik3M4n/ns-eos-population-tutorial/blob/main/notebooks/02_mock_parameter_estimation.ipynb)

Generate controlled mock event-level posterior samples for detected systems.

The mock PE samples are generated in the variables used by the hierarchical analysis: detector-frame masses, luminosity distance, and tidal information. The notebook includes rejection/validity checks and diagnostic plots.

Requires files produced by notebooks 00 and 01. Produces files used by notebook 03.

### 03 — Hierarchical inference with selection effects

[Open in Colab](https://colab.research.google.com/github/Mik3M4n/ns-eos-population-tutorial/blob/main/notebooks/03_hierarchical_inference_with_selection.ipynb)

Infer population and EOS parameters using hierarchical Bayesian inference with selection effects.

The notebook reweights event-level PE samples, includes injection-based selection corrections, monitors Monte Carlo stability, and saves the resulting `InferenceData` object for later comparison.

Requires files produced by notebooks 00--02. Produces posterior samples, figures, and saved inference results used by notebook 04.

### 04 — EOS model misspecification

[Open in Colab](https://colab.research.google.com/github/Mik3M4n/ns-eos-population-tutorial/blob/main/notebooks/04_eos_model_misspecification.ipynb)

Study EOS model misspecification by loading the matched inference result from notebook 03 and then running additional inference cases with different polynomial flexibility.

The goal is to compare a matched EOS model against deliberately under-flexible or over-flexible EOS surrogates, and to diagnose whether the data are informative enough to reveal the mismatch. The notebook includes comparison plots and model-comparison diagnostics.

Requires files produced by notebooks 00--03.

### 05 — Soft EOS constraint and Monte Carlo stability

[Open in Colab](https://colab.research.google.com/github/Mik3M4n/ns-eos-population-tutorial/blob/main/notebooks/05_soft_eos_constraint_and_mc_stability.ipynb)

Study a softened EOS consistency constraint as a numerical and modeling diagnostic.

The notebook replaces an exact/delta-like EOS consistency condition with a finite-width likelihood contribution, allowing the width to be fixed or marginalized. It is intended to illustrate numerical stability, Monte Carlo variance, and the interpretation of softened EOS constraints, rather than to solve EOS model misspecification.

Requires files produced by notebooks 00--03. It can also compare its diagnostics to the matched and misspecified inference results from notebooks 03 and 04 when those outputs are available.

## Data

Raw external data go in:

```text
data/raw/
```

Processed files generated by the notebooks go in:

```text
data/processed/
```

The PCP(BSk24) EOS table used in this tutorial comes from CompOSE.

## Tutorial philosophy

1. Prepare the EOS-derived \(\Lambda(m)\) curve.
2. Build a polynomial surrogate on a declared mass interval.
3. Simulate a controlled BNS population.
4. Generate mock event-level posterior samples.
5. Perform hierarchical inference with selection effects.
6. Study EOS model misspecification with different surrogate flexibility.
7. Study softened EOS constraints and Monte Carlo stability.

Every approximation should be explicit. The notebooks are intended for teaching, not as production code.
