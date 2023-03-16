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

    7.  RPA Sink connector ready to configure now. And here are the sample configurations to be used. 



## Macro File Management:

The IBM i connector requires a macro file to execute operation on IBM i system(s). Accessing file in different ways by using different protocols such as S3,FILE and CLASSPATH etc. and accessing it through these protocols in our application needs to configure in connector configuration. Available Protocols to load macro(python script) file (S3, FILE, CLASSPATH). Based on the protocol parameters needs to be configure.
 
 **Macro File**
 
 Enter the name of the python macro file that was placed into the above mentioned location
 The session variables, which holds all information about the current session, can be referenced as '_session' in the python macro. It is placed into the script beforehand by the module.

	
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
| Name          |Enter a unique label for the connector in your application.| Required                          |AS400RPASinkConnector_0|name|
|User Name     |Enter your AS400 account user name for login.               |    Required                          |null|as400.username|
|User Name X Position |The X position of the user name entry field at the login screen.|Required     |null|as400.usernameXPosition|
|User Name Y Position |The Y position of the user name entry field at the login screen.|Required|null|as400.usernameYPosition|
|Password|Enter your AS400 account password for login.|Required|null|as400.password|
|Password X Position|The X position of the password entry field at the login screen.|Required|null|as400.passwordXPosition|
|Password Y Position|The Y position of the password entry field at the login screen.|Required|null|as400.passwordYPosition|
|Initial Key String|Enter an optional formatted key string to naviagte the AS400 to an ideal screen after login.|Optional|null|as400.initialKeyString|
|Host|Enter the IBM i endpoint.|Optional|localhost|as400.host|
|Port|Enter the port number of your IBM i endpoint.|Optional|-1|as400.port|
|SSL Type|Enter the SSL Type of your IBM i endpoint if it requires a secure connection.|Optional|None|as400.sslType|
|Debug Mode|Enter either True or False. Entering true means that the connector will print the screen after any key is pressed. False means that it will not.|Optional|False|as400.debug|
|Num Attempts|Number of connection retries internally within AS400 connector configuration before the exception is raised to kafka runtime and connection management.|Optional|1|as400.numAttempts|
|Time Between Attempts (ms)|The amount of time inbetween connection retry attempts.|Optional|300|as400.TimeBetweenAttempts|

## AS400 RPA Sink Connector Configuration Properties

Configure these connector properties.

![image]([https://user-images.githubusercontent.com/46368616/133767545-f4f0bf51-c9cb-4435-ac0f-04a5190f3502.png](https://bitbucket.org/infoviewsystems/docs.infoviewsystems.com/raw/8eac4575b5648231190d4563bd7c8f713a25d10a/docs/kafkaRPA/images/Sink%20Parameters%20with%20writing%20response%20back%20params.png))

| Parameter     | Description                                               |Mandatory                          |Default Value|configuration keys for parameters|
|---------------|-----------------------------------------------------------|-----------------------------------|-------------|---------------------------------|
|Macro File Protocol    |refer macro file management section |Optional|FILE|as400.MacroFileProtocol|
|Key String |Enter a comma seperated list of key strokes, input field locations, or input field text, to be executed by the AS400/IBMi System.

Key strokes are defined by being placed inside of brackets. Ex. '[enter]', a full list of valid key strokes can be found starting at line 67 here.

Defining the current input field is done by placing the text SET_INFIELD inside brackets, followed by the x and y coordinates of the input field. Ex. [SET_INFIELD 3 4]

Defining an area of the screen a user wishes to grab is done by placing the text GET_SCREEN inside brackets, followed by the top left x and top left y coordinates of the input field you wish to grab, and then the width and height of the area. The final string inside the brackets is used to define the name of the output parameter a user wishes to store the text in. This will be stored in the attributes.screenOutput field of the mule message as a java HashMap.

Ex. [GET_SCREEN 9 10 10 3 output_test]

Defining when the user wants to run their python macro file is done by placing the text MACRO inside brackets.

Ex. [MACRO]

Input field text is defined by placing a string of whatever you wish to input into the comma seperated list.

FULL EXAMPLE: "[enter],[enter],4,[SET_INFIELD 2 1],3,[enter],[enter]"|Required |null |as400.KeyString|
|Input Parameters|Enter key-value pairs to replace pre-set variables (defined by placing a ':' before a variable name inside '< >' brackets corresponding to the key from the key-value pair) in the Key String.Ex. {"value": "4"} will replace :< value > in the Key String|Optional|null|as400.InputParameters|
|Sink Target Topic|Push Response back to kafka topic after execute script operation|required|null|sink.kafka.topic|
|Kafka Partition Key|Kafka Partition Key|Optional|null|sink.kafka.partition.key|

**Note:** Here is a sample property for AS400 Data Queue Sink connector configuration
```
{
 "name": "AS400RPASinkConnectorConnector_0",
  "config": {
    "connector.class": "com.infoviewsystems.kafka.connect.as400.rpa.core.AS400RPASinkConnector",
    "key.converter": "org.apache.kafka.connect.storage.StringConverter",
    "value.converter": "org.apache.kafka.connect.json.JsonConverter",
    "topics": "RPA",
    "as400.host": "172.20.34.10",
    "as400.username": "OROWADER",
    "as400.usernameXPosition": "53",
    "as400.usernameYPosition": "8",
    "as400.password": "DOAROW07",
    "as400.passwordXPosition": "53",
    "as400.passwordYPosition": "9",
    "as400.initialKeyString": "addlible apanda,[enter],[enter],addlible infocdccom,[enter],[enter]",
    "as400.port": "-1",
    "as400.sslType": "NONE",
    "as400.debug": "true",
    "as400.numAttempts": "1",
    "as400.TimeBetweenAttempts": "300",
    "as400.MacroFileProtocol": "FILE",
    "as400.MacroFilePath": "/home/apeksha/macro",
    "as400.MacroFileName": "addItems.py",
    "as400.KeyString": "call cdcordcmtc,[enter],[SET_INFIELD 29 10],:<uri>,[enter],[enter],[pf6],[MACRO],[pf10],[GET_SCREEN 15 4 8 1 orderNum],[GET_SCREEN 2 24 13 1  orderStatus],[pf12],[pf3]",
    "as400.InputParameters": "uri=orderId",
    "sink.kafka.topic": "sinktopic",
    "value.converter.schemas.enable": "false"
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
