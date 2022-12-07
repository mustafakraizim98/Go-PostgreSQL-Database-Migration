# Go-PostgreSQL-Database-Migration
![database_migration_shutterstock_hanss](https://user-images.githubusercontent.com/113289516/206237396-ea3c6266-074c-42fe-9f41-fcff4c8c70e3.jpg)

## Prerequisites
- Go
- Docker Desktop
- PostgreSQL - Docker Image
- ![golang-migrate](https://github.com/golang-migrate/migrate) library

### Optional Prerequisites
- TablePlus
- PostgreSQL - Independent Installation
  - pgAdmin 4
  - SQL Shell (psql)

## Install golang-migrate 
Let’s open this ![CLI documentation](https://github.com/golang-migrate/migrate/tree/master/cmd/migrate) to see how to install it. I’m on a Windows, so I used Scoop.
```
$ scoop install migrate
```

## Create migrations
Let’s create the 1st migration file to initialise our database schema.
Start with ```migrate create```. Then the extension of the file will be ```sql```, and the directory to store it is ```db/migration```.
```
migrate create -ext sql -dir db/migration -seq init_schema
```
If there were no errors, we should have two files available under ```db/migration``` folder:
- 000001_init_schema.down.sql
- 000001_init_schema.up.sql

We use the ```-seq``` flag to generate a sequential version number for the migration file. And finally the name of the migration, which is ```init_schema``` in this case.

## Up/down migration
![7a1ail02bmla1uf51j4s](https://user-images.githubusercontent.com/113289516/206238720-c66ce351-ee3e-4087-8735-8a3868a22642.png)

### ```init_schema.up.sql``` file
```
CREATE TABLE "accounts" (
  "id" serial PRIMARY KEY,
  "owner" varchar NOT NULL,
  "balance" decimal NOT NULL,
  "currency" varchar NOT NULL,
  "created_at" timestamptz NOT NULL DEFAULT 'now()'
);

CREATE TABLE "entries" (
  "id" serial PRIMARY KEY,
  "account_id" bigint NOT NULL,
  "amount" bigint NOT NULL,
  "created_at" timestamptz NOT NULL DEFAULT 'now()'
);

CREATE TABLE "transfers" (
  "id" serial PRIMARY KEY,
  "from_account_id" bigint NOT NULL,
  "to_account_id" bigint NOT NULL,
  "amount" decimal NOT NULL,
  "created_at" timestamptz NOT NULL DEFAULT 'now()'
);

CREATE INDEX ON "accounts" ("owner");

CREATE INDEX ON "entries" ("account_id");

CREATE INDEX ON "transfers" ("from_account_id");

CREATE INDEX ON "transfers" ("to_account_id");

CREATE INDEX ON "transfers" ("from_account_id", "to_account_id");

COMMENT ON COLUMN "entries"."amount" IS 'can be negative or positive';

COMMENT ON COLUMN "transfers"."amount" IS 'must be positive';

ALTER TABLE "entries" ADD FOREIGN KEY ("account_id") REFERENCES "accounts" ("id");

ALTER TABLE "transfers" ADD FOREIGN KEY ("from_account_id") REFERENCES "accounts" ("id");

ALTER TABLE "transfers" ADD FOREIGN KEY ("to_account_id") REFERENCES "accounts" ("id");
```

### ```init_schema.down.sql``` file
```
DROP TABLE IF EXISTS transfers;
DROP TABLE IF EXISTS entries;
DROP TABLE IF EXISTS accounts;
```

## Write Makefile
Let’s add the command that we used to start postgres container to the Makefile as well.
```
postgres:
	docker run --name postgres15.1 -p 15432:5432 -e POSTGRES_USER=root -e POSTGRES_PASSWORD=secret -d postgres:15.1-alpine
createdb:
	docker exec -it postgres15.1 createdb --username=root --owner=root simple_bank

dropdb:
	docker exec -it postgres15.1 psql -U root simple_bank

migrateup:
	migrate -path db/migration -database "postgres://root:secret@localhost:15432/simple_bank?sslmode=disable" -verbose up

migratedown:
	migrate -path db/migration -database "postgres://root:secret@localhost:15432/simple_bank?sslmode=disable" -verbose down

.PHONY: postgres createdb dropdb migrateup migratedown
```

## Run migrations within your Go app
Here is a very simple app running migrations for the above configuration:
```
import (
	"log"

	"github.com/golang-migrate/migrate/v4"
	_ "github.com/golang-migrate/migrate/v4/database/postgres"
	_ "github.com/golang-migrate/migrate/v4/source/file"
)

func main() {
	m, err := migrate.New(
		"file://db/migration",
		"postgres://root:secret@localhost:15432/simple_bank?sslmode=disable")
	if err != nil {
		log.Fatal(err)
	}
	if err := m.Up(); err != nil {
		log.Fatal(err)
	}
}
```

## Sources to follow for more details
- ![PostgreSQL tutorial for beginners](https://github.com/golang-migrate/migrate/blob/master/database/postgres/TUTORIAL.md)
- ![How to write & run database migration in Golang ](https://dev.to/techschoolguru/how-to-write-run-database-migration-in-golang-5h6g)
