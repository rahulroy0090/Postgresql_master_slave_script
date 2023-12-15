 

 ## Full setup master and slave of postgresql using podman container.

```
#!/bin/bash

# Create a Podman network named rahulnetwork with subnet 172.19.0.0/24
echo "Creating Podman Network: rahulnetwork"
podman network create --subnet=172.19.0.0/24 rahulnetwork

# Create directories for mount volumes
echo "Creating Directories for Mount Volumes"
mkdir -p /home/hulk/data/psql/master
mkdir -p /home/hulk/data/psql/slave
mkdir -p /home/hulk/data/psql/repl

# Run Master Container
echo "Running Master Container"
podman run -d \
  --network rahulnetwork --ip 172.19.0.101 -p 5432:5432 \
  --name master -h master \
  -e "POSTGRES_DB=postgres" \
  -e "POSTGRES_USER=postgres" \
  -e "POSTGRES_PASSWORD=redhat" \
  -v /home/hulk/data/psql/master:/var/lib/postgresql/data \
  docker.io/postgres

# Wait for the Master container to be ready
echo "Waiting for Master container to be ready"
until podman exec master pg_isready -q; do
  sleep 1
done

# Run Slave Container
echo "Running Slave Container"
podman run -d \
  --network rahulnetwork --ip 172.19.0.102 -p 5433:5432 \
  --name slave -h slave \
  -e "POSTGRES_DB=postgres" \
  -e "POSTGRES_USER=postgres" \
  -e "POSTGRES_PASSWORD=redhat" \
  -v /home/hulk/data/psql/slave:/var/lib/postgresql/data \
  -v /home/hulk/data/psql/repl:/var/lib/postgresql/repl \
  docker.io/postgres

# Wait for the Slave container to be ready
echo "Waiting for Slave container to be ready"
until podman exec slave pg_isready -q; do
  sleep 1
done

echo "Containers are up and running!"

# Check the status of Podman containers
podman ps

# Replace 'master' with the actual name of your PostgreSQL container
CONTAINER_NAME="master"

# Create the role from the host system
podman exec -it "$CONTAINER_NAME" psql -U postgres -c "CREATE ROLE amit WITH LOGIN REPLICATION CONNECTION LIMIT 5 PASSWORD 'redhat';"

# Check the role status from the host system
podman exec -it "$CONTAINER_NAME" psql -U postgres -c "\du"

# Path for pg_hba.conf and postgresql.conf.
CONF_PATH="/home/hulk/data/psql/master"

# Create the directory if it doesn't exist
podman exec -it master mkdir -p "${CONF_PATH}"

# Update the container (Assuming Debian-based OS)
podman exec -it master apt-get update

# Install vim inside the container
podman exec -it master apt-get install -y vim

# Setup host inside the pg_hba.conf file
echo "host replication amit 172.19.0.102/24 md5" >> "${CONF_PATH}/pg_hba.conf"

# Configure the postgresql.conf
echo "archive_mode = on" >> "${CONF_PATH}/postgresql.conf"
echo "archive_command = '/bin/date'" >> "${CONF_PATH}/postgresql.conf"
echo "max_wal_senders = 10" >> "${CONF_PATH}/postgresql.conf"
echo "wal_keep_size = 16" >> "${CONF_PATH}/postgresql.conf"
echo "synchronous_standby_names = '*'" >> "${CONF_PATH}/postgresql.conf"

# Restart the master postgres container
podman stop master
podman start master

# Check the status of Podman containers after restart
podman ps

# Access the slave postgres container and run backup
podman exec -it slave /bin/bash -c " pg_basebackup -R -D /var/lib/postgresql/repl -Fp -Xs -v -P -h 172.19.0.101 -p 5432 -U amit"

# Remove the slave postgres container
podman rm -f slave

# Check the status of containers
podman ps

# Path for backup file
BACKUP_PATH="/home/hulk/data/psql"

# Check the backup files
ls "${BACKUP_PATH}"

# Remove the old slave directory
rm -rf "${BACKUP_PATH}/slave"

# Check the removed slave directory
ls "${BACKUP_PATH}"

# Rename the 'data' directory to 'slave'
mv "${BACKUP_PATH}/repl" "${BACKUP_PATH}/slave"

# Check if the 'slave' directory is present
ls "${BACKUP_PATH}"

# Setup the slave PostgreSQL container
podman run -d --network rahulnetwork --ip 172.19.0.102 -p 5433:5432 --name slave -h slave \
  -e "POSTGRES_DB=postgres" \
  -e "POSTGRES_USER=postgres" \
  -e "POSTGRES_PASSWORD=redhat" \
  -v "${BACKUP_PATH}/slave:/var/lib/postgresql/data" \
  docker.io/postgres

# Check the status of the slave container
podman ps

# Check the status of replication
podman exec master psql -U postgres -c "select * from pg_stat_replication;"




```







## Step1 : Setup master and slave postgresql using podman container.

 ```
 #!/bin/bash

# Create a Podman network named rahulnetwork with subnet 172.19.0.0/24
echo "Creating Podman Network: rahulnetwork"
podman network create --subnet=172.19.0.0/24 rahulnetwork

# Create directories for mount volumes
echo "Creating Directories for Mount Volumes"
mkdir -p /home/hulk/data/psql/master
mkdir -p /home/hulk/data/psql/slave
mkdir -p /home/hulk/data/psql/repl

# Run Master Container
echo "Running Master Container"
podman run -d \
  --network rahulnetwork --ip 172.19.0.101 -p 5432:5432 \
  --name master -h master \
  -e "POSTGRES_DB=postgres" \
  -e "POSTGRES_USER=postgres" \
  -e "POSTGRES_PASSWORD=redhat" \
  -v /home/hulk/data/psql/master:/var/lib/postgresql/data \
  docker.io/postgres

# Wait for the Master container to be ready
echo "Waiting for Master container to be ready"
until podman exec master pg_isready -q; do
  sleep 1
done

# Run Slave Container
echo "Running Slave Container"
podman run -d \
  --network rahulnetwork --ip 172.19.0.102 -p 5433:5432 \
  --name slave -h slave \
  -e "POSTGRES_DB=postgres" \
  -e "POSTGRES_USER=postgres" \
  -e "POSTGRES_PASSWORD=redhat" \
  -v /home/hulk/data/psql/slave:/var/lib/postgresql/data \
  -v /home/hulk/data/psql/repl:/var/lib/postgresql/repl \
  docker.io/postgres

# Wait for the Slave container to be ready
echo "Waiting for Slave container to be ready"
until podman exec slave pg_isready -q; do
  sleep 1
done

echo "Containers are up and running!"
  
 ```



## Setting of master postgresql confiregure.

 ```
 
 #!/bin/bash

# Check the status of Podman containers
podman ps

# Replace 'master' with the actual name of your PostgreSQL container
CONTAINER_NAME="master"

# Create the role from the host system
podman exec -it "$CONTAINER_NAME" psql -U postgres -c "CREATE ROLE amit WITH LOGIN REPLICATION CONNECTION LIMIT 5 PASSWORD 'redhat';"

# Check the role status from the host system
podman exec -it "$CONTAINER_NAME" psql -U postgres -c "\du"

# Path for pg_hba.conf and postgresql.conf.
CONF_PATH="/home/hulk/data/psql/master"

# Create the directory if it doesn't exist
podman exec -it master mkdir -p "${CONF_PATH}"

# Update the container (Assuming Debian-based OS)
podman exec -it master apt-get update

# Install vim inside the container
podman exec -it master apt-get install -y vim

# Setup host inside the pg_hba.conf file
echo "host replication amit 172.19.0.102/24 md5" >> "${CONF_PATH}/pg_hba.conf"

# Configure the postgresql.conf
echo "archive_mode = on" >> "${CONF_PATH}/postgresql.conf"
echo "archive_command = '/bin/date'" >> "${CONF_PATH}/postgresql.conf"
echo "max_wal_senders = 10" >> "${CONF_PATH}/postgresql.conf"
echo "wal_keep_size = 16" >> "${CONF_PATH}/postgresql.conf"
echo "synchronous_standby_names = '*'" >> "${CONF_PATH}/postgresql.conf"

# Restart the master postgres container
podman stop master
podman start master

# Check the status of Podman containers after restart
podman ps

 
 ```

# Setting slave postgresql and testing the status.

 ```
 
 #!/bin/bash

# Access the slave postgres container and run backup
#podman exec -it slave /bin/bash -c "pg_basebackup -h master -p 5432 -U rahul -D /var/lib/postgresql/repl -Fp -Xs -v -P"
podman exec -it slave /bin/bash -c " pg_basebackup -R -D /var/lib/postgresql/repl -Fp -Xs -v -P -h 172.19.0.101 -p 5432 -U amit"

# Remove the slave postgres container
podman rm -f slave

# Check the status of containers
podman ps

# Path for backup file
BACKUP_PATH="/home/hulk/data/psql"

# Check the backup files
ls "${BACKUP_PATH}"

# Remove the old slave directory
rm -rf "${BACKUP_PATH}/slave"

# Check the removed slave directory
ls "${BACKUP_PATH}"

# Rename the 'data' directory to 'slave'
mv "${BACKUP_PATH}/repl" "${BACKUP_PATH}/slave"

# Check if the 'slave' directory is present
ls "${BACKUP_PATH}"

# Setup the slave PostgreSQL container
podman run -d --network rahulnetwork --ip 172.19.0.102 -p 5433:5432 --name slave -h slave \
  -e "POSTGRES_DB=postgres" \
  -e "POSTGRES_USER=postgres" \
  -e "POSTGRES_PASSWORD=redhat" \
  -v "${BACKUP_PATH}/slave:/var/lib/postgresql/data" \
  docker.io/postgres

# Check the status of the slave container
podman ps


# Check the status of replication
podman exec master psql -U postgres -c "select * from pg_stat_replication;"

 
 ```