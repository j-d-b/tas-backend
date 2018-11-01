# BCTC Truck Appointment System Backend
The docker-compose deployed API backend service for the [BCTC](http://www.bctc-lb.com/) Truck Appointment System (TAS), composed of [`tas-server`](https://github.com/j-d-b/tas-server/), tas-db, and tas-remind.

* A [MariaDB](https://mariadb.org/) database (`tas-db`)
* A reminder notification spawning **cron** (`tas-remind`)
* The [`tas-server`](https://bitbucket.org/j-d-b/tas-server/)

Using [Docker Compose](https://docs.docker.com/compose/), the TAS and MariaDB servers are started in containers and connected. The container containing the reminder spawning cron is also started.

A `.env` file including all of the following fields is required:
```shell
SERVER_PORT
PRIMARY_SECRET
VERIFY_EMAIL_SECRET
MG_FROM_EMAIL
MG_API_KEY
MG_DOMAIN
MYSQL_HOST # must be the docker-compose service name
MYSQL_USER
MYSQL_PASSWORD
MYSQL_DATABASE
MYSQL_ROOT_PASSWORD
```

## Usage
The `tas-backend` is run with `docker-compose`.

### Development (default)
```
docker-compose up
```

### Production
```
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

## tas-db
The database is initialized with details set in `.env`, specifically:

```shell
MYSQL_USER
MYSQL_PASSWORD
MYSQL_DATABASE
MYSQL_ROOT_PASSWORD
```

These environment variables are used by both the [MariaDB docker image](https://hub.docker.com/_/mariadb/) to build the database connection strong for the `tas-server` to connect to the database.

The database **exposes port 5432** for use by DB management tools and other connections.

### Setup
To set up the database, `docker exec` into the container running `tas-server` and run either `yarn setup` or `yarn setup:dev`.

**Example:**
```
docker exec -it tas-backend_server_1 bash
yarn setup
exit
```

## tas-server
[`tas-server`](https://github.com/j-d-b/tas-server/) is an express web server which exposes a GraphQL API.

`tas-server` has it's own repository and is included as a submodule here.

`tas-server` connects to `tas-db` and is queried by [`tas-app`](https://github.com/j-d-b/tas-app/) (the web UI) and `tas-remind` using the GraphQL API.

`tas-server` related environment variables are:
```shell
SERVER_PORT
PRIMARY_SECRET

MYSQL_USER
MYSQL_PASSWORD
MYSQL_DATABASE

MG_FROM_EMAIL
MG_API_KEY
MG_DOMAIN

PLIVO_AUTH_ID
PLIVO_AUTH_TOKEN
PLIVO_SRC_NUM
```

For more details, see the `tas-server` [README](https://github.com/j-d-b/tas-server/blob/master/README.md).

## tas-remind
`tas-remind` queries the `tas-server` with the `sendApptReminders` mutation **every day at REMIND_HOUR_UTC** using a cron.

`tas-remind` generates it's own signed JWT with no expiration, using the `create-jwt` bash script.

`tas-remind` uses the following environment variables from `.env`:
```shell
PRIMARY_SECRET
SERVER_PORT
REMIND_HOUR_UTC
```
to generate the JWT and connect to the `tas-server`, respectively.

## License
The TAS was built for [BCTC](http://www.bctc-lb.com/) and is licensed under the [GNU General Public License, Version 3](https://www.gnu.org/licenses/gpl-3.0.en.html).

See `LICENSE.md`(https://github.com/j-d-b/tas-backend/blob/master/LICENSE.md) for details.

## TODO
* Switch to MSSQL per client request
* Log reminders sent
* Docker volumes to collect all logs
* Database backups
