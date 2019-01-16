# GBA Bridgehead

The Bridgehead consists of two main components, the [Store][1] and the [Connector][2] running in Tomcat 8. In addition to that, both need a Postgres 9.6 database.

Steps:
* Install and run Bridgehead ([Docker-Compose](#Docker-Compose) which is recommended, or manual with [WINDOWS](#WINDOWS) OR [LINUX](#LINUX))
* Create xml and import into Store ([Import-Readme](import/IMPORT.md))
* [Connect](#Connector) to central [Searchbroker](https://search.germanbiobanknode.de/)


## Docker-Compose
This Docker Compose project includes both, the [Store][1] and the [Connector][2], both Postgres databases and to showcase monitoring, a Prometheus instance together with Grafana.

1. [Install Docker](https://docs.docker.com/install/)
And test Docker in any command line with:
        
        docker run hello-world

2. After checking for free [ports](#Ports-outside-container), bring all up with:

        git clone https://github.com/martinbreu/gba-bridgehead-compose
        cd gba-bridgehead-compose
        docker-compose up

Docker compose will start all containers and print the logs to the console.

If you see database connection errors from the store or the connector, open a second terminal and run `docker-compose stop` followed by `docker-compose start`. Database connection problems should only occur at the first start because the store and the connector doesn't wait for the databases to be ready. Both try to connect at startup which might be to early.

If one needs only one of them, on can bring up only the specific one with:

```sh
docker-compose up store
docker-compose up connector
```

### Environment (Optional)

The Docker containers and docker-compose.yml accept certain environment variables:

#### Ports outside container

* PORT_STORE - defaults to `8081`
* PORT_CONNECTOR - defaults to `8082`
* PORT_STORE_METRICS - defaults to `9101`
* PORT_CONNECTOR_METRICS - defaults to `9102`
* PORT_STORE_POSTGRES - defaults to `5433`
* PORT_CONNECTOR_POSTGRES - defaults to `5434`
* PORT_PROMETHEUS - defaults to `9090`
* PORT_GRAFANA - defaults to `3000`

You can either stop all services occupying those ports or [change and save your personal environments](#Save-your-environments-(Optional)) 


#### Store specific (Optional)
* STORE_MDR_NAMESPACE - current namespace which changes after every change depending data-elements, defaults to `mdr16`
* STORE_MDR_MAP - mapping of mdr elements and db, defaults to `<dataElementGroup name="biobank">urn:mdr16:dataelementgroup:1:1</dataElementGroup><dataElementGroup name="collection">urn:mdr16:dataelementgroup:2:1</dataElementGroup><dataElementGroup name="sample">urn:mdr16:dataelementgroup:3:1</dataElementGroup><dataElementGroup name="sampleContext">urn:mdr16:dataelementgroup:4:1</dataElementGroup><dataElementGroup name="donor">urn:mdr16:dataelementgroup:5:1</dataElementGroup><dataElementGroup name="event">urn:mdr16:dataelementgroup:6:1</dataElementGroup>`
* STORE_MDR_VALIDATION - validation against mdr during store import, defaults to `true`
* STORE_POSTGRES_HOST - the host name of the Postgres DB, defaults to `store-db`. Change only if built-in-databse is not used
* STORE_POSTGRES_PORT - the port of the Postgres DB, defaults to `5432`. Change only if built-in-databse is not used
* STORE_POSTGRES_DB - the database name, defaults to `samply.store`
* STORE_POSTGRES_USER - the database username, defaults to `samply`
* STORE_POSTGRES_PASS - the database password, defaults to `samply`
* STORE_CATALINA_OPTS - JVM options for Tomcat, defaults to `-Xmx1g`

#### Connector specific (Optional)
* CONNECTOR_POSTGRES_HOST - the host name of the Postgres DB, defaults to `connector-db`. Change only if built-in-databse is not used
* CONNECTOR_POSTGRES_PORT - the port of the Postgres DB, defaults to `5432`. Change only if built-in-databse is not used
* CONNECTOR_POSTGRES_DB - the database name, defaults to `samply.connector`. Change only if built-in-databse is not used
* CONNECTOR_POSTGRES_USER - the database username, defaults to `samply`
* CONNECTOR_POSTGRES_PASS - the database password, defaults to `samply`
* CONNECTOR_STORE_URL - the URL of the store to connect to, defaults to `http://store:8080`. Change only if built-in-store is not used
* CONNECTOR_CATALINA_OPTS - JVM options, defaults to `-Xmx1g`
* CONNECTOR_OPERATOR_FIRST_NAME - the IT staff which runs the connector
* CONNECTOR_OPERATOR_LAST_NAME - the IT staff which runs the connector
* CONNECTOR_OPERATOR_EMAIL - the IT staff which runs the connector
* CONNECTOR_OPERATOR_PHONE - the IT staff which runs the connector
* CONNECTOR_MAIL_HOST - mail host which is able to send mails, defaults to ``
* CONNECTOR_MAIL_PORT - port to mail host, defaults to ``
* CONNECTOR_MAIL_PROTOCOL - protocol for mail, defaults to ``
* CONNECTOR_MAIL_FROM_ADDRESS - mail address from which mails are sent, defaults to ``
* CONNECTOR_MAIL_FROM_NAME - subject of mails, defaults to ``

#### Common (Optional)
* MDR_URL - api-url of mdr host, defaults to `http://mdr.germanbiobanknode.de/v3/api/mdr`
* PROXY_URL - the URL of the HTTP proxy to use for outgoing connections, "url:port"; enables proxy usage if set
* PROXY_USER - the user of the proxy account (optional)
* PROXY_PASS - the password of the proxy account (optional)


#### Save your environments (Optional)
In the repo directory, create a file called `.env`, here you can save your environments if default values changed.
Docker-Compose will find this file at startup.

`.env` example:
```
MDR_NAMESPACE=mdr17
MDR_URL=https://mdr.germanbiobanknode.de/v3/api/mdr
```



## WINDOWS

Under construction!!! (Follow config.yml and Dockerfile)

* Install [Tomcat 8.5](https://tomcat.apache.org/download-80.cgi)
* Install [Postgresql 9.6](https://www.postgresql.org/download/)
    * Create databases: samply.store and samply.connector with user and password = samply

* Copy and edit configuration files (DOWNLOADLINK MISSING) to 'C:\%username%\.config\samply\'
* Download and move WAR files to '%tomcat-dir%\webapps\
    https://maven.samply.de/nexus/service/local/repositories/oss-releases/content/de/samply/store-rest/4.2.6/store-rest-4.2.6.war
    https://maven.samply.de/nexus/service/local/repositories/oss-releases/content/de/samply/share-client/3.0.7/share-client-3.0.7.war

## LINUX

Under construction!!!  (Follow config.yml and Dockerfile)


## Description of running components

### Store

You can access the Store under http://localhost:8081. A simple test is fetching the import XSD under: http://localhost:8081/importXSD.

Most work is to import patient/sample data into the store. You have to create a xml file which will be tested against our Metadata Repository.
The current namespace where all data-elements are defined and all GBA-Components work with can be found under http://mdr.germanbiobanknode.de/view.xhtml?namespace=mdr16

Instructions for import can be found [here](import/IMPORT.md)

### Connector

You can access the Connector under http://localhost:8082 and login to it under http://localhost:8082/login.xhtml The login credentials are **admin**, **adminpass**.

Add a Searchbroker to get and answer queries at http://localhost:8082/admin/broker_list.xhtml 
* Broker Adresse = https://search.germanbiobanknode.de/broker/
* Ihre Email Adresse = your email address to get the API-Key for registration
* Automatisch antworten = Nur Anzahl (default, so you answer automatically with number of samples)

You will receive an email with API-Key from Searchbroker Backend, paste these eight numbers and press "ok". Call an ITC to validate your request.


Create a new user at http://localhost:8082/admin/user_list.xhtml

Logout and login as normal user to see all handled queries.


### Grafana (Optional)

You can access Grafana under http://localhost:3000. The login credentials are **admin**, **admin**.

There are two dashboards available. One for the Store and one for the Connector. Currently, they only show JVM metrics.

* Store dashboard: http://localhost:3000/d/wEuwMIpmz
* Connector dashboard: http://localhost:3000/d/wEuwMIpmy



[1]: <https://github.com/martinbreu/samply-store>
[2]: <https://github.com/martinbreu/samply-connector>
