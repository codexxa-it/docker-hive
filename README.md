[![Gitter chat](https://badges.gitter.im/gitterHQ/gitter.png)](https://gitter.im/big-data-europe/Lobby)


# oracle
docker run -it --name oracle -p 1521:1521 store/oracle/database-enterprise:12.2.0.1
.\sqlplus.exe sys/Oradoc_db1@127.0.0.1:1521/ORCLCDB.localdomain as sysadm
.\sqlplus.exe version730/version730@127.0.0.1:1521/ORCLCDB.localdomain


# docker-hive
Use host.docker.internal
docker network create cloudera

docker run --network=cloudera --name nifi --rm -v c:/Users/kabir/nifi/:/opt/nifi/nifi-current/conf/templates -it -p 9090:9090 -e NIFI_WEB_HTTP_PORT='9090' apache/nifi


This is a docker container for Apache Hive 3.1.2. It is based on https://github.com/big-data-europe/docker-hadoop so check there for Hadoop configurations.
This deploys Hive and starts a hiveserver2 on port 10000.
Metastore is running with a connection to postgresql database.
The hive configuration is performed with HIVE_SITE_CONF_ variables (see hadoop-hive.env for an example).

To run Hive with postgresql metastore:
```
    docker-compose up -d
```

To deploy in Docker Swarm:
```
    docker stack deploy -c docker-compose.yml hive
```

To run a PrestoDB 0.181 with Hive connector:

```
  docker-compose up -d presto-coordinator
```

This deploys a Presto server listens on port `8080`

## Testing
Load data into Hive:
```
  $ docker-compose exec hive-server bash
  # /opt/hive/bin/beeline -u jdbc:hive2://localhost:10000
  > CREATE TABLE pokes (foo INT, bar STRING);
  > LOAD DATA LOCAL INPATH '/opt/hive/examples/files/kv1.txt' OVERWRITE INTO TABLE pokes;
```

Then query it from PrestoDB. You can get [presto.jar](https://prestosql.io/docs/current/installation/cli.html) from PrestoDB website:
```
  $ wget https://repo1.maven.org/maven2/io/prestosql/presto-cli/308/presto-cli-308-executable.jar
  $ mv presto-cli-308-executable.jar presto.jar
  $ chmod +x presto.jar
  $ ./presto.jar --server localhost:8080 --catalog hive --schema default
  presto> select * from pokes;
```

## Contributors
* Ivan Ermilov [@earthquakesan](https://github.com/earthquakesan) (maintainer)
* Yiannis Mouchakis [@gmouchakis](https://github.com/gmouchakis)
* Ke Zhu [@shawnzhu](https://github.com/shawnzhu)


## Local Oracle setup

Starting an Oracle Database Server instance
Starting an Oracle database server instance is as simple as executing

$ docker run -it --name oracle -p 1521:1521 store/oracle/database-enterprise:12.2.0.1
./sqlplus sys/Oradoc_db1@127.0.0.1:1521/ORCLCDB.localdomain as sysdba

where <Oracle-DB> is the name of the container and 12.2.0.1 is the Docker image tag.

The database server is ready to use when the STATUS field shows (healthy) in the output of docker ps.

Connecting to the Database Server Container
The default password to connect to the database with sys user is Oradoc_db1.

Connecting from within the container
The database server can be connected to by executing SQL*Plus,

$ docker exec -it oracle bash -c "source /home/oracle/.bashrc; sqlplus /nolog"

Connecting from outside the container
The database server exposes port 1521 for Oracle client connections over SQLNet protocol and port 5500 for Oracle XML DB. SQLPlus or any JDBC client can be used to connect to the database server from outside the container.

To connect from outside the container start the container with -P or -p option as,

$ docker run -it --name oracle -P store/oracle/database-enterprise:12.2.0.1

option -P indicates the ports are allocated by Docker. The mapped port can be discovered by executing

$ docker port <Oracle-DB> 1521/tcp -> 0.0.0.0:<mapped host port>

Using this <mapped host port> and <ip-address of host> create tnsnames.ora in the directory pointed to by environment variable TNS_ADMIN.

ORCLCDB=(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=<ip-address of host>)(PORT=<mapped host port>))
    (CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=ORCLCDB.localdomain)))
ORCLPDB1=(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=<ip-address> of host)(PORT=<mapped host port>))
    (CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=ORCLPDB1.localdomain)))
To connect from outside the container using SQL*Plus,

$ sqlplus sys/Oradoc_db1@ORCLCDB as sysdba

Custom Configurations
The Oracle database server container also provides custom configuration parameters for starting up the container. All the custom configuration parameters are optional. The following list of custom configuration parameters can be provided in the ENV file (ora.conf).

DB_SID
This parameter changes the ORACLE_SID of the database. The default value is set to ORCLCDB.

DB_PDB
This parameter modifies the name of the PDB. The default value is set to ORCLPDB1.

DB_MEMORY
This parameter sets the memory requirement for the Oracle server. This value determines the amount of memory to be allocated for SGA and PGA. The default value is set to 2GB.

DB_DOMAIN
This parameter sets the domain to be used for database server. The default value is localdomain.

To start an Oracle database server with custom configuration parameters

$ docker run -it --name oracle -P --env-file ora.conf store/oracle/database-enterprise:12.2.0.1

Ensure custom values for DB_SID, DB_PDB and DB_DOMAIN are updated in the tnsnames.ora.

Caveats
This Docker image has the following restrictions.

Supports a single instance database.
Dataguard is not supported.
Database options and patching are not supported.
Changing default password for SYS user
The Oracle database server is started with a default password Oradoc_db1. The password used during the container creation is not secure and should be changed. To change the password connect to the database with SQL*Plus and execute

alter user sys identified by <new-password>;

Resource Requirements
The minimum requirements for the container is 8GB of disk space and 2GB of memory.

Database Logs
The database alert log can be viewed with

$ docker logs <Oracle-DB>

where is the name of the container

Reusing existing database
This Oracle database server image uses Docker data volumes to store data files, redo logs, audit logs, alert logs and trace files. The data volume is mounted inside the container at /ORCL. To start a database with a data volume using docker run command,

$ docker run -it --name oracle -v OracleDBData:/ORCL store/oracle/database-enterprise:12.2.0.1

OracleDBData is the data volume that is created by Docker and mounted inside the container at /ORCL. The persisted data files can be reused with another container by reusing the OracleDBData data volume.

Using host system directory for data volume
To use a directory on the host system for the data volume,

$ docker run -it --name oracle -v /data/OracleDBData:/ORCL store/oracle/database-enterprise:12.2.0.1

where /data/OracleDBData is a directory in the host system.

Oracle Database Server 12.2.0.1 Enterprise Edition Slim Variant
The Slim Variant (12.2.0.1-slim tag) of EE has reduced disk space (4GB) requirements and a quicker container startup. This image does not support the following features - Analytics, Oracle R, Oracle Label Security, Oracle Text, Oracle Application Express and Oracle DataVault. To use the slim variant

$ docker run -it --name oracle -p 1521:1521 store/oracle/database-enterprise:12.2.0.1-slim

where <Oracle-DB> is the name of the container and 12.2.0.1-slim is the Docker image tag.
