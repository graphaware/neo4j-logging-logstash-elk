neo4j-logback-elk
=====================

This sample demonstrates how to configure Neo4j to use Logback to broadcast to ELK, making use of logstash-logback-encoder and Neo4j Enterprise 3.0.6.

Note: This documentation is a work in progress.

1) External Dependencies
--------------------

The logstash-logback-encoder requires several third party jars to broadcast to ELK. They are:
```
cd $neo4j_home/lib

wget http://central.maven.org/maven2/net/logstash/logback/logstash-logback-encoder/4.8/logstash-logback-encoder-4.8.jar
wget http://central.maven.org/maven2/ch/qos/logback/logback-core/1.1.6/logback-core-1.1.6.jar
wget http://central.maven.org/maven2/ch/qos/logback/logback-classic/1.1.6/logback-classic-1.1.6.jar
wget http://central.maven.org/maven2/ch/qos/logback/logback-access/1.1.6/logback-access-1.1.6.jar
wget http://central.maven.org/maven2/org/slf4j/slf4j-api/1.7.7/slf4j-api-1.7.7.jar
wget http://central.maven.org/maven2/com/fasterxml/jackson/core/jackson-databind/2.2.3/jackson-databind-2.2.3.jar
wget http://central.maven.org/maven2/com/fasterxml/jackson/core/jackson-annotations/2.3.3/jackson-annotations-2.3.3.jar
wget http://central.maven.org/maven2/com/lmax/disruptor/3.3.4/disruptor-3.3.4.jar
wget http://central.maven.org/maven2/com/fasterxml/jackson/core/jackson-core/2.3.3/jackson-core-2.3.3.jar

```

2) Configuration
--------------------

Neo4j must be configured to make use of Logback and the logstash-logback-encoder. To do this, copy the logback.xml file contained within this project to your Neo4j's /config directory. For example:
```
cp logback.xml $neo4j_home/config
```