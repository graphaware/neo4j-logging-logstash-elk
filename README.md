neo4j-logback-elk
=====================

This sample demonstrates how to configure Neo4j to use Logback to broadcast to ELK, making use of logstash-logback-encoder and Neo4j Enterprise 3.0.6.

Note: This documentation is a work in progress.

### 1) Build
--------------------

The neo4j-logback-elk project builds an uber jar containing all of the transitive dependencies required for logstash-logback-encoder to function. This uber jar is then placed into the $neo4j_home/plugins directory, seemlessly providing Neo4j with all of the dependencies in one deployment. To do this:

1) Clone this repository

2) Open a shell and change into the newly cloned repository from step #1

3) Execute: 
```
mvn clean install
cp target/neo4j-logback-elk-*.jar $neo4j_home/plugin
```

### 2) Configuration
--------------------

Neo4j must be configured to make use of Logback and the logstash-logback-encoder. To do this, copy the logback.xml file contained within this project to your Neo4j's config directory. For example:
```
cp logback.xml $neo4j_home/config
```