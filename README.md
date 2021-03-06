# Go RESTful API Starter Kit (Boilerplate)

[![GoDoc](https://godoc.org/github.com/qiangxue/go-restful-api?status.png)](http://godoc.org/github.com/qiangxue/go-restful-api)
[![Build Status](https://github.com/qiangxue/go-restful-api/workflows/build/badge.svg)](https://github.com/qiangxue/go-restful-api/actions?query=workflow%3Abuild)
[![Code Coverage](https://codecov.io/gh/qiangxue/go-restful-api/branch/master/graph/badge.svg)](https://codecov.io/gh/qiangxue/go-restful-api)
[![Go Report](https://goreportcard.com/badge/github.com/qiangxue/go-restful-api)](https://goreportcard.com/report/github.com/qiangxue/go-restful-api)

This starter kit is designed to get you up and running with a project structure optimized for developing
RESTful API services in Go. It promotes the best practices that follow the [SOLID principles](https://en.wikipedia.org/wiki/SOLID)
and [clean architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html). 
It encourages writing clean and idiomatic Go code. 

The kit provides the following features right out of the box:

* RESTful endpoints in the widely accepted format
* Standard CRUD operations of a database table
* JWT-based authentication
* Environment dependent application configuration management
* Structured logging with contextual information
* Error handling with proper error response generation
* Database migration
* Data validation
* Full test coverage
 
The kit uses the following Go packages which can be easily replaced with your own favorite ones
since their usages are mostly localized and abstracted. 

* Routing framework: [ozzo-routing](https://github.com/go-ozzo/ozzo-routing)
* Database: [ozzo-dbx](https://github.com/go-ozzo/ozzo-dbx)
* Data validation: [ozzo-validation](https://github.com/go-ozzo/ozzo-validation)
* Logging: [zap](https://github.com/uber-go/zap)
* Testing: [testify](https://github.com/stretchr/testify)
* JWT: [jwt-go](https://github.com/dgrijalva/jwt-go)
* Database migration: [golang-migrate](https://github.com/golang-migrate/migrate)

## Getting Started

If this is your first time encountering Go, please follow [the instructions](https://golang.org/doc/install) to
install Go on your computer. The kit requires Go 1.13 or above.

[Docker](https://www.docker.com/get-started) is also needed if you want to try the kit without setting up your
own database server.

After installing Go and Docker, run the following commands to start experiencing this starter kit:

```shell
# download the starter kit
go get github.com/qiangxue/go-restful-api

cd go-restful-api

# start a PostgreSQL database server in a Docker container
make db-start

# seed the database with some test data
make testdata

# run the RESTful API server
make run
```

At this time, you have a RESTful API server running at `http://127.0.0.1:8080`. It provides the following endpoints:

* `GET /healthcheck`: a healthcheck service provided for health checking purpose (needed when implementing a server cluster)
* `POST /v1/login`: authenticates a user and generates a JWT
* `GET /v1/albums`: returns a paginated list of the albums
* `GET /v1/albums/:id`: returns the detailed information of an album
* `POST /v1/albums`: creates a new album
* `PUT /v1/albums/:id`: updates an existing album
* `DELETE /v1/albums/:id`: deletes an album

Try the URL `http://localhost:8080/healthcheck` in a browser, and you should see something like `"OK v1.0.0"` displayed.

If you have `cURL` or some API client tools (e.g. [Postman](https://www.getpostman.com/)), you may try the following 
more complex scenarios:

```shell
# authenticate the user via: POST /v1/login
curl -X POST -H "Content-Type: application/json" -d '{"username": "demo", "password": "pass"}' http://localhost:8080/v1/login
# should return a JWT token like: {"token":"...JWT token here..."}

# with the above JWT token, access the album resources, such as: GET /v1/albums
curl -X GET -H "Authorization: Bearer ...JWT token here..." http://localhost:8080/v1/albums
# should return a list of album records in the JSON format
```

To use the starter kit as a starting point of a real project whose package name is `github.com/abc/xyz`, do a global 
replacement of the string `github.com/qiangxue/go-restful-api` in all of project files with the string `github.com/abc/xyz`.


## Project Layout

The starter kit uses the following project layout:
 
```
.
├── cmd                  main applications of the project
│   └── server           the API server application
├── config               configuration files for different environments
├── internal             private application and library code
│   ├── album            album-related features
│   ├── auth             authentication feature
│   ├── config           configuration library
│   ├── entity           entity definitions and domain logic
│   ├── errors           error types and handling
│   ├── healthcheck      healthcheck feature
│   └── test             helpers for testing purpose
├── migrations           database migrations
├── pkg                  public library code
│   ├── accesslog        access log middleware
│   ├── graceful         graceful shutdown of HTTP server
│   ├── log              structured and context-aware logger
│   └── pagination       paginated list
└── testdata             test data scripts
```

The top level directories `cmd`, `internal`, `pkg` are commonly found in other popular Go projects, as explained in
[Standard Go Project Layout](https://github.com/golang-standards/project-layout).

Within `internal` and `pkg`, packages are structured by features in order to achieve the so-called
[screaming architecture](https://blog.cleancoder.com/uncle-bob/2011/09/30/Screaming-Architecture.html). For example, 
the `album` directory contains the application logic related with the album feature. 

Within each feature package, code are organized in layers (API, service, repository), following the dependency guidelines
as described in the [clean architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html).


## Common Development Tasks

This section describes some common development tasks using this starter kit.

### Implementing a New Feature

Implementing a new feature typically involves the following steps:

1. Develop the service that implements the business logic supporting the feature. Please refer to `internal/album/service.go` as an example.
2. Develop the RESTful API exposing the service about the feature. Please refer to `internal/album/api.go` as an example.
3. Develop the repository that persists the data entities needed by the service. Please refer to `internal/album/repsitory.go` as an example.
4. Wire up the above components together by injecting their dependencies in the main function. Please refer to 
   the `album.RegisterHandlers()` call in `cmd/server/main.go`.

### Updating Database Schema

The starter kit uses [database migration](https://en.wikipedia.org/wiki/Schema_migration) to manage the changes of the 
database schema over the whole project development phase. The following commands are commonly used with regard to database
schema changes:

```shell
# Execute new migrations made by you or other team members.
make migrate

# Create a new database migration.
# In the generated `migrations/*.up.sql` file, write the SQL statements that implement the schema changes.
# In the `*.down.sql` file, write the SQL statements that revert the schema changes.
make migrate-new

# Revert the last database migration.
# This is often used when a migration has some issues and needs to be reverted.
make migrate-down

# Clean up the database and rerun the migrations from the very beginning.
# Note that this command will first erase all data and tables in the database, and then
# run all migrations. 
make migrate-reset
```

### Managing Configurations

The application configuration is represented in `internal/config/config.go`. When the application starts,
it loads the configuration from a configuration file and environment variables. The path to the configuration 
file is specified via the `-config` command line argument, while the environment variables should be named 
with `APP_` prefix. 

The `config` directory contains the configuration files named after different environments. For example,
`config/local.yml` corresponds to local development environment and is used when running the application 
via `make run`.

Do not keep secrets in the configuration files. Provide them via environment variables instead. For example,
you should provide `Config.DSN` using the `APP_DSN` environment variable. 

## Deployment

The application can be run as a docker container. You can use `make build-docker` to build the application 
into a docker image.

The docker container starts with the `cmd/server/entryscript.sh` script which uses the `APP_ENV` environment 
variable to determine which configuration file to use. For example,  if `APP_ENV` is `qa`, the application will
be started with the `config/qa.yml` configuration file.
