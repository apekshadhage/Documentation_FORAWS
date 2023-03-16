# User Manual

# IBM i AS/400 RPA Sink Connector for Confluent Platform

**Introduction** 

The purpose of As400 RPA connector is to enable IBM i (AS/400) integrations with other components.

The IBM AS/400 was first introduced in 1988 and evolved into a very stable modern all-purpose integrated solution that requires little or no management. The system is able to run core line of business applications securely and predictably, focusing on quality of service and availability and offering a compelling total cost of ownership for integrated on-premises solutions. IBM made several changes to the server and OS name (iSeries, System i, IBM i, Power Systems for i) but most still refer to it as AS/400.

The IBM i platform offers a number of integration options including PHP, Java, WebSphere, specialized lightweight web service containers, FTP, SMTP / emails, DB2 interfaces, data queues, integrated file systems - IFS, as well as number of products offered by IBM and Third party vendors. The main benefit of using "native" options such as Program Calls and Data Queues is that IBM i development team does not have to learn another language or purchase and support another technology in order to build integration layer, and can easily communicate with external systems using only traditional development tools.

**Prerequisites**

The connector is designed to work with IBM i objects (e.g. programs and data queues) and therefore this document assumes you are familiar with IBM i operational and development environments and tools.
This document assumes you are familiar with Confluent Kafka, Kafka Connect and Confluent Control Center and provides configuration examples and a detailed explanation of each and properties required to run the connector in Confluent Control Center.

**IBM i server configuration and requirements** 

-   IBM i must have ports 449, 8470, 8472,8473,8475 and 8476, 9470, 9472, 9473, 9475 and 9476 accessible from confluent runtime

-   IBM i must have *CENTRAL, *DTAQ, *RMTCMD, *SIGNON and *SRVMAP host servers running in the QSYSWRK subsystem

-   If secure TLS connection is used, the TLS certificate must be applied to Central, Data Queue, 
    Remote Command, Sign on, and DDM / DRDA services in Digital Certificate Manager

-   IBM i user ID must be authorized to perform the operations on the intended IBM i objects

-   If there’s an additional security software that locks down the remote execution functionality,
    the IBM i user ID defined for connector configuration must be allowed to execute remote calls 
    and access database, IFS and DDM services

**Dependencies** 

-   Access to IBM i server

-   Access to Confluent Kafka  

**Compatibility Matrix**
| Application/Service     | Version       |
| ----------------------- |---------------|
| Confluent Kafka         |6.0.0 or higher|
| Confluent Control Center|6.0.0 or higher|
| IBM i / OS400           |V5R4 or higher |
             

**Connector Operations** 

The IBM i connector is an operation-based connector, which means that when you add the 
connector to your Kafka connect cluster, you need to configure a specific operation the connector
is intended to perform. The connector supports the following operations:

| Operation                          | Description       |
| -----------------------------------|---------------|
| Execute script  | Execute a given key string to perform tasks in the IBMi system and/or run a predefined python macro to peform the tasks.|

## Confluent Setup
    
-  **Steps to setup confluent kafka standalone environment in local system**

      https://docs.confluent.io/5.5.0/quickstart/ce-quickstart.html

-  **Steps to setup confluent kafka standalone environment in cloud**

      https://dzone.com/articles/installing-and-configuring-confluent-platform-kafk


once the confluent kafka install follows the below steps for connector installation
            
  **Install the AS400 Connector** 

  Connectors are packaged as Kafka Connect plugins. Kafka connect isolates each plugin so that the 
  plugin libraries do not conflict with each other. Download and extract the ZIP file for 
  your connector and then follow the below installation instructions.
  
  -  **In Standalone mode**

      1.  kafka-connect-as400 connector can be downloaded directly from the confluent-hub using 
                                
                confluent-hub install infoviewsystems/kafka-connect-as400:1.1.1
                
      2.  Extract the content into the desired location (Preferred: /confluent/share/java or /confluent/share/confluent-hub-component)
          Generally it will install /confluent/share/confluent-hub-component location so no need to extract it manually.

      3.  Start confluent control center using command “confluent local services start”
      
      ![image](https://user-images.githubusercontent.com/46368616/133743614-0642b415-96c1-49c2-9501-507388935129.png)
      
      4.  Control center should be up and running and can be verified with http://{HOST}:9021
        
      ![image](https://bitbucket.org/infoviewsystems/docs.infoviewsystems.com/raw/8eac4575b5648231190d4563bd7c8f713a25d10a/docs/kafkaRPA/images/connector%20is%20available.png)
      
      5.  Sink connector is ready to configure now. And here are the sample configurations to be used.


- **Confluent Setup and connector installation through docker**

    1.  please find the predefine docker-compose.yml file
    
	     https://github.com/infoviewsystems/as400-gateway-kafka-doc/blob/main/docker-compose.yml
	       
    2.	Check once the connect service from the above file for 

		volumes:

		/home/ubuntu/license/:/opt/

		create license directory in local system (ex. mkdir license) and place the as400-license.lic file in the same directory.

		while running docker-compose.yml file license file i.e as400-license.lic will copy from local to docker container path i.e /opt/.
    
    3.	Now execute below command 
        
		docker-compose up -d

		It will download all images from docker hub and install 

		once downloading completed verify the status with below command is all services up and running

		docker-compose ps -a

		![image](https://user-images.githubusercontent.com/88314020/191201820-56c62361-3abb-48c5-8d8b-19b7a3d18530.png)

    
    4.  To verify the license is copied to /opt/, execute the below command to connect with kafka connect service with interactive mode
        
		docker exec -it connect bash

		execute below command to validate

		cd /opt/
		
		ls -l

		find the screenshot for reference

		![image](https://user-images.githubusercontent.com/88314020/191203434-03ebdc39-d320-4c38-b9de-a595950f89fc.png)


    5.  To verify the infoviewsystems-as400-kafka-connect is install or not go through below steps
         
		 cd /usr/share/confluent-hub-components/ 
		 
		 ls -l
		 
		 find the screenshot for reference

		 ![image](https://user-images.githubusercontent.com/88314020/191204410-9d57fe7d-b4ca-4d21-8b57-3238455b2468.png)

     
    6.  Control center should be up and running and can be verified with http://{HOST}:9021 

        ![image](https://bitbucket.org/infoviewsystems/docs.infoviewsystems.com/raw/8eac4575b5648231190d4563bd7c8f713a25d10a/docs/kafkaRPA/images/connector%20is%20available.png)

    7.  Source and Sink connectors are ready to configure now. And here are the sample configurations to be used. 



## Macro File Management:

The IBM i connector requires a macro file to execute operation on IBM i system(s). Accessing file in different ways by using different protocols such as S3,FILE and CLASSPATH etc. and accessing it through these protocols in our application needs to configure in connector configuration. Available Protocols to load macro(python script) file (S3, FILE, CLASSPATH). Based on the protocol parameters needs to be configure.
	
1. FILE
	
    find the attached screenshot for reference
		
	![image](https://bitbucket.org/infoviewsystems/docs.infoviewsystems.com/raw/8eac4575b5648231190d4563bd7c8f713a25d10a/docs/kafkaRPA/images/file.png)

	It requires two values 
		
	a. path 

	macro file path or python script path

	b. filename

	Provide the macro file name
		    
2. S3
	
	If the macro file wanted to access from  S3. Please find the screenshot for reference 
		 
	![image]()https://bitbucket.org/infoviewsystems/docs.infoviewsystems.com/raw/8eac4575b5648231190d4563bd7c8f713a25d10a/docs/kafkaRPA/images/s3.png

	It requires five values to access files from S3
		  
	a. S3 bucket path
		   
	b. Filename
		   
	c. S3 region
		   
	d. Access key
		   
	f. Secret Key 
		   
3. CLASSPATH	    
    
    It requires one value to aceess files from Classpath
    
	a. filename
		  

Based on protocol type needs to configure below properties


| Protocols     | Parameters to configure                                   |Mandatory                    |configuration keys for parameters |
|:--------------:|-----------------------------------------------------------|-----------------------------------|-----------------------------------|
| FILE| path <br> filename                                                   | required <br>  required           |as400.MaroFilePath<br> as400.MacroFileName|
| S3|S3 bucket path<br>filename<br>S3 region<br>Access key<br>Secret key     | required<br>required<br>required<br>required<br>required|s3.bucket<br>as400.MacroFileName<br>s3.region<br>s3.accessKey<br>s3.secretKey|
|CLASSPATH|filename|required|as400.MacroFileName|

Please contact Infoview Systems Connector support team at **(734) 293-2160** and **(+91) 4042707110** or via email sales@infoviewsystems.com and     marketing@infoviewsystems.com 
  
## AS400 RPA Connection Configuration Properties
-  **Connection**

| Parameter     | Description                                               |Mandatory                          |Default Value|configuration keys for parameters |
|---------------|-----------------------------------------------------------|-----------------------------------|-------------|---------------------------------|
| Name          |Enter a unique label for the connector in your application.| Required                          |AS400SourceConnectorConnector_0|name|
|AS400 URL      |AS400 system connection url.                              |    Required                          |null|as400.url|
|User Id        |AS400 System user                                          |Required     |null|as400.userId|
|PASSWORD     |AS400 system connection password.|Required|null|as400.password|
|License/truststore protocol |Please refere above mentioned license management section |Required|null|as400.license.protocol|
|IASP|Logical partitions in the systems.|Optional|null|as400.iasp|
|Library List|List of libraries, in addition to the library list associated with the user id. The libraries must be separated with comma and will be added to the top of the library list.|Optional|null|as400.libraries|
|Secure Connection|Enable secure connection with AS400 over encrypted channel.|Optional|False|as400.secure.connection|
|Socket Timeout|Socket Timeout, ms. Default value of -1 means the default JVM SO_TIMEOUT will be used|Optional|-1|as400.socket.timeout|
|Time unit to be used for Socket Timeout|Socket Timeout time unit|Optional|MILLISECONDS|as400.socket.timeunit|
|Connection Retries|Number of times to retry establishing the connection internally before throwing an exception and passing back to Kafka connection Manager.|Optional|3|as400.connection.retry|
|Reconnection Period|Time between internal reconnection retries in ms.|Optional|60000|as400.reconnection.period|
|Time unit to be used for Reconnection Period|Reconnection period time unit.|Optional|MILLISECONDS|as400.reconnection.timeunit|
|Connection Time to Live|Max time (Seconds) that the connection can be used.|Optional|0|as400.connection.live|
|Reconnection Period time out|Time out to be used for connection time out to live|Optional|SECONDS|

-  **Connection (Optional)**

| Parameter     | Description                                               |Mandatory                          |Default Value|configuration keys for parameters|
|---------------|-----------------------------------------------------------|-----------------------------------|-------------|----------------------------------|
|Operation Type |An Operation type to be done on AS400 FILE = 0, PRINT = 1,COMMAND = 2,DATAQUEUE = 3, DATABASE = 4, RECORDACCESS = 5, CENTRAL = 6, SIGNON = 7|Optional|2|as400.operation.type|
|CCSID|-|Optional|0|as400.ccsid|
|Pre Start Count Data Q|-|Optional|2|as400.prestart.count.dq|
|Pre Start Count Command|-|Optional|2|as400.prestart.count.command|
|Cleanup Interval|-|Optional|2|as400.cleanup.interval|
|Max Connection|Maximum connections allowed.|Optional|5|as400.max.connection|
|Max Inactivity|Maximum time to inactive the session for connection.|Optional|10|as400.max.inactivity|
|Max Lifetime|Maximum lifetime for connection.|Optional|60000|as400.max.lifetime|
|Max Use Count|-|Optional|10|as400.max.usecount|
|Max Use Time|-|Optional|30000|as400.max.usetime|
|Pre-Test Connection|-|Optional|true|as400.pretest.connection|
|Run Maintenance|-|Optional|true|as400.run.maintenance|
|Thread Used|-|Optional|true|as400.thread.used|
|Keep Alive|-|Optional|true|as400.keep.alive|
|Login Timeout|-|Optional|0|as400.login.timeout|
|Receive Buffer Size|-|Optional|1000|as400.receive.buffer.size|
|Send Buffer Size|-|Optional|1000|as400.send.buffer.size|
|So Linger|-|Optional|0|as400.so.linger|
|So Timeout|-|Optional|0|as400.so.timeout|
|TCP No Delay|-|Optional|true|as400.tcp.nodelay|

## AS400 Data Queue Sink Connector Configuration Properties

Configure these connector properties.

![image](https://user-images.githubusercontent.com/46368616/133767545-f4f0bf51-c9cb-4435-ac0f-04a5190f3502.png)

| Parameter     | Description                                               |Mandatory                          |Default Value|configuration keys for parameters|
|---------------|-----------------------------------------------------------|-----------------------------------|-------------|---------------------------------|
|Data Queue     |Write data queue name.|Required|null|as400.write.dataqueue.name|
|Library        |Write data queue library.|Required|null|as400.write.dataqueue.library|
|Is Keyed DataQ |Must be specified for keyed data queues and blank for non-keyed data queues. For reading any message from data queue.|Optional|false|as400.write.dataqueue.key|
|Format File Name|Optional parameter allows treating data queue entry as an externally defined data structure. When defined, the connector will dynamically retrieve the record format from the the specified IBM i file and parse the received data queue entry into the map of field name / value pairs. The connector will perform the type conversion, supporting all types such as packed, date / time etc.|Optional|null|as400.sink.format.name|
|Format File Library|When format file is specified, the format file library can also be specified, otherwise the format file will be located based on the connection library list.|Optional|null|as400.sink.file.library|
|DQ Entry Length|Max DQ Entry Length. When specified and greater than 0, the parameter value will be truncated to fit the max length.|Optional|0|as400.dq.entry.length|
|DQ Key Length|Max DQ Key Length. When specified and greater than 0, the parameter value will be used (instead of dynamically retrieving it from DQ definitions on the server).|Optional|null|as400.dq.key.length|


**Note:** Here is a sample property for AS400 Data Queue Sink connector configuration
```
{
  "name": "AS400DataQueueSinkConnectorConnector_0",
  "config": {
    "connector.class": "com.infoviewsystems.kafka.connect.as400.core.AS400DataQueueSinkConnector",
    "key.converter": "org.apache.kafka.connect.storage.StringConverter",
    "value.converter": "io.confluent.connect.json.JsonSchemaConverter",
    "topics": "default_ksql_processing_log",
    "as400.url": "xxxxxxxxxx",
    "as400.userId": "xxxxx",
    "as400.password": "xxxxxx",
    "as400.secure.connection": "false",
    "as400.license.protocol": "FILE",
    "as400.license.path": "/home/apeksha/license",
    "license.fileName": "as400-license.lic",
    "as400.write.dataqueue.name": "abc",
    "as400.write.dataqueue.library": "Library",
    "as400.sink.format.name": "xyz",
    "as400.sink.file.library": "Library",
    "value.converter.schema.registry.url": "http://localhost:8081"
  }
}
```


## Schema Registry Configuration
Schema Registry must be configured for Schema Converters to avoid problems with registration updated Schemas after updating Format File

**Avro**
```properties
io.confluent.connect.avro.AvroConverter
```
**JSON Schema**
```properties
io.confluent.connect.json.JsonSchemaConverter
```

Schema Registry can be configured in 3 ways:
1. Through Confluent Control Center

Open _Topic_ menu and choose topic where new messages publish. Then click on _Schema_ field. You will be able to see already registered schema. Click on pull down menu **_..._** and choose _Compatibility settings_.

![image](https://bitbucket.org/infoviewsystems/docs.infoviewsystems.com/raw/9db2caa99c34b62ff3d6fe5a6ae69564d3a11532/docs/AS400Gateway/Kafka/images/schema-registry-topic-settings.png)

Then choose compatibility level as **_NONE_** and save changes

![image](https://bitbucket.org/infoviewsystems/docs.infoviewsystems.com/raw/9db2caa99c34b62ff3d6fe5a6ae69564d3a11532/docs/AS400Gateway/Kafka/images/schema-compatibility-settings.png)

These changes will apply for only edited Topic. If you have another topics where messages published with schema you also need to repeat these steps for them.
If you don't want to repeat these steps for every new topic, use other options below. 

2. Through Confluent Platform properties file

Open installation folder for Confluent and go to _/etc/schema-registry/_ folder

Edit schema-registry.properties file with following command
```console
nano schema-registry.properties
```

Add new property in the end of file
```properties
schema.compatibility.level=none
```
Save changes and restart Confluent Platform if needed to apply changes
Compatibility will define as **_NONE_** as default for new schemas

3. Through Docker Compose File

Add new property for _schema-registry_ container in _environment_ section
```properties
SCHEMA_REGISTRY_SCHEMA_COMPATIBILITY_LEVEL: none
```

Then restart docker container with following command if needed to apply changes
```console
docker-compose up -d
```
Compatibility will define as **_NONE_** as default for new schemas

## Contact Us

[Contact us](http://www.infoviewsystems.com/contact-us) for connector pricing info, trial license, or support questions.
