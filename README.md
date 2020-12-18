
# Infrastructure

This repository contains a template of the Chimera infrastructure. The components are run from docker-compose images. If you don not have docker-compose, please follow this [official guide](https://docs.docker.com/compose/install/) to install it.

You can see how are used the components in this [demo](https://github.com/chimera-suite/use-case).

## Configuration

For correctly deploying the infrastructure it is needed to run all docker images in parallel and respecting the boot order.

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
It starts an Apache Spark cluster.
Moreover is starts an Apache Spark Thriftserver, that exposes a jdbc enpoint for SparkSQL queries.
You can access the Apache Spark dashbaord at [http://spark-master:8080](http://localhost:8084).

#### 3. Ontop

Before starting the docker-compose, it is needed to perform the configuration.
Ontop automatically loads the Spark JDBC drivers located in `/ontop/jdbc`, and theree files in the `/ontop/input` containing:
  1. the `*.owl` containing the ontological concepts needed by the Ontop reasoner for describing the semantic of the relational data stored in the Spark tables.
  2. the  `*.obda` file containing SQL-to-RDF mappings used by the Virtual Knowledge Graph mechanism of Ontop.
  3. the JDBC connection configuration for the Apache Spark ThriftServer, in the `spark_jdbc.properties` file.

The ontology and mapping files are missing in the `/ontop/input` folder and needs to be added by according to the relational data stored in the Spark tables, while the JDBC configuration file is already present.

Then, in the `docker-compose-ontop.yml` file, it is needed to bind the files with the enviroment variables as follow.

```
environment:
  - "ONTOP_ONTOLOGY_FILE=/opt/ontop/input/FILE.owl"  # ADD ONTOLOGY FILE FOR ONTOP
  - "ONTOP_MAPPING_FILE=/opt/ontop/input/FILE.obda"  # ADD MAPPING FILE
  - "ONTOP_PROPERTIES_FILE=/opt/ontop/input/spark_jdbc.properties"
```

Finally, it is possible to start the Ontop endpoint by running the following command

```
docker-compose -f docker-compose-ontop.yml up -d
```
This starts an Ontop instance.
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
