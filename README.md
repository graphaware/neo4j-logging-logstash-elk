neo4j-logging-logstash-elk
=====================

The neo4j-logging-logstash-elk project provides a concrete implementation and documentation of how to configure [Neo4j](https://neo4j.com) to make use of [Logback](https://logback.qos.ch/) and [LogStash](https://www.elastic.co/products/logstash) to broadcast log statements to [ElasticSearch](https://www.elastic.co/)/[LogStash](https://www.elastic.co/products/logstash)/[Kibana](https://www.elastic.co/products/kibana) (ELK). 

This implementation and configuration is performed at the Neo4j level, as opposed to configuring individual Neo4j module(s)/plugin(s), making the broadcasting of log statements to ELK available to all participating Neo4j modules/plugins. 

These instructions have been tested using logstash-logback-encoder 4.8 and Neo4j Enterprise 3.0.6.

Making use of this project falls into three basic steps:

1) Building this project

2) Configure the Neo4j instance(s)

3) Updating your code

--------------------
## 1) Building This Project

The neo4j-logging-logstash-elk project builds an uber jar containing all of the transitive dependencies required for the logstash-logback-encoder to function. This uber jar is then placed into the $neo4j_home/plugins directory, seamlessly providing Neo4j with all of the dependencies in one deployment. To do this:

1) Clone this repository
```
git clone https://github.com/graphaware/neo4j-logging-logstash-elk.git
```

2) Open a shell and change into the newly cloned repository from step #1

3) Execute: 
```
mvn clean install
cp target/neo4j-logging-logstash-elk-*.jar $neo4j_home/plugin
```

--------------------
## 2) Configure the Neo4j Instance(s)


Neo4j (or perhaps more accurately, Logback) must be configured to make use of the logstash-logback-encoder. To do this, copy either the example below or the logback.xml file contained within this project to your Neo4j's config directory and customize as needed.

For convenient reference, an example logback.xml that outputs to the console, a file within $neo4j_home/logs, and broadcasts to ELK on **192.168.99.100:5000** is as follows. Having performed the steps from #1 above, simply copy the below contents into $neo4j_home/config/logback.xml and restart Neo4j. 
```
<configuration>

   <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
      <encoder>
         <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>		
      </encoder>      
   </appender>

   <appender name="FILE" class="ch.qos.logback.core.FileAppender">
      <file>../logs/logback.txt</file>
      <append>true</append>
      <encoder>
         <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
      </encoder>
   </appender>
     <appender name="stash" class="net.logstash.logback.appender.LogstashTcpSocketAppender">

         <destination>192.168.99.100:5000</destination>
          <keepAliveDuration>5 minutes</keepAliveDuration>

         <encoder class="net.logstash.logback.encoder.LogstashEncoder">
         </encoder>

     </appender>

     <root level="info">          
          <appender-ref ref="STDOUT" />
          <appender-ref ref="FILE" />
          <appender-ref ref="stash" />
      </root>

     <logger name="com.graphaware" level="DEBUG">
             <appender-ref ref="STDOUT" />
             <appender-ref ref="FILE" />
             <appender-ref ref="stash" />
     </logger>

</configuration>
```

Please tailor the above examples to your specific needs. 

## 3) Updating Your Code
--------------------
In order for the logstash-logback-encoder to broadcast to ELK, the SLF4j logging framework must be used rather than GraphAware's typically used org.neo4j.logging.Log and com.graphaware.common.log.LoggerFactory or Neo4j's logging classes.

This means your project should use:
```
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
...
private static final Logger LOG = LoggerFactory.getLogger(Exmple.class);
```
rather than
```
import import org.neo4j.logging.Log;
import com.graphaware.common.log.LoggerFactory;
...
private static final Log LOG = LoggerFactory.getLogger(Exmple.class);
```
