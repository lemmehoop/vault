#### Commands
```bash
alembic init alembic  # create alembic dir with base
alembic revision --autogenerate -m "Init message"
alembic upgrade head  # all migrations should be applied
alembic upgrade [+1 / <id>]
alemmbic downgrade [-1 / <id>]
```
#### env.py
https://gitlab.com/lemmehoop/it-costs/-/blob/main/alembic/env.py?ref_type=heads