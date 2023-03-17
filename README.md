# User Manual

# IBM i AS/400 RPA Sink Connector for Confluent Platform

**Overview**

Companies running their core Systems of Record on IBM i (formerly known as AS400 or iSeries) platform often find themselves at the mercy of their application vendors when it comes to integrating these applications or automating manual user actions. AS400 RPA Sink Connector for Confluent is a simple automation connector that runs directly as a kafka connect application and simulates green screen user actions such as navigation, data entry and screen capture.  

# Use Cases

There are several options for integrating and automating IBM i based applications. Robotic Process Automation (RPA) is a good fit for companies that only have access to the application via green screen UI and have limited capabilities integrating directly at application program or DB level. If users have to manually key in the data into legacy system this is a good opportunity to automate this step. 

Another often overlooked scenario where RPA can help a lot is test automation. In many cases companies have to run through manual feature and regression testing every time the change is introduced. Automating this regression testing step could help improve the quality of the releases and reduce a risk of introducing the unintended changes in system behavior. 

The following two common use cases address the majority of IBM i integration requirements:

User Executes a Key String to Perform an Operation.

User Grabs Text off of the AS400/IBMi Screen.

User Executes a Pre-Defined Python Macro Script.

**User Executes a Key String to Perform an Operation**
1.User enters "4,[enter],[enter],2,[enter],[enter],1,[enter],[enter],[SET_INFIELD 11 2],5,[enter],[enter]" as the key string and the AS400/IBMi System will enter the strings presented and execute the keystrokes as if it the user were doing it themselves.

**User Grabs Text off of the AS400/IBMi Screen**
1.User enters "addlible muledemodk,[enter],addlible wtf400demo,[enter],[enter],call edtorder,[enter],[enter],[GET_SCREEN 9 10 10 3 output_test]" as the key string and the AS400/IBMi System will enter the strings presented and execute the keystrokes

2.The Module will grab the test within the area beginning at upper left x-position 9, upper left y-position 10, with a width of 10, and a height of 3.

3.The text will be stored in the output of the module in the attributes.screenOutput section. It will be stored in a HashMap behind they key 'output_test'.

**User Executes a Pre-Defined Python Macro Script**
1.User places a 'test.py' file inside the src/main/resources folder.

2.User defines the file in the module by typing the name 'test.py' in the Macro File input field in the Execute Script module. They also define "test_value" as a string in the macro output parameters input field. This string corresponds to a variable in the 'test.py' file with the same name.

3.User enters "addlible muledemodk,[enter],addlible wtf400demo,[enter],[enter],call edtorder,[enter],[enter],[MACRO]" as the key string and the AS400/IBMi System will enter the strings presented and execute the keystrokes

4.The Module will execute the script once it reaches the MACRO keyword in the keystring.

5.The output parameter will be stored in the payload of the module. It will be stored in a HashMap behind they key 'test_value'.


# Product Features

* Automates manual green screen operations and easily exposes legacy IBM i applications as modern APIs
* Simulates green screen user actions directly from kafka applications
* Focuses on one simple thing: automating user screen navigation and data entry for IBM i screen (tn5250) applications
* Supports simple python based script language for more advanced screen navigation logic
* Supports TLS (encrypted) connections

# License

AS400 RPA Sink Connector for Confluent is offered under GPL 2 open source license. 

# How does it work? 

Infoview's AS400 RPA Sink Connector is a simple kafka connect plug-in that opens IBM i telnet (5250) session, executes a sequence of user actions to navigate the screens, type the data into display fields, press function keys, read sections of the screen into the variables, etc. 

The connector works entirely within kafka connect application and does not require any other software to execute. 

The connection configuration includes the credentials and initial set of keystrokes that every session will execute to get to the starting point in the screen navigation (for example navigate to Order Entry screen)

The connector operation executes the script (a series of keystrokes, macro executions etc) then passess the resulting variables back to the kafka topic as a response. 

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

      https://docs.confluent.io/6.0.0/quickstart/ce-quickstart.html

-  **Steps to setup confluent kafka standalone environment in cloud**

      https://dzone.com/articles/installing-and-configuring-confluent-platform-kafk


once the confluent kafka install follows the below steps for connector installation
            
  **Install the AS400 Connector** 

  Connectors are packaged as Kafka Connect plugins. Kafka connect isolates each plugin so that the 
  plugin libraries do not conflict with each other. Download and extract the ZIP file for 
  your connector and then follow the below installation instructions.
  
  -  **In Standalone mode**

      1.  kafka-connect-as400-rpa connector can be downloaded directly from the confluent-hub using 
                                
                confluent-hub install infoviewsystems/kafka-connect-as400-rpa:1.0.0
                
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
	       
    2.	Check once the connect service from the above file for If wanted to access macro file using FILE protocol

		volumes:

		/home/ubuntu/macro/:/opt/

		create macro directory in local system (ex. mkdir macro) and place the python script file in the same directory.

		while running docker-compose.yml file macro file ex. addItems.py will copy from local to docker container path i.e /opt/.
    
    3.	Now execute below command 
        
		docker-compose up -d

		It will download all images from docker hub and install 

		once downloading completed verify the status with below command is all services up and running

		docker-compose ps -a

		![image](https://user-images.githubusercontent.com/88314020/191201820-56c62361-3abb-48c5-8d8b-19b7a3d18530.png)

    
    4.  To verify the macro file is copied to /opt/, execute the below command to connect with kafka connect service with interactive mode
        
		docker exec -it connect bash

		execute below command to validate

		cd /opt/
		
		ls -l


    5.  To verify the infoviewsystems-as400-kafka-connect-rpa is install or not go through below steps
         
		 cd /usr/share/confluent-hub-components/ 
		 
		 ls -l
		
     
    6.  Control center should be up and running and can be verified with http://{HOST}:9021 

        ![image](https://bitbucket.org/infoviewsystems/docs.infoviewsystems.com/raw/8eac4575b5648231190d4563bd7c8f713a25d10a/docs/kafkaRPA/images/connector%20is%20available.png)

    7.  RPA Sink connector ready to configure now. And here are the sample configurations to be used. 



## Macro File Management:

The IBM i connector requires a macro file to execute operation on IBM i system(s). Accessing file in different ways by using different protocols such as S3,FILE and CLASSPATH etc. and accessing it through these protocols in our application needs to configure in connector configuration. Available Protocols to load macro(python script) file (S3, FILE, CLASSPATH). Based on the protocol parameters needs to be configure.
 
 **Macro File**
 
 Enter the name of the python macro file that was placed into the above mentioned location
 The session variables, which holds all information about the current session, can be referenced as '_session' in the python macro. It is placed into the script   beforehand by the module.

	
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
| S3|S3 bucket path<br>filename<br>S3 region<br>Access key<br>Secret key    required<br>required<br>required<br>required<br>required|s3.bucket<br>as400.MacroFileName<br>s3.region<br>s3.accessKey<br>s3.secretKey|
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
|Key String |Enter a comma seperated list of key strokes, input field locations, or input field text, to be executed by the AS400/IBMi System. Key strokes are defined by being placed inside of brackets. Ex. '[enter]', a full list of valid key strokes can be found starting at line 67 [here](https://github.com/tn5250j/tn5250j/blob/master/src/org/tn5250j/keyboard/KeyMapper.java).<br> Defining the current input field is done by placing the text SET_INFIELD inside brackets, followed by the x and y coordinates of the input field.<br>Defining an area of the screen a user wishes to grab is done by placing the text GET_SCREEN inside brackets, followed by the top left x and top left y coordinates of the input field you wish to grab, and then the width and height of the area. The final string inside the brackets is used to define the name of the output parameter a user wishes to store the text in. This will be stored in the attributes.screenOutput field of the mule message as a java HashMap.<br>Ex. [GET_SCREEN 9 10 10 3 output_test]<br>Defining when the user wants to run their python macro file is done by placing the text MACRO inside brackets.<br>Ex. [MACRO]<br>Input field text is defined by placing a string of whatever you wish to input into the comma seperated list.<br>FULL EXAMPLE: "[enter],[enter],4,[SET_INFIELD 2 1],3,[enter],[enter]"|required|null|as400.keyString|
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
**Key names**
Below is a list of standard key codes and descriptions supported by the connector. The keys must appear exactly as they are defined below, and each key or literal or variable or special command must be separated by the comma


|Key Code | Description |
|:--      |:--          |
| [backspace] |Back Space|
| [backtab]|Back Tab|
|[up]|Cursor Up|
| [down]|Cursor Down|
| [left]|Cursor Left|
| [right]|Cursor Rignt|
| [delete]|Delete|
| [tab]|Tab|
| [eof]|End of Field|
| [eraseeof]|Erase EOF|
| [erasefld]|Erase Field|
| [insert]|Insert|
| [home]|Home|
| [keypad0]|Keypad 0|
| [keypad1]|Keypad 1|
| [keypad2]|Keypad 2|
| [keypad3]|Keypad 3|
| [keypad4]|Keypad 4|
| [keypad5]|Keypad 5|
| [keypad6]|Keypad 6|
| [keypad7]|Keypad 7|
| [keypad8]|Keypad 8|
| [keypad9]|Keypad 9|
| [keypad.]|Decimal|
| [keypad,]|Comma|
| [keypad-]|Keypad minus|
| [fldext]|Field Exit|
| [field+]|Field Plus|
| [field-]|Field Minus|
| [bof]|Beginning of Field|
| [enter]|Enter|
| [pf1]|F1|
| [pf2]|F2|
| [pf3]|F3|
| [pf4]|F4|
| [pf5]|F5|
| [pf6]|F6|
| [pf7]|F7|
| [pf8]|F8|
| [pf9]|F9|
| [pf10]|F10|
| [pf11]|F11|
| [pf12]|F12|
| [pf13]|F13|
| [pf14]|F14|
| [pf15]|F15|
| [pf16]|F16|
| [pf17]|F17|
| [pf18]|F18|
| [pf19]|F19|
| [pf20]|F20|
| [pf21]|F21|
| [pf22]|F22|
| [pf23]|F23|
| [pf24]|F24|
| [clear]|Clear|
| [pgup]|Page Up|
| [pgdown]|Page Down|
| [rollleft]|Roll Left|
| [rollright]|Roll Right|

**Special commands**

*[MACRO]* - execute Macro file. The Macro file name must be specified in the Macro File parameter. 

*[SET_INFIELD x y]* - positions the cursor to the input field at column x, row y

*[GET_SCREEN x y length height variable-name]* - reads screen area rectangle starting with column x / row y, with the horizontal size of <length> characters, and vertical size of <height> characters. The output will be stored into the variable-name and passed back into Mule flow or script.

**Parameter Mapping**

To set the value of the input fields dynamically, use the pattern :<variable-name> in the script, for example the below snippet positions the cursort to col 29 row 10, then inserts the content of variable "uri", then sends Enter key. 

    [SET_INFIELD 29 10],:<uri>,[enter] 

The variables must be defined in the Input Parameters section of the connector operation as a list of name and value pairs, for example the value of that variable is mapped to Mule flow value of customerId coming from the HTTP request:

    #[{"uri":attributes.uriParams.customerId}]
	
	
Python script example

        from org.tn5250j.framework.tn5250 import Rect

        print "--------------- tn5250j add items script start -------------"

        textBox = Rect()
        i = 9

        for item in items:
            field = _session.getScreen().getScreenFields().findByPosition(i-1, 2)
            field.setString(str(item["ItemNo"]))
            field = _session.getScreen().getScreenFields().findByPosition(i-1, 14)
            field.setString(str(item["ItemName"]))
            field = _session.getScreen().getScreenFields().findByPosition(i-1, 46)
            field.setString(str(item["Quantity"]))
            field = _session.getScreen().getScreenFields().findByPosition(i-1, 53)
            field.setString(str(item["UnitPrice"]))
            i = i + 1


        _session.getScreen().sendKeys("[enter]") 
        _session.getScreen().sendKeys("[enter]") 

        print "---------------- tn5250j add items script end -------------"


1. Connector executes a call to the order maintenance program and presses Enter.
2. Connector sets the cursor to col 29 row 10, and types the value of the variable uri then presses Enter twice. 
3. Connector sends F6 key event to navigate to new screen
4. Once on the new screen, the connector executes Python macro, passing the Items object coming as macro Input parameters, and for each line item types the data into      appropriate input fields on the Add Order lines subfile and presses Enter
5. After the order is entered, press F10 to create new order in the system
6. Retrieve newly generated order ID and status from the specified screen positions
7. Return back to the Work with Orders screen to prepare for the next transaction



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

