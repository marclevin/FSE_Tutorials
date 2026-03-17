# Wealth-Wise

Maintain and extend the Wealth-Wise tutorial project without breaking existing tutorial steps, tests, or Flask behavior.

## Project Snapshot

- Language: Python 3.11
- Web framework: Flask
- Data/analysis: pandas, matplotlib
- Database: SQLite (via SQLAlchemy)
- Logging: loguru
- Tests: pytest
- Deploy target: Render (`render.yaml`)
- Formatting: black

## Repo Structure

- `app.py`: Flask routes, dashboard rendering, and API endpoints.
- `helpers/`: Core app logic (`analysis.py`, `transactions.py`, `config.py`).
- `data/`: Database setup and seeding (`database.py`, `seed.py`).
- `templates/` and `static/`: Dashboard UI assets.
- `customer_front/`: Standalone frontend artifacts for tutorial work.
- `tests/`: Unit and API tests.
- `instructions/`: Tutorial notes and assignment content.

## Testing & Formatting Notes

- Tests are located in the `tests/` directory and use pytest.
- Run tests with `pytest -v` from the project root.
- Run formatter with `black .` from the project root.

## Deployment Notes

- Render build command installs requirements and seeds data.
- Production start command:

```bash
gunicorn app:app --bind 0.0.0.0:$PORT
```

- Important env vars include `SECRET_KEY`, `DATABASE_URL`, and `CURRENCY_SYMBOL`.

## Agent Working Rules

- Preserve existing API routes and response shapes unless explicitly asked to change them.
- Treat all monetary values carefully; avoid floating-point assumptions for money-sensitive logic.
- Keep tutorial readability high: favor clear, simple code over dense abstractions.
- Make minimal, scoped changes and avoid unrelated refactors.
- Update or add tests when behavior changes.
- Prefer fixing root causes rather than patching symptoms.

## Pre-PR Checklist for Agents

1. Run tests and ensure they pass.
2. Run formatter (black) to maintain code style consistency.
3. Check that the dashboard route (`/`) still renders.
4. Verify API endpoints still return valid JSON for expected inputs.
5. Confirm database-dependent features still work after seeding.
6. Keep logs and generated artifacts out of commits unless explicitly required.
