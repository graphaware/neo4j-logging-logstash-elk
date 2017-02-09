neo4j-logback-elk
=====================

This sample demonstrates how to configure Neo4j to use Logback to broadcast to ELK, making use of logstash-logback-encoder and Neo4j Enterprise 3.0.6.

Note: This documentation is a work in progress.

1) External Dependencies
--------------------

Logback requires several third party jars to broadcast to ELK. They are:
```
cd [neo4j_home]/lib

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

To verify Kafka is operating correctly, open two additional shells and execute:

```
#Do this in shell #3
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test

#Do this in shell #4
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
```

Next, enter some sample text into shell #3. If Kafka is functioning correctly, the text input into shell #3 will appear in shell #4.

2) Configuration
--------------------

### Neo4j: Installation

pim-neo4j-notification-plugin currently is compiled against Neo4j Enterprise 3.0.6. Please download and unpack Neo4j into the desired location. This location will be referred to as ${neo4j_home}.

### Neo4j: Configuration

Modify ${neo4j_home}/conf/neo4j.conf to include:
```
com.graphaware.runtime.enabled=true
com.graphaware.module.PimNotification.1=com.noon.module.pim.notification.PimNotificationBootstrapper
com.graphaware.module.PimNotification.mappingFile=mapping.json

#Optionally set the Kafka topic, defaults to 'pimNotification'
#com.graphaware.module.PimNotification.topic=pimNotification

#Optionally enable running on any HA type, default is MASTER_ONLY
#com.graphaware.module.PimNotification.haInclusionPolicy=ANY

#Optionally enable debugging for local development, testing
#dbms.jvm.additional=-agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=5005
```

This configuration:

1. Enables the GraphAware runtime

2. Registers the pim-neo4j-notification-plugin as the *first* plugin

3. Tells the pim-neo4j-notification-plugin

	1. To find the required mapping.json file in ${neo4j_home}/conf (see below for more details)
	
	2. If enabled, specifies the Kafka topic to send to
	
	3. If enabled, specifies the HA mode the module will operate within

	4. If enabled, starts Neo4j in debug mode
	
**Note: By design, the pim-neo4j-notification-plugin module only executes when executing within the *master* of a Neo4j cluster.**

**For local development you will certainly want to either run your local Neo4j as a cluster or configure this property to ANY.**

This can be changed by specifying the com.noon.module.pim.notification.PimNotification.haInclusionPolicy property, commented but included for reference in the sample above. Valid haInclusionPolicy values are ANY or MASTER_ONLY with MASTER_ONLY the default. 

### Deploy Required Plugin Dependencies

The pim-neo4j-notification-plugin requires additional libraries to made available to Neo4j. To do this, open a shell and execute: 
```
cd [neo4j_home]/plugins

wget http://products.graphaware.com/download/framework-server-enterprise/graphaware-server-enterprise-all-3.0.6.43.jar
```

Note: The Maven shade plugin is used to create an "uber" jar in which all defined dependencies (and their dependencies) are included in the final jar. When selecting witch jar to use within /target, the pim-neo4j-notification-plugin.jar will be the produced artifact minus the included depencies and the graphaware-pim-neo4j-notification-plugin.jar will be the uber; you will want the graphaware-pim-neo4j-notification-plugin.jar.

### Deploy the Plugin's mapping.json	
An example/sample mapping.json file can be found in the project's test/resources. Copy this file or obtain an example mapping.json file from a project team member and deploy it to ${neo4j_home}/conf.

### Compile, Package, and Deploy the Project

To compile, package, and deploy the pim-neo4j-notification-plugin project, open a shell and change into the project's root directory and execute:
```
mvn clean install
cp target/pim-neo4j-notification-plugin-*-SNAPSHOT.jar ${neo4j_home}/plugins/
```

### Test and Validate

With the above steps complete, start your Neo4j. Watch or tail the ${neo4j_home}/logs/neo4j.log and you should see the following entries:
```
...
2017-01-31 21:05:51.921+0000 INFO  [c.g.r.b.RuntimeKernelExtension] GraphAware Runtime enabled, bootstrapping...
...
2017-01-31 21:05:51.948+0000 INFO  [c.g.r.b.RuntimeKernelExtension] Bootstrapping module with order 1, ID PimNotification, using com.noon.module.pim.notification.PimNotificationBootstrapper
...
```

The above two lines indicate that the plugin is registered correctly and is ready for action.

Next, start your local instance of Kafka; you will likely want to monitor the topic to see the messages being broadcast.

With Kafka ready, fire the pim-neo4j-notification-plugin by creating a Product node (and related data) by executing the following Cypher statment:
```
WITH {id:"product123",gpc_code:"123456", sourceId:"EXCEL",values:[{key:"title",value:"Original Title"},{key:"product_type",value:"casual_bag"}]} AS map
MERGE (p:Product {id: map.id})
MERGE (i:ProductInformation {id: map.id + map.sourceId})
MERGE (p)-[:PRODUCT_INFORMATION]->(i)
WITH i, map
UNWIND map.values AS value
MERGE (v:ProductInformationElement {id: i.id + value.key})
SET v.key = value.key, v.value = value.value
MERGE (i)-[:INFORMATION_ELEMENT]->(v)
```
By watching the ${neo4j_home}/log/neo4j.log and/or by using your desired method of veiwing information broadcast to the 'pimNotification' topic, you should see long streams of 0's and 1's (ie: data) flying across the wire like magic.

See the PimNotificationModuleTest for other Cypher examples.

Please note that this example Cypher is current as of 02/2017 but likely will become out dated quickly.

### Sample Data

A small amount of sample data is included for exploration and testing and can be found in src/test/resources/sample-data.cypher. This data can me imported into Neo4j using the following shell command:
 ```
 $neo4j_home/bin/neo4j-shell -file [pim-neo4j-notification-plugin_home]/srctest/resources/sample-data.cypher
 ```
 
 Please note that this data is current as of 02/2017 but likely will become out dated quickly.
