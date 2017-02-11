neo4j-logging-logstash-elk
=====================

The neo4j-logging-logstash-elk project provides a concrete implementation and documentation of how to configure [Neo4j](https://neo4j.com) to make use of [Logback](https://logback.qos.ch/) and [LogStash](https://www.elastic.co/products/logstash) to broadcast log statements to [ElasticSearch](https://www.elastic.co/)/[LogStash](https://www.elastic.co/products/logstash)/[Kibana](https://www.elastic.co/products/kibana) (ELK). 

This implementation and configuration is performed at the Neo4j level, as opposed to configuring individual Neo4j module(s)/plugin(s), making the broadcasting of log statements to ELK available to all participating Neo4j modules/plugins. 

These instructions have been tested using logstash-logback-encoder 4.8 and Neo4j Enterprise 3.0.6.

Making use of this project falls into three basic steps:

1) [Building this project](#building-this-project)

2) [Configure Neo4j](#configure-neo4j)

3) [Update your code](#update-your-code)

--------------------
## Building This Project

The neo4j-logging-logstash-elk project builds an uber jar containing all of the transitive dependencies required for the logstash-logback-encoder to function. This uber jar is then placed into the $neo4j_home/plugins directory, seamlessly providing Neo4j with all of the dependencies in one deployment. 

To build this project and deploy it to your Neo4j instance(s):
```
git clone https://github.com/graphaware/neo4j-logging-logstash-elk
cd neo4j-logging-logstash-elk

mvn clean install

cp target/neo4j-logging-logstash-elk-*.jar $neo4j_home/plugin
```

--------------------
## Configure Neo4j


Neo4j (or perhaps more accurately, Logback) must be configured to make use of the logstash-logback-encoder. To do this, copy either the example below or the logback.xml file contained within this project to your Neo4j's config directory and customize as needed.

For convenient reference, below is an example logback.xml that outputs log statements to the console, to $neo4j_home/logs/logback.log, and broadcasts to ELK on **192.168.99.100:5000**. 
 
```
<configuration>

   <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
      <encoder>
         <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>		
      </encoder>      
   </appender>

   <appender name="FILE" class="ch.qos.logback.core.FileAppender">
      <file>../logs/logback.log</file>
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

Having performed the steps above to build the project, simply copy the above sample into $neo4j_home/config/logback.xml and restart Neo4j.

--------------------

## Update Your Code

In order for the logstash-logback-encoder to broadcast to ELK, the SLF4j logging framework must be used rather than GraphAware's typically used org.neo4j.logging.Log and com.graphaware.common.log.LoggerFactory or org.neo4j.logging.LogProvider classes.

This means your project **must** use:
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
