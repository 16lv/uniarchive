# University SQL Explorer (university-sql-explorer)

![CI Status](https://github.com/16lv/university-sql-explorer/actions/workflows/ci.yml/badge.svg)

**University SQL Explorer** ($(System.Collections.Hashtable.newRepo)) is a relational database schema and parameterised SQL query explorer built with **Python 3.12** and **SQLite**, modelling university academic departments, students, modules, and course enrolments.

## Why I Built It

I built University SQL Explorer alongside my Database Systems coursework (`CM2102`) to test relational normalisation, composite primary keys, credit-weighted grade aggregations, and B-tree index query plans (`EXPLAIN QUERY PLAN`) in an automated Python test suite rather than running ad-hoc queries in a GUI client.

## Schema & Relational Constraints

- **`departments`** — Primary key `code` (`COMSC`, `MATHS`), unique department name, and building (`Abacws`).
- **`students`** — Student ID, email (`UNIQUE`), `enrollment_year` (`CHECK(enrollment_year BETWEEN 2018 AND 2030)`), and `FOREIGN KEY (department_code) REFERENCES departments(code) ON DELETE RESTRICT`.
- **`modules`** — Module code (`CM2102`, `CM2104`, `CM2208`, `MA2001`), title, department foreign key, and credit value (`CHECK(credits IN (10, 20, 40))`).
- **`enrolments`** — Composite primary key `(student_id, module_code, academic_year)`, cascading foreign keys, and nullable mark (`CHECK(mark IS NULL OR (mark BETWEEN 0 AND 100))`).

## Key Queries & Index Verification

1. **Module Statistics (`module_statistics`)**: Uses a `LEFT JOIN` from `modules` to `enrolments` so newly created modules with zero enrolled students still appear in departmental summaries alongside `COUNT`, `AVG`, `MIN`, and `MAX` marks.
2. **Credit-Weighted Student Averages (`top_students_by_weighted_average`)**: Computes $\frac{\sum (\text{mark} \times \text{credits})}{\sum \text{credits}}$ across completed modules, multiplying by `1.0` to force floating-point division in SQLite and filtering minimum credit thresholds via `HAVING SUM(m.credits) >= ?`.
3. **Composite Covering Index (`explain_module_mark_lookup`)**: Creates `idx_enrolments_module_mark ON enrolments(module_code, mark)` and verifies via `EXPLAIN QUERY PLAN` that filtering by `module_code = ? AND mark >= ?` uses `SEARCH enrolments USING COVERING INDEX` instead of a full table scan.

## Usage Example

```python
from src.sql_explorer.database import QueryExplorer

with QueryExplorer(":memory:") as db:
    db.seed_sample_data()
    for mod in db.module_statistics():
        print(f"{mod['module_code']} ({mod['title']}): avg={mod['average_mark']}")

    for student in db.top_students_by_weighted_average(min_credits=40):
        print(f"{student['full_name']}: {student['weighted_average']}% ({student['completed_credits']} credits)")

    print(db.explain_module_mark_lookup("CM2102", 70))
```

## Running Tests & Linter

```bash
pip install -e ".[dev]"
ruff check .
pytest -v
```

## Known Limitations & Future Improvements

- Marks are currently stored as a single integer per enrolment; adding an `assessment_components` table (coursework vs written examination weights) would allow computing module marks from individual assessment submissions.
