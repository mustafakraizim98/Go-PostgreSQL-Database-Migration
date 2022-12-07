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
