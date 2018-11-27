# Docker Compose File for GBA Bridgehead

The bridgehead consists of two main components, the store and the connector. In addition to that, both need a Postgres database. To showcase monitoring, a Prometheus instance together with Grafana is also included.

This Docker Compose project includes both, the [Store][1] and the [Connector][2]. If one needs only one of them, on can either bring up only the specific one or create a custom Docker Compose file.

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

Most work is to import patient/sample data into the store. You have to create a xml file which will be tested against our Metadata Repository.
The current namespace where all data-elements are defined and all GBA-Components work with can be found under http://mdr.germanbiobanknode.de/view.xhtml?namespace=mdr16

Instructions for import can be found [here](import/IMPORT.md)

### Connector

You can access the Connector under http://localhost:8081/gba-connector/ and login to it under http://localhost:8081/gba-connector/login.xhtml (username=admin, password=adminpass).

Add a Searchbroker to get and answer queries at http://localhost:8081/admin/broker_list.xhtml 
* Broker Adresse = https://search.germanbiobanknode.de/broker/
* Ihre Email Adresse = your email address to get the API-Key for registration
* Automatisch antworten = Nur Anzahl (default, so you answer automatically with number of samples)

You will receive an email with API-Key from Searchbroker Backend, paste these eight numbers and press "ok"

Create a new user at http://localhost:8081/gba-connector/admin/user_list.xhtml

Logout and login as normal user to see all handled queries.


### Grafana

You can access Grafana under http://localhost:3000. The login credentials are **admin**, **admin**.

There are two dashboards available. One for the Store and one for the Connector. Currently, they only show JVM metrics.

* Store dashboard: http://localhost:3000/d/wEuwMIpmz
* Connector dashboard: http://localhost:3000/d/wEuwMIpmy

## Partial Usage

One can bring up only parts of the services by starting them individually. For example: starting only the store by starting it's DB and the store itself:

```sh
docker-compose up -d store-db
docker-compose up -d store
```

For the connector:

```sh
docker-compose up -d connector-db
docker-compose up -d connector
```


[1]: <https://bitbucket.org/medicalinformatics/samply.store.docker>
[2]: <https://github.com/alexanderkiel/samply.connector.docker>
