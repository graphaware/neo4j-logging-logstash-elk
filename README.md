neo4j-logback-elk
=====================

This sample demonstrates how to configure Neo4j to use Logback to broadcast to ELK, making use of logstash-logback-encoder and Neo4j Enterprise 3.0.6.

Note: This documentation is a work in progress.

### 1) Build
--------------------

The neo4j-logback-elk project builds an uber jar containing all of the transitive dependencies required for logstash-logback-encoder to function. This uber jar is then placed into the $neo4j_home/plugins directory, seemlessly providing Neo4j with all of the dependencies in one deployment. To do this:

1) Clone this repository
```
git clone https://github.com/graphaware/neo4j-logback-elk.git
```

2) Open a shell and change into the newly cloned repository from step #1

3) Execute: 
```
mvn clean install
cp target/neo4j-logback-elk-*.jar $neo4j_home/plugin
```

### 2) Configuration
--------------------

Neo4j must be configured to make use of Logback and the logstash-logback-encoder. To do this, copy the example below or the logback.xml file contained within this project to your Neo4j's config directory and customize as needed. For example:
```
cp logback.xml $neo4j_home/config
```

For convenient reference, an example logback.xml that outputs to the console, a file within $neo4j_home/logs, and broadcasts to ELK on 192.168.99.100:5000 is:
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