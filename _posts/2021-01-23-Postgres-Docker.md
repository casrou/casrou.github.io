---
layout: post
title: Setup PostgreSQL and pgAdmin on Docker
---

In this guide we will setup PostgreSQL and pgAdmin on Docker.

# docker-compose
You'll need to have `docker-compose` installed.

Create the `docker-compose.yml` file:

{% gist dd7f11ce02031fbb8419c60b726fe3e9 %}

Run it with `docker-compose -f docker-compose.yml up`

(The `links` section creates an alias for the hostname of the postgres container, which makes it easier to connect to)

# pgAdmin
Go to `localhost:5050`. Login with `PGADMIN_DEFAULT_EMAIL` and `PGADMIN_DEFAULT_PASSWORD`.

"Add New Server"
- General
    - Name: DemoDB
- Connection
    - Host name/address: pgsql-server
    - Port: 5432
    - Maintenance database: postgres (`POSTGRES_DB`)
    - Username: admin (`POSTGRES_USER`)
    - Password: secret (`POSTGRES_PASSWORD`)
    - Save password: ✓

To create a table, right-click on a database (DemoDB->Databases->postgres), select "Query Tool" and insert the following into the Query Editor:

```
CREATE TABLE accounts (
	user_id serial PRIMARY KEY,
	username VARCHAR ( 50 ) UNIQUE NOT NULL,
	password VARCHAR ( 50 ) NOT NULL,
	email VARCHAR ( 255 ) UNIQUE NOT NULL,
	created_on TIMESTAMP NOT NULL,
	last_login TIMESTAMP 
);
```

We can also run other queries in the Query Editor, such as:
```
INSERT INTO accounts VALUES (DEFAULT, 'casrou', 'secret', 'casper@casrou.com', current_timestamp - interval '1 day', current_timestamp);

SELECT * FROM accounts;
```

# Notes
## Reset Postgres / Change environment variables
If we want postgres to rerun the initialization script, we need to delete the volume for the container.

List all volumes:

`docker volume ls`

Find the volume and delete it with: 

`docker volume rm *volume name*`

Quick and dirty solution (worked for me, but be careful):

`docker-compose down`

`docker volume prune`