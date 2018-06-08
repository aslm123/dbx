# dbx

:fire: Easy database operations for R, including upsert

Supports Postgres, MySQL, SQLite, and more

[![Build Status](https://travis-ci.org/ankane/dbx.svg?branch=master)](https://travis-ci.org/ankane/dbx)

## Installation

Install dbx

```r
install.packages("devtools")
devtools::install_github("ankane/dbx")
```

And follow the instructions for your database library

- [Postgres](#postgres)
- [MySQL](#mysql)
- [SQLite](#sqlite)
- [Others](#others)

### Postgres

Install the R package

```r
install.packages("RPostgreSQL")
```

And use:

```r
library(dbx)

con <- dbxConnect(adapter="postgres", dbname="mydb")
```

You can also pass `user`, `password`, `host`, `port`, and `url`.

### MySQL

Install the R package

```r
install.packages("RMySQL")
```

And use:

```r
library(dbx)

con <- dbxConnect(adapter="mysql", dbname="mydb")
```

You can also pass `user`, `password`, `host`, `port`, and `url`.

### SQLite

Install the R package

```r
install.packages("RSQLite")
```

And use:

```r
library(dbx)

con <- dbxConnect(adapter="sqlite", dbname=":memory:")
```

### Others

Install the appropriate R package and use:

```r
con <- dbxConnect(adapter=odbc::odbc(), database="mydb")
```

## Operations

### Select

Create a data frame of records from a SQL query

```r
records <- dbxSelect(con, "SELECT * FROM forecasts")
```

### Insert

Insert records

```r
table <- "forecasts"
records <- data.frame(temperature=c(32, 25))
inserts <- dbxInsert(con, table, records)
```

Returns a data frame of inserted rows. For Postgres, this includes auto-incremented primary keys.

### Update

Update records

```r
records <- data.frame(id=c(1, 2), temperature=c(16, 13))
dbxUpdate(con, table, records, where_cols=c("id"))
```

Use `where_cols` to specify the columns used for lookup. Other columns are written to the table.

### Upsert

*Atomically* insert if they don’t exist, otherwise update them

```r
records <- data.frame(id=c(2, 3), temperature=c(20, 25))
upserts <- dbxUpsert(con, table, records, where_cols=c("id"))
```

Returns a data frame of upserted rows. For Postgres, this includes auto-incremented primary keys.

Use `where_cols` to specify the columns used for lookup. There must be a unique index on them, or an error will be thrown.

*Only available for PostgreSQL 9.5+, MySQL 5.5+, and SQLite 3.24+*

### Delete

Delete specific records

```r
bad_records <- data.frame(id=c(1, 2))
dbxDelete(con, table, where=bad_records)
```

Delete all records (uses `TRUNCATE` when possible for performance)

```r
dbxDelete(con, table)
```

## Logging [github]

Log all SQL queries with:

```r
options(dbx_verbose=TRUE)
```

## Database Credentials

Environment variables are a good way to store database credentials. This keeps them outside your source control. It’s also how platforms like [Heroku](https://www.heroku.com) store them.

Create an `.Renviron` file in your home directory with:

```
DATABASE_URL=postgres://user:pass@host/dbname
```

And use:

```r
con <- dbxConnect()
```

## Batching [github]

By default, operations are performed in a single statement or transaction. This better for performance and prevents partial writes on failures. However, when working with large data frames on production systems, it can be better to break writes into batches. Use the `batch_size` option to do this.

```r
dbxInsert(con, table, records, batch_size=1000)
dbxUpdate(con, table, records, where_cols, batch_size=1000)
dbxUpsert(con, table, records, where_cols, batch_size=1000)
dbxDelete(con, table, records, where, batch_size=1000)
```

## Origin [github]

Add comments to queries to make it easier to see where time-consuming queries are coming from on production systems.

```r
options(dbx_origin=TRUE)
```

The comment will be appended to queries, like:

```sql
SELECT * FROM users /*script:forecast.R*/
```

## Reference

To close a connection, use: [github]

```r
dbxDisconnect(con)
```

All connections are simply [DBI](https://cran.r-project.org/package=DBI) connections, so you can use them with DBI functions as well.

```r
dbGetInfo(con)
```

## Contributing

Everyone is encouraged to help improve this project. Here are a few ways you can help:

- [Report bugs](https://github.com/ankane/dbx/issues)
- Fix bugs and [submit pull requests](https://github.com/ankane/dbx/pulls)
- Write, clarify, or fix documentation
- Suggest or add new features