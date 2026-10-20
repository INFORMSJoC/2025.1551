[![INFORMS Journal on Computing Logo](https://INFORMSJoC.github.io/logos/INFORMS_Journal_on_Computing_Header.jpg)](https://pubsonline.informs.org/journal/ijoc)

# Balanced spanning forests of graphs: Hardness and solving methods

This archive is distributed in association with the [INFORMS Journal on Computing](https://pubsonline.informs.org/journal/ijoc) under the [MIT License](LICENSE). Separate notices in the source files and external dependency licenses continue to apply.

The code and data accompany **Balanced spanning forests of graphs: Hardness and solving methods**, by **Morteza Davari, Phablo F. S. Moura, and Hande Yaman**. The algorithms partition an edge-weighted graph into a prescribed number of vertex-disjoint trees spanning all vertices, minimizing the weight of the heaviest tree.

The authors' public repository is [Edge-Weighted-Balanced-Connected-Partitions](https://github.com/md6712/Edge-Weighted-Balanced-Connected-Partitions). An earlier paper version is available as [arXiv:2504.02421](https://arxiv.org/abs/2504.02421).

## Cite

Please cite both the paper and this repository:

- Paper: [10.1287/ijoc.2025.1551](https://doi.org/10.1287/ijoc.2025.1551)
- Code: [10.1287/ijoc.2025.1551.cd](https://doi.org/10.1287/ijoc.2025.1551.cd)

**Code citation**

```bibtex
@misc{davari2026balancedspanningforests,
  author =    {Davari, Morteza and Moura, Phablo F. S. and Yaman, Hande},
  publisher = {INFORMS Journal on Computing},
  title =     {{Balanced spanning forests of graphs: Hardness and solving methods}},
  year =      {2026},
  doi =       {10.1287/ijoc.2025.1551.cd},
  url =       {https://github.com/INFORMSJoC/2025.1551},
  note =      {Available for download at https://github.com/INFORMSJoC/2025.1551}
}
```

**Paper citation** — provisional; add the publication year, volume, issue, and pages when confirmed by the publisher.

```bibtex
@article{davariBalancedSpanningForestsPaper,
  author =  {Davari, Morteza and Moura, Phablo F. S. and Yaman, Hande},
  title =   {{Balanced spanning forests of graphs: Hardness and solving methods}},
  journal = {INFORMS Journal on Computing},
  doi =     {10.1287/ijoc.2025.1551},
  url =     {https://doi.org/10.1287/ijoc.2025.1551}
}
```

## Repository contents

| Location | Contents |
| --- | --- |
| `solver/` | Algorithm implementations, `main.cpp`, and `Setting.txt` |
| `instance-generation/` | Random instance generator, `generate_instances.cpp` |
| [Instances/min-max-bsf.zip](Instances/min-max-bsf.zip) | 1,368 saved benchmark instances |
| [Computational Results/Final-Results.xlsx](Computational%20Results/Final-Results.xlsx) | Results workbook |
| [RUNNING.md](RUNNING.md) | Build setup, settings, examples, and output format |

## Build and run

The retained source contents are unchanged from the original **Windows/Visual Studio implementation** (MSVC v143, C++17). This is a source-only distribution; readers create their own build configuration. Linux requires adaptations for Windows-specific code and has not been validated.

Follow [RUNNING.md](RUNNING.md) to configure dependencies, extract the benchmarks, select an algorithm, and launch the solver. The default settings run preprocessing only. Use the saved benchmark ZIP for reproduction; generating new instances does not recreate the same graphs.

The guide also describes the [paper's instance batches and time limits](RUNNING.md#4-paper-experiments). A complete workflow for regenerating all paper tables and figures is not included.

## Contact and acknowledgments

For questions, contact the authors listed in [AUTHORS](AUTHORS).

We thank [Shunji Tanaka](https://sites.google.com/site/shunjitanaka/home) for making his research software publicly available. The `memory` and `bmemory` routines in `solver/` include his code and adaptations of it; their original copyright notices and redistribution terms are retained.
