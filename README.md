# Database Design and Query Explorer (Python & SQL)

A relational database schema and parameterised SQL query explorer built with **Python 3.12** and **SQLite**, modelling university departments, students, modules, and academic enrolments.

## Overview

`sql-query-explorer` demonstrates practical relational database design rather than isolated SQL snippets:
- **Normalised 4-Table Schema:** `departments`, `students`, `modules`, and `enrolments` with strict `FOREIGN KEY` constraints (`ON DELETE RESTRICT` / `ON DELETE CASCADE`) and `CHECK` constraints on credit weights (`10`, `20`, `40`) and marks (`0..100`).
- **Parameterised Multi-Table Joins & Aggregations:** Computes per-module statistics (`COUNT`, `AVG`, `MIN`, `MAX`), student transcripts, and credit-weighted student averages (`SUM(mark * credits) / SUM(credits)` with `HAVING` filters).
- **Index & Query Plan Inspection:** Includes a composite B-tree index (`idx_enrolments_module_mark`) and verifies via `EXPLAIN QUERY PLAN` that range queries over `(module_code, mark)` perform an index search rather than a full table scan.

## Running Tests & Linter

```bash
ruff check .
pytest -v
```
