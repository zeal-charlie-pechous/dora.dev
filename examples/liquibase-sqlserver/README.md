# Liquibase + Microsoft SQL Server starter

A minimal Maven project that applies a version-controlled Liquibase changelog to
Microsoft SQL Server. It includes a Docker Compose SQL Server for local use and
an example migration that creates an `example` table.

## Requirements

- Java 11 or newer
- Maven 3.6 or newer
- Docker Compose (only if running the included local SQL Server)

## Start a local SQL Server

Set a strong local development password, then start the container:

```sh
export MSSQL_SA_PASSWORD='Choose_a_Strong_Password1!'
docker compose up -d
```

Create the example database using `sqlcmd` in the SQL Server container:

```sh
docker compose exec sqlserver /opt/mssql-tools18/bin/sqlcmd \
  -S localhost -U sa -P "$MSSQL_SA_PASSWORD" -C \
  -Q "IF DB_ID(N'liquibase_demo') IS NULL CREATE DATABASE liquibase_demo"
```

## Configure and apply the changelog

Set connection details in the environment rather than storing credentials in
the project. The URL below uses the local SQL Server and trusts its development
certificate; use your organization's TLS configuration for other environments.

```sh
export DB_URL='jdbc:sqlserver://localhost:1433;databaseName=liquibase_demo;encrypt=true;trustServerCertificate=true'
export DB_USERNAME=sa
export DB_PASSWORD="$MSSQL_SA_PASSWORD"
mvn liquibase:update
```

Liquibase creates its `DATABASECHANGELOG` and `DATABASECHANGELOGLOCK` tracking
tables and applies the changes in
`src/main/resources/db/changelog/db.changelog-master.yaml`. Re-running the
command only applies changesets that have not already been recorded.

To check the changelog without applying it, run `mvn liquibase:validate`.
Add future schema changes as new changesets; do not edit a changeset after it
has been applied to a shared database.

Stop the local SQL Server with `docker compose down`. Its data is ephemeral and
will be discarded when the container is removed.
