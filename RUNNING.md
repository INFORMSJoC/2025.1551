# Building and running the experiments

This guide describes the supplied driver in [solver/main.cpp](solver/main.cpp) and its input file [solver/Setting.txt](solver/Setting.txt). It was prepared by inspecting the source and the original build configuration. No compilation or solver run was performed for this guide.

**Platform note:** The retained C++ source contents are unchanged from the original Windows/Visual Studio implementation. Linux users will need a suitable build configuration and adaptations for Windows-specific code; see [Linux and other compiler ports](#linux-and-other-compiler-ports). A Linux build has not been validated.

The source folders are `solver/` and `instance-generation/`. The generator entry point is [generate_instances.cpp](instance-generation/generate_instances.cpp). Visual Studio solution/project files, generated dependency builds, and executables are intentionally omitted. Readers create their own local build configuration and choose the executable names and locations.

## 1. What to build and what to run

| Purpose | Entry point | Dependencies |
| --- | --- | --- |
| Run the algorithms on saved instances | `solver/main.cpp`, together with the solver sources listed below | CPLEX/Concert, LEMON, CSFML |
| Generate new random instances | `instance-generation/generate_instances.cpp` | Boost |

Build these as **two separate programs**. Do not combine their `main()` functions into one executable.

**Build the solver once, then select algorithms by editing the numeric values in `solver/Setting.txt` before each launch. Changing this settings file does not require recompilation.** `Release` and `Debug` are compiler configurations; they do not select the optimization algorithm.

There is no supplied `Makefile`, `make compile` target, `CMakeLists.txt`, or Visual Studio solution. The implementation was developed with Visual Studio on Windows. Choosing a different build tool is possible, but using another compiler or operating system may also require the source adaptations described below.

## 2. Compiler and dependencies

The original solver configuration used **MSVC v143**, a Windows SDK, **C++17**, and **x64**. It linked libraries compatible with the **`/MD`** runtime. Use compatible compiler, architecture, runtime, and dependency builds throughout. A Release build is appropriate for computational experiments.

| Dependency | Used by | Version information in the repository/manuscript |
| --- | --- | --- |
| IBM ILOG CPLEX Optimization Studio | Solver | The paper reports **22.1.1**. A suitable CPLEX license is needed. |
| LEMON | Solver | The paper and original build configuration identify **1.3.1**. |
| CSFML, the C binding of SFML | Solver | Exact version is not pinned. The source uses the older C API; select a compatible package and architecture. |
| Boost | Instance generator | The paper reports **1.80.0**; the former project referenced **1.84.0**. No package configuration is included now. |

Boost is not needed to build only the solver. CSFML is needed when compiling/linking the original solver source set, including its rendering code, even when graph drawing is not invoked.

### Solver source files

The original solver build used these 24 `.cpp` translation units, all under `solver/`:

```text
Abor.cpp          bb.cpp              binary.cpp
bmemory.cpp       CG.cpp              CGCallBack.cpp
CGPrice.cpp       Cplex.cpp           CutF.cpp
DiCutF.cpp         ExF.cpp             FlowF.cpp
main.cpp          memory.cpp          PartitionF.cpp
RepresentativeF.cpp                  sbbt.cpp
SetCoverF.cpp     timer.cpp           _g.cpp
_mwcs.cpp         _pcst.cpp           _render.cpp
_tree.cpp
```

Keep the supplied `.h` files available to these sources. `CG_branch.h` is included by other headers and does not need a separate compilation step. `Source.cpp` is empty and was not part of the original solver build.

### Include paths, definitions, and libraries

Configure your local solver build with these include directories:

- CPLEX Studio's `concert/include` and `cplex/include`.
- The LEMON header root and the configuration-header root produced by a fresh LEMON build. The compiler must be able to find both `lemon/...` headers and `lemon/config.h`.
- The CSFML `include` directory, which provides `SFML/Graphics.h`.
- The repository's `solver/` directory as needed by your local build arrangement.

The original Windows Release configuration defined `IL_STD`, `NDEBUG`, and `_CONSOLE`, and used the Unicode character set (`UNICODE` and `_UNICODE`). The Unicode setting matters for the Windows process/pipe calls in `_mwcs.cpp`.

Link the following libraries for a Windows solver build:

```text
concert.lib
ilocplex.lib
cplex<VERSION>.lib
lemon.lib
csfml-graphics.lib
csfml-window.lib
csfml-system.lib
```

Replace `<VERSION>` with the installed CPLEX suffix, for example `cplex2211.lib` for CPLEX 22.1.1. The original CPLEX library directories were `concert/lib/x64_windows_msvc14/stat_mda` and `cplex/lib/x64_windows_msvc14/stat_mda`, relative to the CPLEX Studio installation. Locate the matching LEMON and CSFML libraries in your own installations and retain the normal Windows SDK libraries.

Build/install LEMON separately following the [LEMON installation instructions](https://lemon.cs.elte.hu/trac/lemon/wiki/InstallCmake). Generated configuration headers and dependency libraries are not included here. On other platforms, use the corresponding native dependency libraries and linker options rather than Windows `.lib` files.

### Configuring a local Visual Studio build

1. Create an **empty C++ console application** in a local build directory. Add the existing solver sources listed above; do not add another template `main()` function.
2. Select `Release | x64`, C++17, the `/MD` runtime, and the Unicode character set. Disable precompiled headers unless your own build arrangement supplies them; these sources do not include a template `pch.h`.
3. Add the definitions, include directories, library directories, and link libraries listed above.
4. Choose an output filename/location, such as `<repository>/bin/solver.exe`, and build locally when ready. This path is an example, not an executable included in the archive.
5. For launches from the IDE, set the debugging working directory to the repository's **`solver/`** directory. A project created elsewhere will not have the correct working directory by default.

Readers using another IDE, compiler command line, or their own Make/CMake configuration should use the same translation-unit list and dependencies. The repository does not prescribe a local project filename or build command.

### Linux and other compiler ports

The current source is Windows-oriented. A Linux build requires source compatibility work as well as a new build configuration. Inspect at least these areas:

| Area | Examples to adapt/check |
| --- | --- |
| Windows-only headers and APIs | `Windows.h`/`windows.h`, `direct.h`, and `conio.h`; `GetCurrentTime()` in `CG.cpp`; process and named-pipe APIs in `_mwcs.cpp`; `_mkdir()` in the generator. |
| Microsoft CRT calls | `sprintf_s`, `strcat_s`, `fopen_s`, and `fprintf_s`, including Microsoft-specific overloads. Choose replacements with appropriate buffer handling. |
| Paths and filename case | `main.cpp` writes `output\\...` and opens lowercase `setting.txt`, while the supplied file is `Setting.txt`. Linux normally treats those filenames as distinct. |
| Timing | `timer.cpp` uses `clock()` and `CLK_TCK`. Check the available clock constant and timing semantics when comparing results across platforms. |
| Optional external solver integration | `_mwcs.cpp` contains `R:` drive paths and Windows process calls for an external SCIP helper. Inspect whether your chosen route invokes it and configure or port it accordingly. |
| Optional drawing | `_render.cpp` refers to `arial.ttf`; the normal driver's drawing call is commented out. The font is not supplied. |

These are observed portability issues, not a complete or tested Linux port. No source adaptations were made for this guide.

## 3. Prepare the inputs and launch the solver

The saved benchmarks are distributed as [Instances/min-max-bsf.zip](Instances/min-max-bsf.zip). The ZIP contains a `min-max-bsf/` wrapper directory and 1,368 text instance files. The driver expects the text files directly inside `solver/instances/`.

For a fresh checkout, run these **one-time preparation commands** from the repository root in PowerShell:

```powershell
Expand-Archive -LiteralPath '.\Instances\min-max-bsf.zip' -DestinationPath '.\solver'
Rename-Item -LiteralPath '.\solver\min-max-bsf' -NewName 'instances'
New-Item -ItemType Directory -Path '.\solver\output' -Force | Out-Null
```

Use these extraction/rename commands only when `solver/min-max-bsf/` and `solver/instances/` do not already exist. If the dataset is already prepared, skip extraction. With another archive tool, arrange the same final layout; avoid an extra `solver/instances/min-max-bsf/` level. The driver does not create its output directory.

After building your solver and choosing the settings, launch it with `solver/` as the working directory. For example, **if your local build produced `bin/solver.exe`**, run from the repository root:

```powershell
$solverExecutable = (Resolve-Path -LiteralPath '.\bin\solver.exe').Path
Set-Location .\solver
Test-Path .\Setting.txt
Test-Path .\instances
Test-Path .\output
& $solverExecutable
```

Replace the example executable path with the actual output of your build. All three input/directory checks should print `True`. Make the required CPLEX and CSFML runtime DLLs discoverable through the local `PATH` or executable directory, using versions matching the link libraries. No executable or DLL is included in the archive.

The driver accepts no command-line algorithm or instance arguments. It reads `setting.txt` in the working directory; the supplied Windows filename is `Setting.txt`. It constructs instance filenames under `instances/` and writes results under `output/`.

To inspect the most recent result files while still in `solver/`:

```powershell
Get-ChildItem .\output -Filter '*.txt' |
  Sort-Object LastWriteTime -Descending |
  Select-Object -First 5 Name, Length, LastWriteTime
```

Before changing settings for another run, keep a copy of the settings used with the output. The generated output filename does not encode every setting.

## 4. Every setting explained

There are **nine numeric values** in `Setting.txt`, currently on lines **2, 5, 8, 11, 14, 17, 20, 23, and 26**. Preserve the label/value/blank-line layout. The parser consumes fixed lines and does not validate inputs robustly; do not insert comments or remove blank lines inside the file.

| Setting | Values | What it controls |
| --- | --- | --- |
| Model | `1`–`5` | Algorithm choice; see the next section. |
| Vertex Size | `0` or `1`–`9` | Index of the vertex count, not the count itself. `0` visits all nine configured sizes. |
| Edge probability | `0` or `1`–`5` | Index of the density parameter. `0` visits all five densities. |
| Num Trees | `0` or `1`–`9` | Index of the desired number of trees. `0` visits the even counts `2,4,6,8,10`, subject to the driver's filters. |
| Duplication | `0` or `1`–`4` | Saved instance replicate. `0` visits replicates `1,2,3,4`. It does not regenerate a graph. |
| Linear / Integer | `0` or `1` | `0`: continuous relaxation; `1`: integer formulation, for models `1`–`3`. See the branch-and-price exception below. |
| User cuts for Flow Problem | `0` or `1` | Enable (`1`) or disable (`0`) the flow formulation's custom user-cut callback work. Only model `2` uses this flag. It is not a switch for all built-in CPLEX cuts. |
| flow mutual exclusion cycles | `0` or `1` | Include (`1`) or omit (`0`) the additional `AddConsCycleXY()` constraints in the flow formulation. Only model `2` uses this flag. |
| BB Heuristic Time Limit | Integer seconds, normally `1` | Budget for the shared preliminary BB heuristic. This is not the final solver's time limit. |

### Index mappings

| Selector | Vertex count `n` | Number of trees `k` |
| --- | --- | --- |
| `1` | 10 | 2 |
| `2` | 15 | 3 |
| `3` | 20 | 4 |
| `4` | 25 | 5 |
| `5` | 30 | 6 |
| `6` | 35 | 7 |
| `7` | 40 | 8 |
| `8` | 45 | 9 |
| `9` | 50 | 10 |

| Edge-probability selector | Density parameter `p` |
| --- | --- |
| `1` | 0.1 |
| `2` | 0.2 |
| `3` | 0.3 |
| `4` | 0.4 |
| `5` | 0.5 |

The label in the existing settings file lists only some choices. The tables above give the complete arrays in `main.cpp`.

For each selection, the driver computes `m = int(p * n * (n - 1) / 2)` and reads:

```text
instances/test_<n>_<m>_<k>_<replicate>.txt
```

It skips combinations with `m < n` or `k > n/2`. Therefore an apparently valid selection can produce an empty results file if every combination is skipped. For example, `n=10, p=0.1` is skipped.

The `Num Trees=0` behavior comes from advancing the tree-selector index by two. To run an odd number of trees, select it explicitly: for example, `Num Trees=2` means `k=3`.

Each instance begins with `n m k`, followed by `m` rows of `u v w`, using vertex labels `0` through `n-1`. The current parser reads integer values, including edge weights. The dimensions in the file should match those implied by its filename and selected settings.

## 5. Algorithm choices

All choices first prepare the graph, compute bounds/approximation information, run the BB heuristic, prepare incumbent trees, and compute shortest paths.

| Model value | Console/output label | Action after shared preprocessing |
| --- | --- | --- |
| `1` | `Cut` | Cycle-elimination formulation (`CutF`). For integer runs, computes a preliminary continuous relaxation and then solves the warm-started integer model. |
| `2` | `Flow` | Flow formulation (`FlowF`). For integer runs, computes a preliminary continuous relaxation and then solves the warm-started integer model. Uses the two flow flags. |
| `3` | `SetCover` | Enumerates trees and solves the tree-based set-partition formulation (`SetCoverF`). The file label says Set Cover, but coverage constraints are equalities. |
| `4` | `CG` | Calls `CG::Run_BP()` for branch-and-price, including its internal master/pricing work. |
| `5` | `None` | Runs only the shared preprocessing and heuristic; no final formulation is solved. |

**Model 3 has a size restriction:** its tree-generation routine rejects `n > 32`. Among the supplied vertex selectors, use only `1`–`5` (10–30 vertices). `Vertex Size=0` eventually reaches unsupported sizes. Tree enumeration can also be expensive within the supported range.

**Model 4 has no LP-only mode through this settings file:** even with `Linear / Integer=0`, the driver calls `Run_BP()`. Use `1` to describe the intended branch-and-price run clearly. A filename containing `CG` and `linear` does not establish that only a relaxation was solved.

For models `1`–`3`, `Linear / Integer=0` skips the preliminary integer-run relaxation stage and solves the selected continuous formulation directly. The resulting continuous objective appears in output column 11.

## 6. A small single-instance example

After the extraction in section 3, the following example selects `solver/instances/test_10_13_2_1.txt`: 10 vertices, density parameter 0.3, 13 edges, two trees, replicate 1.

For an integer cycle-elimination run, use these contents in `solver/Setting.txt`:

```text
Model  (1: CUT - Cycle Elimination    2: Flow    3: Set Cover - Partition)   4: BP    5- Only Preprocessing
1

Vertex Size  (1: 10   2: 15   3: 20   4: 25  .... )
1

Edge probability (0: all  1: 0.1  2: 0.2  3: 0.3)
3

Num Trees (0: all   1: 2   2: 3   3: 4   4: 5)
1

Duplication (0: all   1: 1   2: 2   3: 3   4: 4)
1

Linear / Integer ( 1: integer       0: linear )
1

User cuts for Flow Problem ( 1: enabled       0: disabled )
0

flow mutual exclusion cycles ( 1: enabled       0: disabled )
1

BB Heuristic Time Limit
1
```

Keep the same instance selectors and change only the indicated numeric settings to compare methods:

| Desired run | Model | Linear / Integer | Flow user cuts | Flow mutual exclusion |
| --- | --- | --- | --- | --- |
| Cycle-elimination integer model | `1` | `1` | Ignored | Ignored |
| Cycle-elimination continuous relaxation | `1` | `0` | Ignored | Ignored |
| Flow integer model, with mutual exclusion | `2` | `1` | `0` | `1` |
| Flow integer model, also with custom user cuts | `2` | `1` | `1` | `1` |
| Flow continuous relaxation, with mutual exclusion | `2` | `0` | `0` | `1` |
| Tree-based set-partition integer model | `3` | `1` | Ignored | Ignored |
| Tree-based set-partition continuous relaxation | `3` | `0` | Ignored | Ignored |
| Branch-and-price | `4` | `1` | Ignored | Ignored |
| Preprocessing/heuristic only | `5` | `1` | Ignored | Ignored |

Launch the same executable after each settings change; there is no need to rebuild.

The checked-in settings currently select model `5`, vertex selector `5` (`n=30`), all densities, even tree counts, and all four replicates. That is a preprocessing batch, not a full optimization run.

## 7. Batches and the paper's instance subset

To run the paper's described 380-instance subset for a selected algorithm, use four separate launches:

| Launch | Vertex Size selector | Actual `n` | Number of selected files |
| --- | --- | --- | --- |
| 1 | `3` | 20 | 80 |
| 2 | `5` | 30 | 100 |
| 3 | `7` | 40 | 100 |
| 4 | `9` | 50 | 100 |

For each launch, set `Edge probability=0`, `Num Trees=0`, and `Duplication=0`. The `n=20, p=0.1` combinations are skipped, giving 380 rather than 400 files. All of these files are included in the benchmark ZIP and become available under `solver/instances/` after extraction.

Choose model `1`, `2`, `4`, or `5` as appropriate. Model `3` cannot run the 40- and 50-vertex groups. Do not use `Vertex Size=0` as a shortcut for the paper subset: that also includes 10, 15, 25, 35, and 45 vertices.

Use the 1,368 saved files from `Instances/min-max-bsf.zip` for this driver. Newly generated graphs can share filenames with the saved benchmarks while having different contents. The authors must confirm which frozen dataset and settings produced each reported table; a matching filename alone is insufficient.

These selections describe the instance subset, not a complete reproduction of the published tables. The results still depend on the solver settings, runtime budgets, software versions, and postprocessing.

## 8. Time limits and controls outside Setting.txt

| Control | Current implementation |
| --- | --- |
| CPLEX time limit | `TIME_LIMIT = 10800` seconds in `solver/Cplex.h`, applied per solve |
| CPLEX threads | `1` |
| CPLEX relative MIP gap tolerance | `0.00001` |
| Branch-and-price outer time checks | Also use `TIME_LIMIT`, checked at intervals |
| BB heuristic budget | The ninth setting, normally `1` second |

The final solver limit, thread count, and MIP gap tolerance are not options in `Setting.txt`. The BB heuristic limit does not change them. The BB heuristic may stop earlier under its internal stopping rules, and the supplied budget is not a strict bound on all preprocessing.

An integer run of models `1`–`3` has a preliminary continuous solve and a final integer solve. Tree generation, preprocessing, and internal branch-and-price solves also take time. Consequently, 10,800 seconds is not a whole-instance or whole-batch deadline.

The paper specifies 1,800 seconds for instances with at most 20 vertices and 10,800 seconds for instances with at least 30 vertices. The current driver does not switch limits by size. Running the four batches above alone does not reproduce that time-budget policy. Resolve the intended experiment configuration before making a replication claim.

## 9. Where results go and how to read them

Each launch creates a text file under `solver/output/`, using this pattern:

```text
<Model>_<nmin>-<nmax>_<pmin>-<pmax>_<kmin>-<kmax>_<integer|linear>_<YYYY_M_D_H_M_S>.txt
```

For the small integer cycle-elimination example, the prefix is:

```text
Cut_10-10_0.3-0.3_2-2_integer_
```

The timestamp comes from the machine running the program. The filename omits the replicate selector, both flow flags, and the heuristic budget. Keep a copy of `Setting.txt` and the software/build version with each output. Avoid launching identical configurations in the same second: the driver initially opens the generated filename in write mode.

The result file contains one row per completed instance, **20 tab-separated columns, and no header**. Floating-point values are written with two decimal places.

| Column | Source field | Meaning |
| --- | --- | --- |
| 1 | `getFilename()` | Instance path |
| 2 | `num_vertices` | Vertex count |
| 3 | `num_edges` | Edge count |
| 4 | `num_trees` | Required tree count |
| 5 | `ii` | Replicate number |
| 6 | `UB_approx` | Upper bound recorded for the approximation routine |
| 7 | `UB_naive` | Upper bound recorded for the naive construction |
| 8 | `UB` | Current stored upper bound/incumbent information; its update depends on the method |
| 9 | `_lp_bound` | Preliminary continuous bound recorded for integer runs of models 1–3; not populated by that preliminary stage for LP-only runs |
| 10 | `_lb_root` | Stored root-bound field; populated only by methods that assign it |
| 11 | `_opt` | Reported objective: integer incumbent or continuous objective, depending on the run |
| 12 | `_gap` | Reported relative gap as a fraction, not a percentage |
| 13 | `elapsedtime_lb` | Preliminary continuous-solve time, seconds |
| 14 | `elapsedtime_ub` | BB heuristic time, seconds |
| 15 | `elapsedtime_model` | Final formulation time, seconds, for the timed model branches |
| 16 | `elapsedtime` | Total instance time measured by the driver, seconds |
| 17 | `number_of_trees` | Enumerated tree count for model 3; selected/generated tree-pool size otherwise |
| 18 | `n_user_cuts` | Stored custom user-cut counter |
| 19 | `n_lazy_cuts` | Stored lazy-callback counter; in `CutF`, this counts callback invocations |
| 20 | `num_bp_nodes` | Stored branch-and-price node counter |

The timing implementation uses `clock()`; use the same timing convention when comparing outputs.

Interpret the fields in the context of the method:

- For model `5`, objective, gap, LP, and root-bound fields remain initialized to zero. These zeros do not indicate an optimal solution. Inspect the heuristic bounds and timing instead.
- For model `4`, columns 13 and 15 remain zero because the driver does not populate those two timers around branch-and-price. Use column 16 for the driver-measured total time.
- For LP-only runs of models `1`–`3`, the continuous objective is in column 11.
- A displayed gap of `0.00` is rounded and is not by itself proof of optimality. The shared CPLEX wrapper also uses negative sentinels: `-1` for infeasible, `-2` for an unexpected solved status, and `-3` for a failed solve. Treat these as status indicators, not objective values.
- Standard CPLEX progress output and warnings are suppressed by the wrapper; the application still prints settings, graphs, and its own progress/results.

To inspect a selected result in PowerShell:

```powershell
Get-Content -LiteralPath '.\output\YOUR_RESULT_FILE.txt' -TotalCount 5
```

In Excel, import the result as a **tab-delimited text file**, with no header row, and label its columns using the table above. The workbook `Computational Results/Final-Results.xlsx` is a supplied results artifact; the solver does not automatically update it or regenerate the paper's figures.

The current archive has no complete table/figure postprocessing workflow. A reproducibility package still needs the exact run-to-table mapping and the plotting/aggregation files used for the manuscript.

## 10. Generating new instances

The separate generator is [instance-generation/generate_instances.cpp](instance-generation/generate_instances.cpp); it does not read `solver/Setting.txt`. Its graph-size/density/tree loops are specified in its source.

Create a separate local console application containing this one `.cpp` file and configure the Boost include root, so that headers such as `boost/graph/adjacency_list.hpp` can be found. The original generator used Visual Studio; the portability notes in section 2 also apply to it. Install Boost separately. The paper reports version 1.80.0 and the former project referenced 1.84.0; record the version used in any new experiments.

If your local build produces `bin/generate_instances.exe`, a Windows launch from the repository root is:

```powershell
$generatorExecutable = (Resolve-Path -LiteralPath '.\bin\generate_instances.exe').Path
Set-Location .\instance-generation
& $generatorExecutable
```

Replace the example executable path with your actual build output. The generator creates an `instances/` directory relative to its working directory and writes its graphs there. With the working directory above, this is `instance-generation/instances/`. Rerunning it in the same directory can replace previously generated files; use a fresh working directory when keeping earlier generated datasets.

The source uses a time-based graph random seed. Its separate `srand(2025)` call does not make the graph topology repeatable. Generating new instances therefore does not recreate the frozen benchmark files exactly.

For reproducing the supplied experiments, use the saved ZIP extracted to `solver/instances/`. Do not run the generator with `solver/` as its working directory, because it can replace those benchmark files.

## 11. Troubleshooting

| Symptom | Check |
| --- | --- |
| A referenced solution, Makefile, or executable is missing | This is a source-only distribution. Create your local build configuration and use its actual executable path. |
| A template `pch.h` is missing or there are duplicate entry points | Disable precompiled headers and include only one program's `main()` in each local target. |
| CPLEX, LEMON, or `SFML/Graphics.h` cannot be included | Check the include locations in section 2, including the generated LEMON configuration header. CSFML supplies the C header. |
| `LNK1104` for a dependency library | Check matching architecture/runtime, installed library directories, and the CPLEX library version suffix. Dependency binaries are not bundled. |
| Windows headers or `*_s` functions are unavailable on Linux | Address the portability issues in section 2; changing compiler flags alone is insufficient. |
| A DLL is missing at launch | Make the matching CPLEX/CSFML runtime directories available through `PATH` or the executable directory. |
| `Error opening file!` or a crash immediately after it | Check the working directory, `Setting.txt`, the constructed instance filename, and the existing writable `output/` directory. Missing inputs are not handled robustly. |
| A run creates an empty result file | Check the `m < n` and `k > n/2` filters; also check whether the run failed before completing its first instance. |
| Only preprocessing results appear | The default model is `5`. Select `1`, `2`, `3`, or `4` for a final algorithm. |
| Model 3 rejects a selected size | Its tree-generation routine rejects more than 32 vertices. |
| A run takes longer than the BB budget | That setting controls only the shared heuristic, not the final solver or the full batch. |
| A `CG` run labeled `linear` still branches | Model 4 always invokes branch-and-price; the flag does not provide an LP-only mode. |

## 12. Source references for this guide

- [solver/main.cpp](solver/main.cpp): setting parsing, selection loops, algorithm dispatch, filenames, and output-column order.
- [solver/Cplex.h](solver/Cplex.h) and [solver/Cplex.cpp](solver/Cplex.cpp): solver limits, parameters, status handling, and output suppression.
- [solver/_g.cpp](solver/_g.cpp): instance filenames, input parsing, initialization, and tree-generation restriction.
- [solver/bb.cpp](solver/bb.cpp): preliminary heuristic and its stopping rules.
- [solver/FlowF.cpp](solver/FlowF.cpp): flow-specific options.
- [solver/CG.cpp](solver/CG.cpp): branch-and-price execution.
- [instance-generation/generate_instances.cpp](instance-generation/generate_instances.cpp): random instance generation.
