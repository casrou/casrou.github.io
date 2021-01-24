---
layout: post
title: Setup PostgreSQL and pgAdmin on Docker
---

In this guide we will setup PostgreSQL and pgAdmin on Docker.

## docker-compose
You'll need to have `docker-compose` installed.

Create the `docker-compose.yml` file:

{% gist dd7f11ce02031fbb8419c60b726fe3e9 %}

Run it with `docker-compose -f docker-compose.yml up`

(The `links` section creates an alias for the hostname of the postgres container, which makes it easier to connect to)

## pgAdmin
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

## Notes
### Reset Postgres / Change environment variables
If we want postgres to rerun the initialization script, we need to delete the volume for the container.

List all volumes:

`docker volume ls`

Find the volume and delete it with: 

`docker volume rm *volume name*`

Quick and dirty solution (worked for me, but be careful):

`docker-compose down`

`docker volume prune`

### Error: 'The system cannot find the file specified
If you forget to start up Docker, you will get the following error when running `docker-compose -f .\docker-compose.yml up`.

On Windows, simply open Docker Desktop and wait for it to run.

```
PS C:\Dev\docker-postgres> docker-compose -f .\docker-compose.yml up
Traceback (most recent call last):
  File "site-packages\docker\api\client.py", line 205, in _retrieve_server_version
  File "site-packages\docker\api\daemon.py", line 181, in version
  File "site-packages\docker\utils\decorators.py", line 46, in inner
  File "site-packages\docker\api\client.py", line 228, in _get
  File "site-packages\requests\sessions.py", line 543, in get
  File "site-packages\requests\sessions.py", line 530, in request
  File "site-packages\requests\sessions.py", line 643, in send
  File "site-packages\requests\adapters.py", line 449, in send
  File "site-packages\urllib3\connectionpool.py", line 677, in urlopen
  File "site-packages\urllib3\connectionpool.py", line 392, in _make_request
  File "http\client.py", line 1244, in request
  File "http\client.py", line 1290, in _send_request
  File "http\client.py", line 1239, in endheaders
  File "http\client.py", line 1026, in _send_output
  File "http\client.py", line 966, in send
  File "site-packages\docker\transport\npipeconn.py", line 32, in connect
  File "site-packages\docker\transport\npipesocket.py", line 23, in wrapped
  File "site-packages\docker\transport\npipesocket.py", line 72, in connect
  File "site-packages\docker\transport\npipesocket.py", line 59, in connect
pywintypes.error: (2, 'CreateFile', 'The system cannot find the file specified.')

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "docker-compose", line 3, in <module>
  File "compose\cli\main.py", line 67, in main
  File "compose\cli\main.py", line 123, in perform_command
  File "compose\cli\command.py", line 69, in project_from_options
  File "compose\cli\command.py", line 132, in get_project
  File "compose\cli\docker_client.py", line 43, in get_client
  File "compose\cli\docker_client.py", line 170, in docker_client
  File "site-packages\docker\api\client.py", line 188, in __init__
  File "site-packages\docker\api\client.py", line 213, in _retrieve_server_version
docker.errors.DockerException: Error while fetching server API version: (2, 'CreateFile', 'The system cannot find the file specified.')
[9620] Failed to execute script docker-compose
```