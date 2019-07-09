<p align="center">
    <img src="https://raw.githubusercontent.com/NeoBlocks/neoblocks/master/assets/logo/png/dark_square.png" width="128px" alt="NeoBlocks">
</p>

<h1 align="center">NeoBlocks</h1>
<h3 align="center">Installation</h3>

<p align="center">
    Building an ecosystem to make the Smart Economy accessible and secure for as many people as possible.
</p>

# Summary
The NeoBlocks platform delivers an infrastructure to read and index the NEO blockchain and add various endpoints to access
the data. For each instance we need to prepare a database server and allow a `neoblocks-node` container to sync with the NEO
network. The `neoblocks-agent` container will read data from the node, parse it and write it to the database server. Any
endpoint service will read the data directly from the database server. We are using Docker to run the containers.

<p align="center">
    <img src="https://github.com/NeoBlocks/neoblocks/raw/master/assets/infrastrucuture.png" width="800px" alt="NeoBlocks infrastructure">
</p>

# Installation
These are simplified installation instructions for a complete NeoBlocks instance. Please note that some required components
have not been released at this point. Please contact us for additional information or licensing.

Before continuing read more about the [`neoblocks-node`](https://github.com/NeoBlocks/neoblocks-node/)
and [`neoblocks-agent`](https://github.com/NeoBlocks/neoblocks-agent/) packages.

  * Prepare three servers `node1` `node2` and `node3`
    * Recommended for `node1` and `node2` is a minimum of `8GB` memory, `80GB` disk and `4` CPU cores
    * Recommended for `node3` is a minimum of `16GB` memory, `80GB` disk and `8` CPU cores
  * Install Docker and MySQL server
  * Fetch the repositories, build the Docker images

## neoblocks-node
```bash
ssh node1
apt install docker.io
docker network create -d bridge neoblocks

cd /usr/local/src
git clone git@github.com:NeoBlocks/neoblocks-node.git
cd neoblocks-node
docker build -t neoblocks/neoblocks-node:2.10.2 .
docker tag neoblocks/neoblocks-node:2.10.2 neoblocks/neoblocks-node
```
## neoblocks-agent
```bash
ssh node2
apt install docker.io
docker network create -d bridge neoblocks

cd /usr/local/src
git clone git@github.com:NeoBlocks/neoblocks-agent.git
cd neoblocks-agent
docker build -t neoblocks/neoblocks-agent:1.3.3 .
docker tag neoblocks/neoblocks-agent:1.3.3 neoblocks/neoblocks-agent

cd /usr/local/src
git clone git@github.com:NeoBlocks/neoblocks-api-service.git
cd neoblocks-api-service
docker build -t neoblocks/neoblocks-api-service:1.0.0 .
docker tag neoblocks/neoblocks-api-service:1.0.0 neoblocks/neoblocks-api-service
```
## Database server
```bash
ssh node3
apt install mysql-server
cd /usr/local/src
git clone git@github.com:NeoBlocks/neoblocks-agent.git
mysql
```
```sql
-- Import the NeoBlocks schema and stored procedures
SOURCE /usr/local/src/neoblocks-agent/schema.sql;

-- Create a user for the neoblocks-agent
CREATE USER 'neo'@'%' IDENTIFIED BY 'neo';
GRANT ALL PRIVILEGES ON `neoblocks`.* TO 'neo'@'%';
GRANT ALL PRIVILEGES ON `statistics`.* TO 'neo'@'%';
GRANT ALL PRIVILEGES ON `asset`.* TO 'neo'@'%';
GRANT ALL PRIVILEGES ON `token`.* TO 'neo'@'%';

-- Create a read-only user for the other neoblocks services
CREATE USER 'neoblocks'@'%' IDENTIFIED BY 'neoblocks';
GRANT SELECT ON `neoblocks`.* TO 'neoblocks'@'%';
GRANT SELECT ON `statistics`.* TO 'neoblocks'@'%';
GRANT SELECT ON `asset`.* TO 'neoblocks'@'%';
GRANT SELECT ON `token`.* TO 'neoblocks'@'%';

FLUSH PRIVILEGES;
```

First start the `neoblocks-node` container, then the `neoblocks-agent` container and finally the
`neoblocks-api-service` container. The synchronization is now starting and will take one or two days
to complete, depending on the used hardware and the network performance.

