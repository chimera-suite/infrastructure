
# Infrastructure

This repository contains a template of the Chimera infrastructure. The infrastructure template is composed of several docker images, that simulates the behaviour of data pipeline starting from big data technologies and passing trough semantic techs.

The components are run from docker-compose images. If you do not have it installed, please follow this [official guide](https://docs.docker.com/compose/install/).

The goal of this repository is to make available a model for integrating OntopSpark and PySPARQL into the data pipeline while minimizing your efforts. Instead, if you're only interested on trying to run Chimera on testing environment, we strongly suggest to look first at this [__demo__](https://github.com/chimera-suite/use-case).

## Configuration

Here we briefly discuss the components. For correctly deploying the infrastructure it is needed to run all docker images in parallel and respecting the boot order. Please take in consideration that some docker images (Apache Hive and Apache Spark) needs several minutes to complete the startup. We suggest to use 5 terminal windows (one for each component), and run all images without the `-d` option to be able to check the output logs.

__REMARK__: The infrastructural template is described from a docker's network viewpoint, as consequence all the network addresses and ports are expressed from an inside the docker's virtual network perspective. For connecting to the docker from outside the virtual network is needed to take the address of the machine and the port mapped on the left of the instance's dockerfiles. For example, in the below dockerfile snippet code, the `dummy-example` instance can be accessed from the internal docker network called `qwerty` at [http://dummy-example:10000](http://dummy-example:10000), instead from outside the network at [http://localhost:9000](http://localhost:9000) (assuming that the docker instance is run on the same machine).

```
version: "3.8"
services:
  dummy-example:
      image: dummy-example
      hostname: dummy-example
      container_name: dummy-example
      ports:
        - "9000:10000"
      environment:
        - "SPARK_MASTER=spark-master:7077"
        - "HADOOP_CONF_DIR=/spark/conf/hive-site.xml"
      volumes:
        - "./spark/conf/hive-site.xml:/spark/conf/hive-site.xml"

networks:
  default:
    external:
      name: qwerty
```

#### 1. Apache Hive
It simulates an `hdfs` file system.
```
docker-compose -f docker-compose-hive.yml up -d
```
You can use jdbc connection to `jdbc:hive2://hive-server:10000` to check the tables and run HiveQL queries.
An Hive metastore is available at `thrift:hive-metastore:9083`.

#### 2. Apache Spark
```
docker-compose -f docker-compose-spark.yml up -d
```
It starts an Apache Spark cluster, and loads the `hive-site.xml` that contains the configuration parameters for enabling the Spark tables HDFS persistance in Hadoop.
Moreover it starts an Apache `Spark Thrift Server` that exposes a JDBC enpoint. You can use jdbc connection to `jdbc:hive2://thriftserver:10000` for checking the tables and run SparkSQL queries.
You can access the Apache Spark dashbaord at [http://spark-master:8080](http://localhost:8084).

#### 3. Ontop

Before starting the docker-compose, it is needed to perform the configuration.
Ontop automatically loads the Spark JDBC drivers located in `/ontop/jdbc`, and theree files in the `/ontop/input` containing:
  1. the `*.owl` file containing the ontological concepts needed by the Ontop reasoner for describing the semantic of the relational data stored in the Spark tables.
  2. the  `*.obda` file containing SQL-to-RDF mappings used by the Virtual Knowledge Graph mechanism of Ontop.
  3. the JDBC connection configuration for the Apache Spark ThriftServer, in the `spark_jdbc.properties` file.

The ontology and mapping files are missing in the `/ontop/input` folder and needs to be added by according to the relational data stored in the Spark tables, while the JDBC configuration file is already present.

Then, in the `docker-compose-ontop.yml` file, it is needed to bind the files with the enviroment variables as follow.

```
environment:
  - "ONTOP_ONTOLOGY_FILE=/opt/ontop/input/FILE.owl"  # TODO: ADD ONTOLOGY FILE FOR ONTOP
  - "ONTOP_MAPPING_FILE=/opt/ontop/input/FILE.obda"  # TODO: ADD MAPPING FILE
  - "ONTOP_PROPERTIES_FILE=/opt/ontop/input/spark_jdbc.properties"
```

Finally, it is possible to start the OntopSpark endpoint instance by running the following command.

```
docker-compose -f docker-compose-ontop.yml up -d
```

In particular the instance is configured to communicate with the Apache Spark Thriftserver.
The web interface is available at [http://ontop:8080](http://localhost:8090).

#### 4. Jena Fuseki
```
docker-compose -f docker-compose-jena-fuseki.yml up -d
```
It start a Jena Fuseki container.
You can access the Jene Fuseki web interface at [http://jena-fuseki:3030](http://localhost:3030).
Moreover, once Jena Fuseki is ready, it is needed to log in with `admin:admin` (can be changed in the docker-compose file) and  manually load the Knowledge Graph by creating a new dataset in the `MANAGE DATASETS` section.


#### 5. Jupyter Notebook
```
docker-compose -f docker-compose-jupyter.yml up -d
```
You can access the Jupyter web interface to create and upload notebooks at [http://jupyter:8888](http://localhost:8888).
