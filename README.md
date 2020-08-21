# Docker: Oracle Database 18c XE

- [Pre](#Pre)
- [Build Image](#build-image)
- [Run Container](#run-container)
- [Container Commands](#container-commands)
- [Other](#other)
  - [SQL](#sql)
  - [SSH into Container](#ssh-into-container)

## Pre

1. [Download RPM](https://www.oracle.com/technetwork/database/database-technologies/express-edition/downloads/index.html): ~/Downloads/oracle-database-xe-18c-1.0-1.x86_64.rpm

2. _Optional:_ Setup docker network: `docker network create oracle_network`.

3. Create a folder `mkdir ora-data` : Oracle XE data -- volume.

## Build Image

```bash
-- Clone repo
git clone git@github.com:di2k/docker-oracle-xe.git

-- Set the working directory to the project folder
cd docker-oracle-xe

-- Copy the RPM to folder files
cp ~/Downloads/oracle-database-xe-18c-1.0-1.x86_64.rpm files/

-- Build Image
docker build -t oracle-xe:18c .
```

## Run Container

```bash
docker run -d -p 32118:1521 -p 35518:5500 --name=oracle-xe --volume ./ora-data:/opt/oracle/oradata --network=oracle_network oracle-xe:18c
  
# track of the initial installation by running:
docker logs oracle-xe
```

Run parameters:

Name | Required | Description 
--- | --- | ---
`-p 1521`| Required | TNS Listener. `32118:1521` maps `32118` on your laptop to `1521` on the container.
`-p 5500`| Optional | Enterprise Manager (EM) Express. `35518:5500` maps `35518` to your laptop to `5500` on the container. You can then access EM via https://localhost:35518/em 
`--name` | Optional | Name of container. Optional but recommended
`--volume /opt/oracle/oradata` | Optional | (recommended) If provided, data files will be stored here. If the container is destroyed can easily rebuild container using the data files.
`--network` | Optional | If other containers need to connect to this one (ex: [ORDS](https://github.com/martindsouza/docker-ords)) then they should all be on the same docker network.
`oracle-xe:18c` | Required | This is the `name:tag` of the docker image that was built in the previous step

## Container Commands

```bash
# Status:
# Check STATUS column for "(health: ...)".
docker ps

# Start container
docker start oracle-xe

# Stop container
docker stop -t 200 oracle-xe
```

## Other

### SQL

_Note `sqlcl` is an alias for [SQLcl](https://www.oracle.com/database/technologies/appdev/sqlcl.html). Can also use `sqlplus`_

```bash
-- Connect to CDB
sql sys/Oracle18@localhost:32118/XE as sysdba

-- Connect to default PDB
sql sys/Oracle18@localhost:32118/XEPDB1 as sysdba
```


```bash
-- DBeaver
host: localhost
port: 32118
database: XE
user: sys
role: SYSDBA
pass: Oracle18
```

### SSH into Container

Login to server:
```bash
docker exec -it oracle-xe bash -c "source /home/oracle/.bashrc; bash"
```

```bash
# Once connected to run sqlplus:
$ORACLE_HOME/bin/sqlplus sys/Oracle18@localhost/XE as sysdba
$ORACLE_HOME/bin/sqlplus sys/Oracle18@localhost/XEPDB1 as sysdba
```

```bash
# Listener start/stop
$ORACLE_HOME/bin/lsnrctl stop
$ORACLE_HOME/bin/lsnrctl start
```
