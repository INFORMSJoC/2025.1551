# Building and running the experiments

The code was developed on **Windows with Visual Studio**. The retained source contents are unchanged. This source-only archive contains no build projects or executables; create a local build configuration using the requirements below. These instructions were checked against the source but have not been validated by a fresh build or run.

1. [Build](#1-build)
2. [Prepare inputs and run](#2-prepare-inputs-and-run)
3. [Settings and algorithms](#3-settings-and-algorithms)
4. [Paper experiments](#4-paper-experiments)
5. [Read the results](#5-read-the-results)
6. [Generate new instances](#6-generate-new-instances)

## 1. Build

Build the solver and instance generator as **separate executables**. Their entry points are [solver/main.cpp](solver/main.cpp) and [instance-generation/generate_instances.cpp](instance-generation/generate_instances.cpp).

### Dependencies

| Dependency | Program | Version information |
| --- | --- | --- |
| [CPLEX Optimization Studio](https://www.ibm.com/products/ilog-cplex-optimization-studio), including Concert | Solver | Paper: 22.1.1; requires a valid license |
| [LEMON](https://lemon.cs.elte.hu/trac/lemon) | Solver | 1.3.1 |
| [CSFML](https://www.sfml-dev.org/download/csfml/), the C binding of SFML | Solver | Version not recorded; use a compatible C API package |
| [Boost](https://www.boost.org/) | Generator | Paper: 1.80.0; former project: 1.84.0 |

CSFML is required by the solver source set even when drawing is not used. Build/install LEMON separately using its [installation instructions](https://lemon.cs.elte.hu/trac/lemon/wiki/InstallCmake). Use dependency libraries matching your compiler, architecture, and runtime.

### Solver build configuration

For a local Visual Studio build, create an **empty C++ console application**, disable precompiled headers, and add these 24 files from `solver/`:

```text
Abor.cpp            bb.cpp              binary.cpp       bmemory.cpp
CG.cpp              CGCallBack.cpp      CGPrice.cpp      Cplex.cpp
CutF.cpp            DiCutF.cpp          ExF.cpp          FlowF.cpp
main.cpp            memory.cpp          PartitionF.cpp   RepresentativeF.cpp
sbbt.cpp            SetCoverF.cpp       timer.cpp        _g.cpp
_mwcs.cpp           _pcst.cpp           _render.cpp      _tree.cpp
```

Keep all supplied headers available. Do not add a second `main()` from an IDE template. `CG_branch.h` is included by other headers; the empty `Source.cpp` needs no compilation.

| Build option | Original Windows configuration |
| --- | --- |
| Toolset / language | MSVC v143 / C++17 |
| Configuration / architecture | Release / x64 |
| Runtime | `/MD` |
| Character set | Unicode (`UNICODE`, `_UNICODE`) |
| Preprocessor definitions | `IL_STD`, `NDEBUG`, `_CONSOLE` |

Add these include directories:

- CPLEX Studio's `concert/include` and `cplex/include`.
- LEMON's header root and generated configuration-header root, exposing `lemon/...` and `lemon/config.h`.
- CSFML's `include` directory, exposing `SFML/Graphics.h`.
- The repository's `solver/` directory, as needed by your build arrangement.

Link these Windows libraries, alongside the normal Windows SDK libraries:

```text
concert.lib  ilocplex.lib  cplex<VERSION>.lib  lemon.lib
csfml-graphics.lib  csfml-window.lib  csfml-system.lib
```

Replace `<VERSION>` with the installed CPLEX suffix, such as `2211`. The original CPLEX library directories were `concert/lib/x64_windows_msvc14/stat_mda` and `cplex/lib/x64_windows_msvc14/stat_mda`. Use your local LEMON and CSFML library directories.

Choose your own executable name and output directory. The launch examples below assume `bin/solver.exe` and `bin/generate_instances.exe`; replace these paths with your build outputs. Make matching CPLEX and CSFML DLLs available through `PATH` or the executable directory.

### Linux and other compilers

A Linux build requires source adaptations as well as a new build configuration. The main portability points are Windows headers and process/pipe APIs, Microsoft CRT calls such as `sprintf_s` and `fopen_s`, `_mkdir`, and timing functions/constants. Also check paths: `main.cpp` writes `output\...` and opens lowercase `setting.txt`, while the supplied file is `Setting.txt`. The optional external SCIP helper in `_mwcs.cpp` contains `R:` drive paths. No Linux port is included or validated.

## 2. Prepare inputs and run

### Extract the benchmarks

From the repository root, use PowerShell to prepare a fresh working copy:

```powershell
Expand-Archive -LiteralPath '.\Instances\min-max-bsf.zip' -DestinationPath '.\solver'
Rename-Item -LiteralPath '.\solver\min-max-bsf' -NewName 'instances'
New-Item -ItemType Directory -Path '.\solver\output' -Force | Out-Null
```

Skip extraction if the instances are already prepared. These commands assume neither `solver/min-max-bsf/` nor `solver/instances/` exists. The ZIP's 1,368 text files must end up directly in `solver/instances/`, without an extra `min-max-bsf/` level. The driver does not create `output/`.

Each instance starts with `n m k`, followed by `m` rows of `u v w`: vertex count, edge count, required tree count, and integer-weighted edges. Vertex labels range from `0` to `n−1`.

### Launch

Choose the settings in [section 3](#3-settings-and-algorithms), then run from the repository root:

```powershell
$solverExecutable = (Resolve-Path -LiteralPath '.\bin\solver.exe').Path
Set-Location .\solver
& $solverExecutable
```

The **working directory must be `solver/`**, containing `Setting.txt`, `instances/`, and a writable `output/`. Set the same working directory when launching through an IDE. The driver reads its settings from the file; it accepts no command-line algorithm or instance arguments.

## 3. Settings and algorithms

Edit only the nine numeric values in [solver/Setting.txt](solver/Setting.txt). Preserve the labels and blank lines: the parser reads fixed line positions. Changing settings does **not** require recompilation.

### Settings reference

| Line | Setting | Values and meaning |
| --- | --- | --- |
| 2 | Model | `1`–`5`; see the algorithm table below |
| 5 | Vertex Size | `1`–`9`: vertex-count selector; `0`: all sizes |
| 8 | Edge probability | `1`–`5`: densities `0.1, 0.2, 0.3, 0.4, 0.5`; `0`: all densities |
| 11 | Num Trees | `1`–`9`: tree-count selector; `0`: even counts `2,4,6,8,10` |
| 14 | Duplication | `1`–`4`: saved replicate number; `0`: all four |
| 17 | Linear / Integer | `0`: continuous relaxation; `1`: integer formulation, for models `1`–`3` |
| 20 | User cuts for Flow Problem | Model `2` only: `1` enables custom user cuts; `0` disables them. Does not control all built-in CPLEX cuts. |
| 23 | flow mutual exclusion cycles | Model `2` only: `1` includes the additional `AddConsCycleXY()` constraints; `0` omits them |
| 26 | BB Heuristic Time Limit | Integer seconds for the preliminary heuristic, normally `1`; does not set the final solver limit |

| Selector | Vertex count `n` | Tree count `k` |
| --- | --- | --- |
| 1 | 10 | 2 |
| 2 | 15 | 3 |
| 3 | 20 | 4 |
| 4 | 25 | 5 |
| 5 | 30 | 6 |
| 6 | 35 | 7 |
| 7 | 40 | 8 |
| 8 | 45 | 9 |
| 9 | 50 | 10 |

For each selection, the driver calculates `m = int(p*n*(n-1)/2)` and reads `instances/test_<n>_<m>_<k>_<replicate>.txt`. It skips `m < n` or `k > n/2`, so some selections produce no results. To run an odd tree count, select it explicitly; for example, `Num Trees=2` selects three trees.

### Algorithms

All models first compute bounds and run the shared preliminary heuristic.

| Model | Output label | Algorithm |
| --- | --- | --- |
| 1 | `Cut` | Cycle-elimination formulation |
| 2 | `Flow` | Flow formulation, using the two flow settings |
| 3 | `SetCover` | Tree enumeration and set-partition formulation |
| 4 | `CG` | Branch-and-price |
| 5 | `None` | Preprocessing and heuristic only |

- **Models 1–3:** integer runs solve a preliminary continuous relaxation before the integer model. With `Linear / Integer=0`, only the selected continuous formulation is solved.
- **Model 3:** supports at most 32 vertices; use vertex selectors `1`–`5`. Selecting all sizes eventually reaches unsupported instances.
- **Model 4:** always invokes branch-and-price, even if `Linear / Integer=0`. Use `1` for this flag.

The supplied settings select **model 5**, `n=30`, all densities, even tree counts, and all four replicates.

### Single-instance example

To solve `test_10_13_2_1.txt` with the integer cycle-elimination model, set:

| Setting | Value |
| --- | --- |
| Model | 1 |
| Vertex Size | 1 |
| Edge probability | 3 |
| Num Trees | 1 |
| Duplication | 1 |
| Linear / Integer | 1 |
| User cuts for Flow Problem | 0 |
| flow mutual exclusion cycles | 1 |
| BB Heuristic Time Limit | 1 |

This selects 10 vertices, 13 edges, two trees, and replicate 1. For other algorithms, keep the instance selectors and change `Model` using the table above. For continuous relaxations, choose model `1`, `2`, or `3` and set `Linear / Integer=0`. Launch the same executable after each change.

## 4. Paper experiments

For the paper's described 380-instance subset, run four batches for each chosen algorithm, with `Edge probability=0`, `Num Trees=0`, and `Duplication=0`:

| Vertex Size selector | Actual `n` | Selected instances |
| --- | --- | --- |
| 3 | 20 | 80 |
| 5 | 30 | 100 |
| 7 | 40 | 100 |
| 9 | 50 | 100 |

The `n=20, p=0.1` combinations are skipped. All selected files are in the benchmark ZIP. `Vertex Size=0` also includes other sizes; model 3 cannot handle the 40- and 50-vertex batches.

### Limits and reproducibility

| Control | Current source |
| --- | --- |
| CPLEX time limit | 10,800 seconds per solve, defined in [Cplex.h](solver/Cplex.h) |
| CPLEX threads | 1 |
| Relative MIP gap tolerance | 0.00001 |
| Branch-and-price time checks | Use the same time-limit constant |

These controls are not set through `Setting.txt`. The heuristic budget covers only the preliminary heuristic, which may stop earlier. Multiple solves and preprocessing mean the CPLEX limit is not a total instance or batch deadline.

**The paper uses 1,800 seconds for instances with at most 20 vertices and 10,800 seconds for instances with at least 30 vertices; the current driver does not switch limits by size.** Match the paper's experiment configuration before comparing results.

Use the saved ZIP rather than newly generated graphs. Exact table reproduction also requires the corresponding settings, dependency versions, and postprocessing. A complete table/figure regeneration workflow is not included.

## 5. Read the results

Runs write timestamped text files to `solver/output/`, named:

```text
<Model>_<nmin>-<nmax>_<pmin>-<pmax>_<kmin>-<kmax>_<integer|linear>_<YYYY_M_D_H_M_S>.txt
```

Keep the settings and build version with each result: the filename omits the replicate selector, flow flags, and heuristic budget. Identical configurations launched in the same second can overwrite an output file.

Each completed instance produces **20 tab-separated columns without a header**. Import the file into Excel as tab-delimited text and label the columns below. Times are in seconds; floating-point values have two decimal places.

| Column | Field | Meaning |
| --- | --- | --- |
| 1 | Instance | Input path |
| 2 | `n` | Vertices |
| 3 | `m` | Edges |
| 4 | `k` | Required trees |
| 5 | Replicate | Saved instance number |
| 6 | `UB_approx` | Approximation upper bound |
| 7 | `UB_naive` | Naive-construction upper bound |
| 8 | `UB` | Stored incumbent/upper bound; updates depend on the method |
| 9 | `_lp_bound` | Preliminary continuous bound for integer runs of models 1–3 |
| 10 | `_lb_root` | Stored root bound, when populated |
| 11 | `_opt` | Reported integer or continuous objective |
| 12 | `_gap` | Relative gap as a fraction |
| 13 | `elapsedtime_lb` | Preliminary continuous-solve time |
| 14 | `elapsedtime_ub` | Preliminary heuristic time |
| 15 | `elapsedtime_model` | Final formulation time, when populated |
| 16 | `elapsedtime` | Total time measured by the driver |
| 17 | `number_of_trees` | Enumerated tree count for model 3; selected/generated tree pool otherwise |
| 18 | `n_user_cuts` | Custom user-cut counter |
| 19 | `n_lazy_cuts` | Lazy-callback counter; counts invocations in `CutF` |
| 20 | `num_bp_nodes` | Branch-and-price node counter |

Interpretation:

- Model 5 leaves objective, gap, LP, and root-bound fields at zero; these do not establish optimality.
- Model 4 leaves timing columns 13 and 15 at zero; use column 16 for total time.
- Continuous-only runs of models 1–3 report their objective in column 11.
- A rounded gap of `0.00` does not prove optimality. Objective sentinels are `-1` (infeasible), `-2` (unexpected solved status), and `-3` (failed solve).
- Timing uses `clock()`. CPLEX progress output and warnings are suppressed by the wrapper.

[Final-Results.xlsx](Computational%20Results/Final-Results.xlsx) is the supplied results workbook. The solver does not update it or generate the paper's figures automatically.

## 6. Generate new instances

Create a separate console application containing [generate_instances.cpp](instance-generation/generate_instances.cpp), with Boost's header root in the include path. The generator's size, density, and tree-count loops are defined in its source; it does not use `Setting.txt`.

If your local build produces `bin/generate_instances.exe`, launch it from the repository root as follows:

```powershell
$generatorExecutable = (Resolve-Path -LiteralPath '.\bin\generate_instances.exe').Path
Set-Location .\instance-generation
& $generatorExecutable
```

The generator creates `instances/` beneath its working directory. Use a fresh working directory to preserve earlier generated data, and do not run it from `solver/`, where it could overwrite the saved benchmarks.

Graph generation uses a time-based seed. The separate `srand(2025)` call does not fix graph topology, so newly generated files can share benchmark filenames without containing the same graphs.
