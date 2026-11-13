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

## Sampling and update

For every model, choose the lattice size uniformly from 5 through 80.
For each binary sample, choose a probability p uniformly from [0, 1),
then independently assign each site the value 1 with probability p.

For each max-plus sample, choose one common denominator m uniformly from
1 through 11. Each initial site value is independently chosen uniformly
from 0/m, 1/m, ..., m/m. The random seed is reset for every model, giving
the same initial configurations within each family.

The update uses periodic boundary conditions:

```text
q_j     = q(u_(j-2), u_(j-1), u_j, u_(j+1))
u_j_new = u_j + q_j - q_(j+1).
```

For binary models, `flux_table` contains the flux values in Appendix B,
ordered as `1111, 1110, ..., 0001, 0000`. The reflection columns are used
for the four R rules; the canonical columns are used for the other rules.

| Binary rule | Pattern for the second density |
| --- | --- |
| PCA5-11R | `1*0` |
| PCA5-19R | `01*1` |
| PCA5-24R | `0*01` |
| PCA5-17R | `011` |
| PCA5-20 | `1*10` |
| PCA5-28 | `1*0` |
| PCA5-42 | `0*01` |
| PCA5-56 | `01*1` |
| PCA5-57 | Maximal odd-length blocks of 1s |
| PCA5-61 | `01*0` |
| PCA5-93 | `01*0` |

The wildcard `*` accepts either 0 or 1. Blocks and patterns crossing the
periodic boundary are included. For PCA5-57, an all-1 configuration is
one block of length L. Its conserved odd-block count is calculated from
the initial configuration.

## Exact evolution of rational states

For the max-plus extensions, the array stores v = m*u as integers. Every
constant 1 in the flux and local density is replaced by m. Addition,
subtraction, minimum, and maximum preserve this scaling, so this evaluates
the same rational-valued trajectory without rounding during time evolution.
The models remain real-valued max-plus extensions; the integers are their
computational representation for these rational initial conditions.

Particle density, mean flow, and second density are obtained by division
by m and L and temporal averaging. These normalized calculations use
double precision. The four formulas are:

| Extension | F(x, y) |
| --- | --- |
| mPCA5-17R-1 | `max(2*y, 2*x-1)` |
| mPCA5-17R-2 | `max(2*y, 2*x-1)` |
| mPCA5-93-1 | `min(x-y, 1-x)` |
| mPCA5-93-2 | `min(x-y, 1-x)` |

Here x and y are the particle density and second density. Each extension
uses its own flux and local-density function from Section 4.

## Error calculation

For each sample, calculate:

```text
error = abs(Q - F(rho1, rho2)).
```

The printed `E_max` is the largest error over all samples of that model.
Flow and second density are averaged over the same measurement window,
including cases where the latter is periodic.

The reference errors are below 1e-14 for all fifteen tested relations.
The largest binary value is approximately 2.44e-15, and the largest
max-plus value is approximately 1.67e-15. These are finite-sample results
under the specified protocol, rather than proofs for all initial states.

For the max-plus models, the code also checks the relations before
floating-point normalization. Let P be the initial integer particle sum,
S the sum of all integer fluxes in the T-step measurement window,
R the corresponding sum of integer local-density values, and D = m*L*T.
The residual numerator is:

- mPCA5-17R-1 and -2: `S - max(2*R, 2*P*T-D)`.
- mPCA5-93-1 and -2: `S - min(P*T-R, D-P*T)`.

Every tested sample has zero integer residual in the reference run.

## Swapped second density

The final cell uses the time evolution of mPCA5-17R-1 and the second
density of mPCA5-17R-2. On a periodic lattice with L = 6, it checks:

| Initial configuration | Particle density | Swapped second density | Mean flow |
| --- | --- | --- | --- |
| All sites equal to 2/3 | 2/3 | 1/3 | 1/3 |
| `(1,1,0,1,1,0)` | 2/3 | 1/3 | 2/3 |

The first configuration is stationary and the second has period 3.
The two trajectories have the same particle and second densities but
different flows, demonstrating loss of single-valuedness for this pairing.
The array stores the numerators with common denominator 3.

