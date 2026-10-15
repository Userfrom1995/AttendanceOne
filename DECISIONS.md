# DECISIONS.md

1. **Indexes.** On `employees`: `{ emp_code: 1 }` (unique) ensures identity; `{ department: 1, joined_on: 1 }` and `{ joined_on: 1 }` back trends and summary scans. On `attendance_logs`: `{ emp_code: 1, date: 1 }` (unique) enforces daily uniqueness; `{ date: -1, emp_code: 1 }` and `{ status: 1, date: -1, emp_code: 1 }` eliminate in-memory sorting; `{ emp_code: 1, punch_in: -1 }` accelerates punch-outs; `{ date: 1, late_minutes: 1, emp_code: 1 }` enables covered leaderboard scans. `history` is omitted as an unqueried audit log.

2. **Punch-in race.** Both concurrent requests calculate the date and call `insert_one()`. The unique index on `(emp_code, date)` serializes them: one succeeds (201), the other triggers `DuplicateKeyError` caught to return 409 Conflict.

3. **Ties.** Both tied employees are returned, even if results exceed `limit`. Competition ranking (`$setWindowFields` with `$rank`: 1, 2, 2, 4) followed by `{ rank: { $lte: limit } }` preserves tied cutoff rows.

4. **Headcount.** The pipeline starts from `employees` matching `joined_on <= month_end`, left-joining `attendance_logs` via `$lookup`. Counting employees (`$sum: 1`) guarantees inclusion of active staff with zero logs.

5. **100x data.** We would shard `attendance_logs` by `{ emp_code: "hashed", date: 1 }` and pre-aggregate daily department stats via Change Streams rather than scanning raw collections.
