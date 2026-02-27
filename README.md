# BDSP Validator

A Python tool for validating solutions to the **Bus Driver Scheduling Problem (BDSP)**.

Given an instance (JSON) and a solution (CSV binary assignment matrix), the validator:
1. Checks **feasibility** — all legs assigned exactly once, employee constraints satisfied
2. Computes the **objective value** with a per-employee breakdown

**Links:**
- [BDSP Benchmark Collection](https://tommanmaz.github.io/bdsp.html) — 284 instances, best known solutions, algorithm results
- [Web Validator](https://tommanmaz.github.io/bdsp_validate.html) — validate solutions directly in the browser (no installation needed)

## Installation

```bash
pip install sortedcontainers
```

Or install all dependencies from `requirements.txt`:

```bash
pip install -r requirements.txt
```

## Usage

### Validate a single solution

```bash
python validator.py -m file -j path/to/instance.json -i path/to/solution.csv
```

### Validate and save objective breakdown

```bash
python validator.py -m file -j path/to/instance.json -i path/to/solution.csv -o breakdown.csv
```

### Batch validate a folder of solutions

Requires an `instances/` directory containing the instance JSON files.

```bash
python validator.py -m folder -i path/to/solutions/ -o report.csv
```

## Input Format

### Instance (JSON)

Instance files are available for download from the [BDSP collection page](https://tommanmaz.github.io/bdsp.html).
Each JSON file contains:
- `legs`: array of bus legs with `tour`, `start`, `end`, `startPos`, `endPos`
- `distances`: position-to-position travel time matrix
- `extra`: per-position `startWork` and `endWork` times (walk/ride time to/from depot)

### Solution (CSV)

A binary matrix with *n* rows (employees) and *l* columns (legs, ordered by start time).
Entry *(i, j) = 1* means leg *j* is assigned to employee *i*.

## Objective Function

The objective for each employee (shift) is:

```
Obj_e = 2 * W' + T + ride + 30 * changes + 180 * splits
```

Where:
- **W'** = max(work_time, 390) — paid working time (minimum 6.5 hours)
- **T** = total shift time (end − start, including breaks)
- **ride** = passive ride time (travel between positions without driving a bus)
- **changes** = number of vehicle (tour) changes
- **splits** = number of split shifts (breaks ≥ 180 minutes)

The total solution cost is the sum of `Obj_e` over all employees.

### Hard Constraints (penalty = 1000 × violation in minutes)

| Constraint | Limit |
|---|---|
| Maximum driving time | 9 h (540 min) |
| Maximum working time | 10 h (600 min) |
| Maximum total shift time | 14 h (840 min) |
| Driving rest (4-hour rule) | Break every 4 h of driving |
| Rest break | ≥ 45 min total break after 6 h of work |

## Output

### Single file mode (`-o`)

CSV with columns: Employee, Feasible, Objective, W', T, ride, changes, splits,
bus_penalty, drive_penalty, rest_penalty, work_time, unpaid, upmax, drive_time, legs

### Folder mode (`-o`)

CSV report with columns: filename, instance, objective, feasible, errors

## Project Structure

```
bdsp-validator/
├── validator.py          # Main validator script (CLI entry point)
├── data/
│   ├── instance.py       # Instance class (loads from JSON)
│   ├── solution.py       # Solution class (loads binary matrix CSV)
│   ├── employee.py       # Employee class with objective evaluation
│   └── busleg.py         # Bus leg data class
└── utils/
    └── logging.py        # Logger configuration
```

## Citation

If you use this validator or the BDSP benchmark collection in your research, please cite:

```bibtex
@article{kletzander2025integrating,
  title={Integrating Column Generation and Large Neighborhood Search for Bus Driver Scheduling with Complex Break Constraints},
  author={Kletzander, Lucas and Mannelli Mazzoli, Tommaso and Musliu, Nysret and Van Hentenryck, Pascal},
  journal={Journal of Artificial Intelligence Research},
  year={2025}
}
```

## License

This project is licensed under the MIT License — see [LICENSE](LICENSE) for details.
