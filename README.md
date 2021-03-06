sqld
====

SQL over HTTP.
  
**sqld** supports MySQL (`-type mysql`), Postgres (`-type postgres`), and SQLite (`-type sqlite3`) databases.

[![build status](https://travis-ci.org/mmaelzer/sqld.svg?branch=master)](http://travis-ci.org/mmaelzer/sqld)
[![Coverage Status](https://coveralls.io/repos/mmaelzer/sqld/badge.svg?branch=master&service=github)](https://coveralls.io/github/mmaelzer/sqld?branch=master)
[![go report card](https://goreportcard.com/badge/github.com/mmaelzer/sqld)](https://goreportcard.com/report/github.com/mmaelzer/sqld)

Install
-------
```
go get github.com/FloydZ/sqld
```

Usage
-----
```
Usage of 'sqld':
	sqld -u root -db database_name -h localhost:3306 -type mysql

Flags:
  -db string
    	database name
  -dsn string
    	database source name
  -h string
    	database host
  -p string

    	database password
  -port int
    	http port (default 8080)
  -raw
    	allow raw sql queries
  -type string
    	database type (default "mysql")
  -u string
    	database username (default "root")
  -url string
    	url prefix (default "/")
```

Command Line Arguments
----------------------
### -db
The name of the database. Just like `use my_database`. This paremeter is not mandatory. You can specify the database in the URL.

### -dsn
The `dsn` is the data source name for the database, used when making the initial connection to the database. If specified, any host (`h`), user (`u`), or password (`p`) values will be ignored in favor of the `dsn`.

* **MySQL** : For MySQL the format looks like `{user}/{password}@({host})/{database_name}?parseTime=true`. More info on MySQL dsn values: https://github.com/go-sql-driver/mysql#dsn-data-source-name

* **Postgres** : For Postgres the format looks like `postgres://{user}:{password}@{host}/{database_name}?sslmode=disable`. More info on Postgres dsn values: https://godoc.org/github.com/lib/pq#hdr-Connection_String_Parameters

* **SQLite** : For SQLite the format can be a file name `test.db` or `file:test.db?cache=shared&mode=memory` or an in-memory store with `:memory:`.  More info on SQLite dsn values: https://godoc.org/github.com/mattn/go-sqlite3#SQLiteDriver.Open

### -h
The database hostname. For example, running locally, MySQL will generally be `localhost:3306` and for Postgres `localhost:5432`.

### -p
The database password.

### -port 
The HTTP port to serve requests from.

### -type
The database type. Currently supported types are `mysql`, `postgres`, and `sqlite3`.

### -u
The database username.

### -url
The url prefix to use. For example `-url api` will serve requests from `http://hostname:port/api/table` or `-url foo/bar` will serve requests from `http://hostname:port/foo/bar/table`.

Query
-----
Interact with the database via URLs.
```
http://localhost:8080/table_name
```
or 
```
http://localhost:8080/database_name/table_name
```
it doesnt matter if you specified a database in the `-db` command line argument. `sqld` will dynamically create new connections to different databases.
### Filtering
```
http://localhost:8080/table_name?id=10
http://localhost:8080/table_name?name=fred&age=67
```
and again 
```
http://localhost:8080/database_name/table_name?id=10
http://localhost:8080/database_name/table_name?name=fred&age=67
```
sqld is able to create new connections to different databases on the fly.
### Limit
```
http://localhost:8080/table_name?__limit__=20&name=bob
http://localhost:8080/database_name/table_name?__limit__=20&name=bob
```

### Offset
```
http://localhost:8080/table_name?__limit__=20&__offset__=100
http://localhost:8080/database_name/table_name?__limit__=20&__offset__=100
```

### Order By
```
http://localhost:8080/table_name?__order_by__=id+DESC
http://localhost:8080/database_name/table_name?__order_by__=id+DESC
```

Create
------
Create rows in the database via POST requests.
```
POST http://localhost:8080/table_name
POST http://localhost:8080/database_name/table_name
```
### Request
```json
{
  "name": "jim",
  "age": 54
}
```

### Response (201)
```json
{
  "id": 10,
  "name": "jim",
  "age": 54
}
```

Update
------
Update a row in the database with PUT requests.
```
PUT http://localhost:8080/table_name/:id?where=clause
```
### Request
```json
{
  "name": "jill"
}
```

### Response (204)
Empty


Delete
------
Delete a row in the database with DELETE requests.
```
DELETE http://localhost:8080/table_name/:id?where=clause
```

### Response (204)
Empty


Raw SQL Queries
---------------
If you use the `-raw` flag when launching *sqld*, you can `POST` raw SQL queries that will be evaluated and returned. Queries are provided inside of the JSON request body with _either_ `read` or `write` keys and string values that contain the SQL to execute.
  
For example, if we run `sqld -name=my_db -raw` we can perform queries like:
```
POST http://localhost:8080
```
### Request
```json
{
  "read": "SELECT * FROM user WHERE name LIKE %ji%"
}
```
### Response (200)
```json
[
  {
    "id": 66,
    "name": "jill"
  },
  {
    "id": 71,
    "name": "jim"
  }
]
```
### Request
```json
{
  "write": "CREATE TABLE number (id INT NOT NULL AUTO_INCREMENT, num INT NOT NULL, PRIMARY KEY ( id ) )"
}
```
### Response (200)
```json
{
    "last_insert_id": 0,
    "rows_affected": 0
}
```


License
-------
MIT
