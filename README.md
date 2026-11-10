# postural_glm_hmm: multi-state GLMs of neural activity with 3D postural predictors

Code I wrote as a full-time research assistant in the Whitlock lab (Prof. Jonathan Whitlock;
[Whitlock-Group on GitHub](https://github.com/Whitlock-Group)) at the Kavli Institute for Systems
Neuroscience, NTNU, Trondheim (2021–2022), as paid work for the lab. It bridges the lab's postural
pipeline (preprocessed 3D pose tracking of freely moving rats) with a state-space model, so that
single-neuron activity from Neuropixels recordings can be explained by time-varying 3D posture and
movement predictors.

**Status:** beta. Tested on Windows 10 with Python 3.9.

## What it does

A **GLM-HMM** extends the generalised linear model with an arbitrary number of hidden states, each
with its own set of GLM weights (Calhoun et al. 2019; Ashwood et al. 2022). Here each neuron's
binned spiking (spike / no spike) is modelled with input-driven Bernoulli observations whose inputs
are postural predictors. The model infers, at every time point, the posterior probability of each
state, so it can reveal epochs in which a neuron's tuning to posture changes. Inference uses
Linderman's [`ssm`](https://github.com/lindermanlab/ssm) with expectation-maximisation.

## Repository layout

| Path | Contents |
|---|---|
| `main.py` | Example run with one allowed parameter set |
| `test_beta_package/` | Loading and preprocessing, inference and output analysis, plots |
| `neurons_selection_notebook.ipynb` | Select neurons without rerunning the postural pipeline |
| `multi_predictor_notebook.ipynb` | Inference with multiple predictors, from the processed pickle |
| `plots_loading_notebook.ipynb` | Load processed results and make the plots |
| `ssm/` | Bundled copy of Linderman's `ssm` (MIT, see `ssm/LICENSE`) |
| `ratemaps/` | Copy of the Whitlock group's [RateMap](https://github.com/Whitlock-Group/RateMap) package (BSD-3-Clause, see `ratemaps/LICENSE`) |
| `spikestats/` | Helpers to read sessions and compute spike statistics; a few input-validation and GLM helpers come from statsmodels and pyglmnet (see `spikestats/THIRD_PARTY_NOTICES.md`) |
| `additional_details.txt` | Model constraints and theory references |

## Data requirements

The input is one `.mat` file per session exported by the Whitlock group's postural pipeline GUI,
holding the spike times, the tracking timestamps and frame rate, and the 3D postural variables.
These recordings, and the results computed from them, are not part of this repository; they belong to
the Whitlock lab. Results are written to `results_folder/analysis_dir/`, which git ignores.

## Installation

1. Install [git](https://git-scm.com/downloads) and conda, then clone the repository:

   ```bash
   git clone https://github.com/teo-fantacci/postural_glm_hmm.git
   cd postural_glm_hmm
   ```

2. Create the environment, either with conda or with pip:

   ```bash
   conda env create -f environment.yml
   conda activate glmhmmbeta
   ```

   ```bash
   pip install -r requirements.txt      # alternative, in a Python 3.9 environment
   ```

3. Install the bundled `ssm`:

   ```bash
   cd ssm
   pip install numpy cython
   pip install -e .
   cd ..
   ```

## Usage

1. Copy the session's `.mat` file to `data/raw_data/`.
2. In `main.py`, set the file name in the `mat_file=` argument of `processing_and_loading`.
3. Run `python main.py`.

`main.py` contains one of the parameter combinations `ssm` allows; the notebooks cover neuron
selection, multiple predictors and plotting from saved results.

## Model parameters

- `num_dimen`: number of possibly dependent dimensions. Must be 1 to use time-dependent predictors
  with input-driven observations.
- `num_categorical_obs`: number of possible outcomes of the neural response with input-driven
  observations; 2 here (Bernoulli).
- `N_iters`: maximum number of iterations, regardless of the log-likelihood.
- `tolerance`: stopping threshold on the log-likelihood.
- `num_indep_neurons`: number of neurons treated as independent (the total likelihood is the
  product over neurons).
- `num_predicotrs`: number of predictors in the inference.
- `max_num_states`: models from 2 states up to this number are tested (`list_states`).
- `observation_type`: observation distribution. Bernoulli is the simplest; input-driven
  observations allow time-dependent predictors.
- `transistion_type`: transition model and its constraints (`recurrent`, `inputdriven`, `sticky`,
  `standard`).
- `optim_method`: optimisation method; `em` for these options.

More detail, including the constraints between options, is in `additional_details.txt`.

## References

- Calhoun, A. J., Pillow, J. W. & Murthy, M. Unsupervised identification of the internal states
  that shape natural behavior. *Nature Neuroscience* 22, 2040–2049 (2019).
- Ashwood, Z. C. et al. Mice alternate between discrete strategies during perceptual
  decision-making. *Nature Neuroscience* 25, 201–212 (2022).
- Linderman, S. et al. `ssm`: Bayesian learning and inference for state space models.
  https://github.com/lindermanlab/ssm

## License

See [`LICENSE.md`](LICENSE.md). The bundled third-party code keeps its own license: `ssm/` (MIT,
Scott Linderman), `ratemaps/` (BSD-3-Clause, Jingyi G. Fuglstad, Whitlock lab), and the helpers
listed in `spikestats/THIRD_PARTY_NOTICES.md` (statsmodels, BSD-3-Clause; pyglmnet, MIT).
