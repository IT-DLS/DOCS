The Business Intelligence database is a postgreSql server that lives on the DLS network and must be accessed whilst on the vpn.  The database is fed by daily ETLs scheduled in the remote desktop environment on the LROE user currently. Eventually this will change and we will strike-through this in the documentation.

Currently, all ETLs are ran using python scripts that are leveraging the following libraries:

* Pandas
* Psycopg2
* SQLAlchemy
* Pyodbc

