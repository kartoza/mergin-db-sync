# DB Sync Script

This tool takes care of two-way synchronization between Mergin and another database (currently supporting PostGIS).

That means you can:
- insert / update / delete features in PostGIS database - and the changes will get automatically
  pushed to a configured Mergin project
- insert / update / delete features in a GeoPackage in Mergin project - and the changes will get
  automatically pushed to the PostGIS database 

### Installation

1. Install Mergin client: `pip3 install mergin-client`
2. download/clone this git repo

### How to use

Initialization:

1. set up configuration in config.ini  (see config.ini.default for a sample)
2. make your database schema ready (the one marked as 'modified')
3. download mergin project to the local working directory specified
    ```
    $ mergin download username/projectname /tmp/dbsync
    ```
4. run `python3 dbsync.py init-from-db` to create 'base' schema in the database, create GeoPackage in the working dir and push it to Mergin

Optionally, if your Mergin project already contains a GeoPackage file with data,
use `init-from-gpkg` instead of `init-from-db` to initialize DB sync the other way:
based on the existing GeoPackage, it will create and populate tables in the database.

Once initialized:

- run `python3 dbsync.py status' to see if there are any changes on Mergin server or in the database
- run `python3 dbsync.py pull` to fetch data from Mergin and apply them to the database
- run `python3 dbsync.py push` to fetch data from the database and push to Mergin


### Creating a local database (Ubuntu 20.04)

Install PostgreSQL server and PostGIS extension:
```
sudo apt install postgresql postgis
```

Add a user `john` and create a database for the user:
```
sudo -u postgres createuser john
sudo -u postgres psql -c "CREATE DATABASE john OWNER john"
sudo -u postgres psql -d john -c "CREATE EXTENSION postgis;"
``` 

### Creating a working schema

One can use `psql` tool to create a new schema and a single table there:

```
CREATE SCHEMA sync_data;

CREATE TABLE sync_data.points (
  fid serial primary key,
  name text,
  rating integer, geom geometry(Point, 4326)
);
```

### Running Tests

To run automatic tests:

    cd mergin
    export TEST_GEODIFFINFO_EXE=<geodiffinfo>   # path to geodiffinfo executable
    export TEST_DB_CONNINFO=<conninfo>          # connection info for DB
    export TEST_MERGIN_URL=<url>                # testing server
    export TEST_API_USERNAME=<username>
    export TEST_API_PASSWORD=<pwd>
    pytest-3 test/
