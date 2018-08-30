# Docker Compose File for GBA Bridgehead

The bridgehead consists of two main components, the store and the connector. In addition to that both need a Postgres database. To showcase monitoring, a Prometheus instance together with Grafana is also included.

## Usage

* use a machine running Docker
* checkout this repo
* inside the repo dir, run `docker-compose up`

Docker compose will start all containers and print the logs to the console.

If you see database connection errors from the store or the connector, open a second terminal and run `docker-compose stop` followed by `docker-compose start`. Database connection problems should only occur at the first start because the store and the connector doesn't wait for the databases to be ready. Both try to connect at startup which might be to early.

### Ports

You might see Docker compose complaining about used ports. For this compose file to start successfully, you need to have the following ports available on your machine:

* 8080 - the Store
* 8081 - the Connector
* 9100 - Store metrics
* 9101 - Connector metrics
* 5432 - Store database
* 5433 - Connector database
* 9090 - Prometheus
* 3000 - Grafana

You can either stop all services occupying those ports or simply edit the `docker-compose.yml` at the root of the repo dir. Be sure to only edit the first (the inbound localhost) port in the ports declarations.

### Store

You can access the Store under http://localhost:8080/gba-store. A simple test is fetching the import XSD under: http://localhost:8080/gba-store/importXSD.

### Connector

You can access the Connector under http://localhost:8081/gba-connector/ and login to it under http://localhost:8081/gba-connector/login.xhtml.

### Grafana

You can access Grafana under http://localhost:3000. The login credentials are **admin**, **admin**.

There are two dashboards available. One for the Store and one for the Connector. Currently, they only show JVM metrics.

* Store dashboard: http://localhost:3000/d/wEuwMIpmz
* Connector dashboard: http://localhost:3000/d/wEuwMIpmy
