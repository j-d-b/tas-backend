# BCTC Truck Appointment System Backend
The docker-compose deployed API backend service for the [BCTC](http://www.bctc-lb.com/) Truck Appointment System (TAS).

## Usage
The `tas-backend` is run with `docker-compose`, which starts four services:
* **api** - web-server exposing a GraphQL API
* **db** - MariaDB database
* **remind** - spawning appointment notifications at a scheduled time
* **backup** - scheduled database dumps

### Development (default)
```
docker-compose up
```

### Production (TODO)
**Note:** This section requires further work
```
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

## Required Environment Variables
A `.env` file including all of the following fields is required:
* `DB_ROOT_PASSWORD`: database root user password
* `DB_USER`: username for the database
* `DB_PASSWORD`: password for the TAS database user (`DB_USER`)
* `DB_NAME`: name of the TAS database
* `REMIND_HOUR_UTC`: two-digit UTC hour (in 24-hour time) to send appointment reminders
* `SERVER_PORT`: port to open for TAS API server
* `SECRET_KEY`: key used to sign API access tokens
* `WEB_APP_URL`: full URL of TAS web app
* `MG_FROM_EMAIL`: Mailgun sender email address (email sending)
* `MG_API_KEY`: Mailgun API key (email sending)
* `MG_DOMAIN`: Mailgun domain (email sending)
* `PLIVO_AUTH_ID`: Plivo auth ID (SMS sending)
* `PLIVO_AUTH_TOKEN`: Plivo auth (SMS sending)
* `PLIVO_SRC_NUM`: Plivo sender mobile number (SMS sending)

## Components
The TAS backend is composed of:
* [`tas-server`](https://github.com/j-d-b/tas-server/): A web server exposing a GraphQL API
* `tas-remind`: A cron which runs once a day and spawns notification sending by querying the API with the `sendApptReminders` mutation
* A MariaDB database

### tas-server
[`tas-server`](https://github.com/j-d-b/tas-server/) is an express web server which exposes a GraphQL API.

`tas-server` has it's own repository and is included as a submodule here.

`tas-server` connects to the database and is queried by [`tas-app`](https://github.com/j-d-b/tas-app/) (the web UI) and `tas-remind` using throught the GraphQL API.

`tas-server` uses the following environment variables:
```
SERVER_PORT
SECRET_KEY
WEB_APP_URL
DB_USER
DB_PASSWORD
DB_NAME
MG_FROM_EMAIL
MG_API_KEY
MG_DOMAIN
PLIVO_AUTH_ID
PLIVO_AUTH_TOKEN
PLIVO_SRC_NUM
```

`tas-server` persists logs to the `tas-server-logs` Docker area volume.

`tas-server` **exposes port 4000**.

For more details, see the `tas-server` [README](https://github.com/j-d-b/tas-server/blob/master/README.md).

### tas-remind
`tas-remind` queries the `tas-server` with the `sendApptReminders` mutation **every day at REMIND_HOUR_UTC**.

`tas-remind` generates its own signed JWT with no expiration for the user `remind@bctc-tas.com` using the `create-jwt` bash script.

`tas-remind` uses the following environment variables from `.env`:
```
SECRET_KEY
SERVER_PORT
REMIND_HOUR_UTC
```
to generate the JWT, connect to the `tas-server`, and schedule send requests respectively.

### Database
The database is initialized with the following details set in `.env`:
```
DB_ROOT_PASSWORD
DB_USER
DB_PASSWORD
DB_NAME
```

The database **exposes port 5432** for use by DB management tools and other connections.

#### Setup
To set up the database, `docker exec` into the container running `tas-server` and run either `yarn setup` or `yarn setup:dev`.

**Example:**
```
docker exec -it tas-backend_api_1 bash
yarn setup
exit
```

### Backup
Performs full database dumps using `mysqldump` every day at `23:00`.

These dumps are gzipped and persisted to the `db-backup` volume in the Docker area.

## Logs
All containers logs are persisted

## License
The TAS was built for [BCTC](http://www.bctc-lb.com/) and is licensed under the [GNU General Public License, Version 3](https://www.gnu.org/licenses/gpl-3.0.en.html).

See `LICENSE.md`(https://github.com/j-d-b/tas-backend/blob/master/LICENSE.md) for details.

## Todo
* Switch to MSSQL per client request
* Elegant logging
