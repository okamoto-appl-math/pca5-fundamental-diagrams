# PCA5 fundamental-diagram verification

Numerical verification material for *Three-dimensional Fundamental Diagrams
of Five-neighbor Particle Cellular Automata* by Kazuya Okamoto and Daisuke
Takahashi.

The package contains one notebook. It checks the eleven binary relations in
Table 3, the four max-plus relations in Table 4, and one example in which
exchanging the second density destroys single-valuedness.

## Files

| File | Contents |
| --- | --- |
| `pca5_fd_error_check.ipynb` | All calculations and saved reference outputs; code cells only. |
| `requirements.txt` | The numerical dependency used for the reference run. |
| `pca5_table3_errors.csv` | Maximum absolute errors for the eleven binary rules. |
| `maxplus_errors.csv` | Maximum absolute errors and integer residual checks for the four max-plus extensions. |
| `swapped_density_check.csv` | Two configurations with the same densities and different mean flows. |
| `pca5_*_samples.csv` | One file per binary rule, with all 50000 measured samples. |
| `mpca5_*_samples.csv` | One file per max-plus extension, with all 50000 measured samples. |

The sample files contain the measured particle density, second density,
mean flow, formula value, and absolute error. They can also be used to plot
the fundamental diagrams. The max-plus files additionally contain the
initial denominator. `Sample` is numbered within each model, in increasing
lattice-size groups.

## Running the notebook

The reference run used Python 3.12.14 and NumPy 2.3.5. Open the notebook in
Jupyter or Google Colab and run its cells from top to bottom. The calculation
uses NumPy only; no plotting library is needed.

To install the reference NumPy version in a local environment:

```bash
python -m pip install -r requirements.txt
```

Running the notebook writes its CSV results in the current working
directory. Explanations are in this README rather than in notebook cells.

## Settings

All settings are in the first cell.

| Variable | Default | Meaning |
| --- | --- | --- |
| `N_samples` | 50000 | Initial configurations per model. |
| `T_burn` | 1000 | Transient time steps. |
| `T_avg` | 100 | Subsequent time steps used to average both flow and second density. |
| `L_min`, `L_max` | 5, 80 | Inclusive range of periodic lattice sizes. |
| `Denom_max` | 11 | Largest initial denominator for the max-plus extensions. |
| `seed` | 0 | Random seed. |
| `SAVE_SAMPLES` | `True` | Save the individual measured samples as CSV files. |
| `RULES` | All eleven binary rules | Binary models to run. |
| `MAXPLUS_RULES` | All four extensions | Max-plus models to run. |

For a short trial, set `N_samples = 100`. Use the default settings to
reproduce the supplied reference results.

To check binary PCA5-17R alone:

```python
RULES = ['17R']
MAXPLUS_RULES = []
```

To check only mPCA5-17R-1:

```python
RULES = []
MAXPLUS_RULES = ['17R-1']
```

The final cell always runs the two fixed configurations for the
swapped-density example. It uses the same `T_burn` and `T_avg` settings.
