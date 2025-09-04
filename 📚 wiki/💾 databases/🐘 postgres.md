##### Dumping database
```bash
docker exec -i archer-postgres /bin/bash -c \
"PGPASSWORD=password pg_dump --username username archer_db" > dump_all2024-05-17.sql
```
##### Loading database from dump
```bash
cat ~/work/dumps/10.159.3.201_prod/dump_all2024-04-11.sql | docker exec -i \
`docker ps --no-trunc -aqf name=archer_archer-postgres_1` \
psql --username username archer_db
```