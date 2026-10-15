# AttendanceOne

Employee Attendance & Analytics API (FastAPI + MongoDB). All code lives in `app/main.py`.

## Run
```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
# set MONGO_URI / MONGO_DB in environment or .env (real env vars win)
# local docker: docker run -p 27017:27017 mongo:7
uvicorn app.main:app --port 8000
```

## Config
- `MONGO_URI` (default `mongodb://localhost:27017`), `MONGO_DB` (default `attendance_db`)
- Requires MongoDB 6.0+ 
- Indexes are created idempotently at startup via `lifespan`; startup fails fast if index creation fails

## Notes
- Employees keyed by `emp_code`; instants are epoch ms at API boundary, BSON dates in MongoDB (IST rules R1-R5)
- See `DECISIONS.md` / `REVIEW.md` for design and starter defects
