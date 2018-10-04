Backend part of Osmose QA tool
==============================

This is the part of osmose [http://osmose.openstreetmap.fr] which analyses OSM
and send results to frontend. This works as following:

  - an .osm.bz2 or .osm.pbf extract is downloaded from a path
  - downloaded file is converted to .osm, with bunzip2 or osmconvert
  - if necessary, an osmosis dump is generated in a local database
  - analyses are run directly on .osm file, or on the database
  - analyses are stored on a local webserver, and a link is sent to the
    frontend so that it can download the results
  - temporary extract files and database are purged

Installation Python
-------------------

Osmose QA backend requires python > 2.6.

Setup system dependencies (Ubuntu Server 14.04)
```
apt install python
```

You can install python dependencies in the system or in a virtualenv.

In the system install the folowing packages:
```
apt install python-dateutil python-imposm-parser python-lockfile python-polib python-poster python-psycopg2 python-shapely python-regex python-requests
```

Alternatively instal python-virtualenv and create a new virtualenv.

Setup system dependencies (Ubuntu Server 14.04)
```
apt install git python-dev python-virtualenv libpq-dev protobuf-compiler libprotobuf-dev
```

Create a python virtualenv, active it and install python dependencies
```
virtualenv --python=python2.7 osmose-backend-venv
source osmose-backend-venv/bin/activate
pip install -r requirements.txt
```

To run tests, additional packages are needed.
```
pip install -r requirements-dev.txt
```


Installation Database
---------------------

Setup system dependencies (Ubuntu Server 14.04)
```
apt install postgresql-9.3 postgresql-contrib-9.3 postgresql-9.3-postgis-2.1
```

As postgres user:
```
createuser osmose
# Set your own password
psql -c "ALTER ROLE osmose WITH PASSWORD '-osmose-';"
createdb -E UTF8 -T template0 -O osmose osmose
# Enable extensions
psql -c "CREATE extension hstore; CREATE extension fuzzystrmatch; CREATE extension unaccent; CREATE extension postgis;" osmose
psql -c "GRANT SELECT,UPDATE,DELETE ON TABLE spatial_ref_sys TO osmose;" osmose
psql -c "GRANT SELECT,UPDATE,DELETE,INSERT ON TABLE geometry_columns TO osmose;" osmose
```


Dependencies
------------

Java JRE for osmosis (Ubuntu Server 14.04):
```
apt install openjdk-7-jre-headless
```

osmosis is installed in osmosis/osmosis-0.44/.
osmconvert is installed in osmconvert/.


Configuration
-------------
A few paths are hardcoded in modules/config.py, and should be adapted or created.

  - dir_osmose is the path of where osmose is installed
  - dir_work is where extracts are stored, and results generated.
  - url_frontend_update is the url used to send results generated by analyses


The local postgresql database should be configured in osmose_config.py:

  - db_base = osmose # database name
  - db_user = osmose # database user
  - db_password = # database password if needed
  - db_host = # database hostname if needed

You may want to include this info in ~/.pgpass to avoid entering the database
password while processing the files.

See https://wiki.postgresql.org/wiki/Pgpass for more info.


Fetching josm translations
--------------------------

JOSM translations are used by some MapCSS plugins, and can be retrieved by bzr:
```
apt install bzr
cd po/josm
bzr checkout --lightweight lp:~openstreetmap/josm/josm_trans
```

Run
---

Look at the osmose_run.py help for options
```
osmose_run.py -h
```


Connection to the "official" frontend at http://osmose.openstreetmap.fr
-----------------------------------------------------------------------

When you have configured the backend for the country you want to add, please
send an email to osmose-contact@openstreetmap.fr. We will then send you the
password to use to connect to the frontend.


Run Tests
---------

Setup a `~/.pgpass` file to allow pgsql to connect to the test database without asking for password:
```
hostname:port:database:username:password
```

Create a test database `osmose_test` and initialize it:
```
createdb -O fred osmose_test
psql -c "CREATE extension hstore; CREATE extension fuzzystrmatch; CREATE extension unaccent; CREATE extension postgis;" osmose_test
psql -c "GRANT SELECT,UPDATE,DELETE ON TABLE spatial_ref_sys TO osmose;" osmose_test
psql -c "GRANT SELECT,UPDATE,DELETE,INSERT ON TABLE geometry_columns TO osmose;" osmose_test
```

Finally run the tests:
```
nosetests analysers/Analyser_Osmosis.py
nosetests analysers/analyser_sax.py
```
