# Introduction

Infoview Systems AS400Gateway suite of products eliminates the stress and impact of IBM i / AS400 legacy system integration on development teams, minimizes the time and resources put into building integrations by hand, and enables non-IBM i developers to unlock legacy business logic and data directly from the comfort of their modern integration development stack. AS400Gateway for AWS is an Amazon Machine Image (AMI) that provides generic REST API integration for IBM i operations (Program call, Service Program call, Command call, Data Queue Read and Write). In addition the product can be configured to continuously listen for Data Queue messages and forward them to AWS SQS or SNS for further processing.

The product provides a convenient out of the box IBM i integration capabilities for companies running their enterprise integrations on AWS stack. Certified and rigorously tested by AWS marketplace team, the connector was designed to accelerate IBM i / AS400 integrations with other systems and services. In addition to the core product available to customers in AMI format, we also provide a reference AWS architecture that includes a sample API and event based use cases.

We are a global cross-platform service team with a unique fluency in legacy and modern technology stacks, including Mulesoft and IBM i / AS400. Infoview&#39;s dedicated customer success representative coordinates just-in-time technical assistance and support to client teams ensuring you have all the help you may need, when you need it. We are more than happy to provide a trial license for our products, participate in discovery sessions, run live demos for typical integration scenarios with our Gateway products, as well as assist with or perform a proof of concept based on particular use cases.

[Contact us](http://www.infoviewsystems.com/contact-us) for connector pricing info, trial license, or support questions.

# AS400Gateway for AWS Overview

The IBM AS/400 was first introduced in 1988 and evolved into a very stable modern all-purpose integrated solution that requires little or no management. The system is able to run core line of business applications securely and predictably, focusing on quality of service and availability and offering a compelling total cost of ownership for integrated on-premises solutions. IBM made several changes to the server and OS name (iSeries, System i, IBM i, Power Systems for i) but most still refer to it as AS/400.

The IBM i platform offers a number of integration options including PHP, Java, WebSphere, specialized lightweight web service containers, FTP, SMTP / emails, DB2 interfaces, data queues, integrated file systems - IFS, as well as number of products offered by IBM and Third party vendors. The main benefit of using &quot;native&quot; options such as Program Calls and Data Queues is that integration or cloud development team does not have to acquire in-depth knowledge of IBM i technologies in order to build integration layer, and can easily communicate with external systems using only traditional development tools.

Program Call is the most straightforward and low code option for exposing IBM i business logic as a reusable API. The AS400Gateway for AWS provides a generic API interface to call any program on IBM i, passing parameters into the program and receiving the results back in real time.

Data queues are native IBM i objects designed primarily for inter-process communications. They are lightweight persistent queues that support processing by a key, FIFO or LIFO. The majority of integration use cases can be implemented with the pair of request and response Data Queues. Source system places a message to request data queue and waits for acknowledgement message on response data queue. The target system receives and processes a message from the request data queue then places the acknowledgement to the response data queue.

The AS400Gateway for AWS provides the REST API to read or write the messages into Data Queue on demand, or continuously listen for new Data Queue messages and forward to AWS SQS / SNS in near real time.

# IBM i Prerequisites

- IBM i OS version:V5R4 and higher
- AWS VPC where the AMI is running must be able to reach the IBM i servers on ports 446, 449, 8470, 8472,8473,8475 and 8476 for non-SSL communications, and ports 448, 449, 9470, 9472, 9473, 9475 and 9476 accessible for SSL communications.
- IBM i must have **\*CENTRAL, \*DTAQ, \*RMTCMD, \*SIGNON and \*SRVMAP** host servers running in the QSYSWRK subsystem
- If secure TLS connection is used, the TLS certificate must be applied to Central, Data Queue, Remote Command, File, Signon, and DDM / DRDA services in Digital Certificate Manager
- IBM i user ID must be authorized to perform the operations on the intended IBM i objects
- If there&#39;s an additional security software that locks down the remote execution functionality, the IBM i user ID defined for connector configuration must be allowed to execute remote calls and access database, IFS and DDM services

# AWS Reference Architecture

Below is a sample AWS architecture that includes various services and components typically used when implementing cloud integration solution for bidirectional integrations with IBM i based back-end systems.

![image](https://user-images.githubusercontent.com/88314020/165104309-cde0f780-ac9d-4512-aad2-bda005bfc01f.png)



Salesforce is used as a sample external system that sends orders to IBM i based ERP and receives order statuses back from the ERP in near real time. Note that most companies will likely have different services and applications used for API management, security policies, token validations, routing and other AWS components interacting with AS400Gateway for AWS.

**Prerequisites for use case:**

1. Salesforce account
2. Lambda functions
3. SNS topic
 
**Steps to create lambda function in AWS and configure cloudwatch:**


**Steps to create SNS topic:**

Go to Simple Notification Services&rarr;click on Topics&rarr;Create Topic&rarr;select Type as standard&rarr;name e.g (as400Topic)&rarr;set other properties if you want&rarr;click on create topic.

**create subscription for sns topic:**

Click on Create Subcription&rarr;select protocol(e.g. AWS Lambda)&rarr;Enter a valid AWS Lambda ARN (for example, arn:aws:lambda:us-east-1:123456789012:function:MyLambdaFunction)&rarr;After your subscription is created, you must confirm it.&rarr;set bother properties according(Optional)&rarr;click on Create Subscription




![image](https://user-images.githubusercontent.com/88314020/165104639-b994e4a4-1c9b-4a54-9af2-0c2c7b8a3be0.png)

Considering an use case with Salesforce is a source system whenever we create a record in salesforce we use Apex classes in salesforce to fetch the record and push to API gateway since AS400 gateway doesn’t understand the request coming from Salesforce we use Lambda function in between Salesforce and API gateway to format the input in AS400 consumable format.
We do necessary operations in AS400 gateway and response will be push to SNS topic Lambda function pick the messages from SNS topic and update response back to the salesforce.




# AWS setup guide

Here are the following AWS services for implementing the solution

| **#** | **Resource Name** | **Qty** |
| --- | --- | --- |
| 1 | VPC | 1 |
| 2 | Subnets | 2 |
| 3 | EC2 Instance | 2 |
| 4 | NLB | 1 |
| 5 | Security Groups | 2 |
| 6 | NACL | 2 |
| 7 | SNS | 1 |
| 8 | Lambda Functions | 2 |
| 9 | CloudWatch Logs | 2 |
| 10 | VPCLink | 1 |
| 11 | AWS Gateway API | 1 |

AWS user account with appropriate roles for managing EC2 instances, AWS API Gateway, AWS Lambda, VPC, CloudWatch Logs etc.

**Step-by-Step Solution Implementation:**

1. Login / Sign into AWS Management Console
2. Create a VPC with couple of subnets. One subnet is configured as public subnet and other is configured as private subnet

   **Steps to create VPC**
   
   Click on Services&rarr;VPC&rarr;your VPCs&rarr;click on Create VPC &rarr;give name as MyVPC and select CIDR block as 10.0.0.0/16 and click on create VPC button.
   
   You successfully created vpc-0b91b185302d79fea / MyVPC
   
   **Steps to create Subnet**
   
   Click on Subnets&rarr;click on Create subnet &rarr;select VPC ID as vpc-0b91b185302d79fea (MyVPC)
   Subnet name is publicsubnet and give availability zone as no preferences and IPV4 CIDR block as 10.0.0.0/24 
   Click on AddNew subnet button 
   Name as privatesubnet and give availability zone as no preferences and IPV4 CIDR block as 10.0.1.0/24 
   
   You have successfully created 2 subnets: subnet-002169dd0dd057d4f, subnet-02a9384689d068bcb
   
   **Steps to create Internet Gateway**
   
   Click on Internet Gateway&rarr;create Internet Gateway&rarr;give name as igwMyVpc&rarr;click on create internet Gateway
   The following internet gateway was created: igw-08b7822759edbeeee - igwMyVpc. You can now attach to a VPC to enable the VPC to communicate with the internet.
   Select IGW i.e igwMyVpc&rarr;go to Action tab&rarr;attach to vpc&rarr;select MyVPC and click on attach to vpc
  
   Now we need to define roote to for networks to go to the internet
   Create 2 route tables 1 is for public subnet and 2nd one for private subnet
  
   **Steps to create route tables**
  
   Click on route tables&rarr;create route table&rarr;give name as publicroute and select vpc i.e MyVpc&rarr;click on create button
  
   Route table rtb-0e444f5f4e1cf1f13 | publicroute was created successfully.
  
   Click on publicroute&rarr;subnet Associations&rarr;click on edit subnet association&rarr;select publicsubnetForMyVpc&rarr;click on save associations 
  
   You have successfully updated subnet associations for rtb-0e444f5f4e1cf1f13 / publicroute.
  
   Select publicroute&rarr;routs&rarr;edit routes&rarr;add routes&rarr;add 0.0.0.0/0  and target as Internet Gateway 
   igw-08b7822759edbeeee(MyVpc)&rarr;click on save changes
  
   Updated routes for rtb-0e444f5f4e1cf1f13 / publicroute successfully

   create route table&rarr;give name as privateroute and select vpc i.e MyVpc&rarr;click on create button
  
   Route table rtb-0e63259e088aa4ea7 | privateroute was created successfully.
  
   Click on privateroute&rarr;subnet Associations&rarr;click on edit subnet association&rarr;select privatesubnetForMyVpc&rarr;click on save associations 
  
   You have successfully updated subnet associations for rtb-0e63259e088aa4ea7 / privateroute.


4. Create an EC2 instance by selecting AMI from AWS marketplace. and attach this instance to private subnet. This instance hosts the AS400 API interfaces which in turn communicates with back-end AS400 servers through Infoview AS400 connector.

   **Steps to create EC2 Instance in privatesubnets inside VPC**
   
    Click on Launch Instance&rarr;give name as AS400CommonAPiForMyVpc&rarr;click on My AMIs&rarr;select AS400-common-API ami&rarr;select Instance type according to         requirement(eg.t2.micro)&rarr;go with create new key pair option create as .ppk file by selection .ppk option and click on create.(securedAPI)&rarr;select VPC(eg.     MyVpc)&rarr;select privatesubnetForMyvpc&rarr;make sure Auto Assign  public Ip is Disable.&rarr;select create security group                                           option(eg.securityGroupForMyVpc)&rarr;add description related to security group(Optional)&rarr;(Add All rules which needs to be added to security group)&rarr;
    if requires add storage&rarr;finally click on Launch Instance
    
    Successfully initiated launch of instance (i-0cf3ed2965af92b3e)

6. Create a normal EC2 instance and attach this instance to public subnet. This instance will act as a bastion host or NAT Gateway host.
   
    **Steps to create EC2 Instance in publicsubnets inside MyVPC**
    
    Click on Launch Instance&rarr;give name as AS400GatewayForMyVpc&rarr;select ubuntu or any other os&rarr;select Instance type according to requirement(eg.               t2.micro)&rarr;go with create new key pair option create as .ppk file by selection .ppk option and click on create.(eg.publickey)&rarr;select VPC (eg.                 MyVpc)&rarr;select publicsubnetForMyVpc&rarr;make sure Auto Assign public Ip is enable.&rarr;select existing security group(eg. securityGroupForMyVpc)&rarr;if         requires add storage&rarr;finally click on Launch Instance
    
    Successfully initiated launch of instance (i-0f9569de3565e58e1)

8. Create a Network Load Balance and attach it to bastion host

   **Steps to Configure Load Balancer inside MyVpc**
   
   To create Network Load Balancer Target group first needs to be created
   
   Click on create target group&rarr;choose a target type instances&rarr;give Target Group Name (e.g. TGTForMyVpc)&rarr;select protocol TCP and port 8080&rarr;select      Vpc(MyVpc)&rarr;select health check protocol as TCP&rarr;click on Next&rarr;To register target instances select instances give port and&rarr;click on include all      pendings and&rarr;click on create Target group Click on Load Balancer&rarr;select type Network Load Balancer&rarr;click on create&rarr;name                            (e.g.NLBForMyVpc)&rarr;select scheme (e.g. Internal)&rarr;Ip address Type (e.g. IPV4)&rarr;select VPC (e.g. MyVpc)&rarr;select availability zone&rarr;select public    subnets&rarr;select IPV4 address (Assigned by AWS)&rarr;select protocol TCP with port 8080&rarr;forward it to Target group(e.g. TGTForMyVpc)&rarr;click on create      load Balancer.

10. Create a Lambda function i.e., InputTransformation and deploy (upload Jar), which transforms raw input json payload to as/400 compatible format and invokes the Program call API with this converted payload
11. Create AWS Gateway API and import the swagger collection which represents all the AS400 API Interfaces.

    Search and click on Gateway API from AWS services dashboard, shows Gateway API dashboard
    Create new API by clicking on API link on the left side menu, leave some name and continue
    Resource Importing – Swagger collection into API
    Click on Resources \&gt; Actions picklist, then click on Import API under API Actions
    Next copy and paste the Swagger collections into the text area, and then click on import
    If everything seems to be ok then we can the below image
    Map API Gateway Interfaces with Service API interfaces and Lambda functions using HTTP and VPCLink.
    All the above Gateway API can be tested with in the configuration editor. Please refer this sample-test-case-payloads.txt for test payloads.
    Once the mapping configuration is done, then Create a Test Stage environment where the API gets deployed.
    click on Action button&rarr;click on Deploy API&rarr;provide name to create stage&rarr;click on Deploy button.
    Testing API Gateway Interfaces in two ways
        1. From within the Gateway API
        2. Externally
    

![image](https://user-images.githubusercontent.com/88314020/165258740-72c9a19d-9108-414c-810e-139fd6dac32c.png)



1. The following table depicts Gateway API interface mapping to AS400 Connector Service API which is hosted onto a EC2 private instance

| API Interface Name | Integration Type | Use Proxy Integration | Method | VPC Link | Endpoint URL|
| --- | --- | --- | --- | --- | --- |
| /connections | VPC Link | Keep it deselected | POST | Provide VPC link name. In the current implementation, it named as as400-vpclink | [http://x.x.x.xx:8080/connections](http://x.x.x.xx:8080/connections) |
| /connections/{connection-name} | VPC Link | Keep it deselected | DELETE | Provide VPC link name. In the current implementation, it named as as400-vpclink | [http://x.x.x.xx:8080/connections/{connection-name}](http://x.x.x.xx:8080/connections/%7Bconnection-name%7D) |
| /connections/{connection-name} | VPC Link | Keep it deselected | GET | Provide VPC link name. In the current implementation, it named as as400-vpclink | [http://x.x.x.xx:8080/connections/{connection-name}](http://10.0.1.233:8080/connections/%7Bconnection-name%7D) |
| /connections/{connection-name} | VPC Link | Keep it deselected | PUT | Provide VPC link name. In the current implementation, it named as as400-vpclink | [http://x.x.x.xx:8080/connections/{connection-name}](http://10.0.1.233:8080/connections/%7Bconnection-name%7D) |
| /connections/{connection-name}/command | VPC Link | Keep it deselected | POST | Provide VPC link name. In the current implementation, it named as as400-vpclink | [http://x.x.x.xx:8080/connections/{connection-name}/command-call](http://10.0.1.233:8080/connections/%7Bconnection-name%7D/command-call) |
| /connections/{connection-name}/data-queue/{library-name}/{data-queue-name} | VPC Link | Keep it deselected | GET | Provide VPC link name. In the current implementation, it named as as400-vpclink | [http://x.x.x.xx:8080/connections/{connection-name}/data-queue/{library-name}/{data-queue-name}](http://10.0.1.233:8080/connections/%7Bconnection-name%7D/data-queue/%7Blibrary-name%7D/%7Bdata-queue-name%7D) |
| /connections/{connection-name}/data-queue/{library-name}/{data-queue-name} | VPC Link | Keep it deselected | POST | Provide VPC link name. In the current implementation, it named as as400-vpclink | [http://x.x.x.xx:8080/connections/{connection-name}/data-queue/{library-name}/{data-queue-name}](http://10.0.1.233:8080/connections/%7Bconnection-name%7D/data-queue/%7Blibrary-name%7D/%7Bdata-queue-name%7D) |
| /connections/{connection-name}/invoke-program-call/{library-name}/{program-name} | Lambda | -na- | POST | InputTransformation | Lambda Region: us-east-2Name of the Lambda function: InputTransformation |
| connections/{connection-name}/program-call/{library-name}/{program-name} | VPC Link | Keep it deselected | POST | Provide VPC link name. In the current implementation, it named as as400-vpclink | [http://x.x.x.xx:8080/connections/{connection-name}/program-call/{library-name}/{program-name}](http://10.0.1.233:8080/connections/%7Bconnection-name%7D/program-call/%7Blibrary-name%7D/%7Bprogram-name%7D) |
| /connections/{connection-name}/close | VPC Link | Keep it deselected | POST | Provide VPC link name. In the current implementation, it named as as400-vpclink |
[http://x.x.x.xx:8080/connections/{connection-name}/close](http://x.x.x.xx:8080/connections/%7Bconnection-name%7D/close) |
| /connections/{connection-name}/reopen | VPC Link | Keep it deselected | POST | Provide VPC link name. In the current implementation, it named as as400-vpclink |
[http://x.x.x.xx:8080/connections/{connection-name}/reopen](http://x.x.x.xx:8080/connections/%7Bconnection-name%7D/reopen) |

**Note:** Here x.x.x.xx is the IP address of the EC2 private instance, 8080 is the port on which AS400 Connector Service API is running.

1. The following table depicts Gateway API interface mapping to AS400 Connector Service API which is hosted onto a EC2 public instance

| API Interface Name | Integration Type | Use Proxy Integration | Method | Endpoint URL|
| --- | --- | --- | --- | --- |
| /connections | HTTP | Keep it deselected | POST | [http://x.x.x.xxx:8080/connections](http://x.x.x.xxx:8080/connections) |
| /connections/{connection-name} | HTTP | Keep it deselected | DELETE | [http://x.x.x.xxx:8080/connections/{connection-name}](http://10.0.1.233:8080/connections/%7Bconnection-name%7D) |
| /connections/{connection-name} | HTTP | Keep it deselected | GET | [http://x.x.x.xxx:8080/connections/{connection-name}](http://10.0.1.233:8080/connections/%7Bconnection-name%7D) |
| /connections/{connection-name} | HTTP | Keep it deselected | PUT | [http://x.x.x.xxx:8080/connections/{connection-name}](http://10.0.1.233:8080/connections/%7Bconnection-name%7D) |
| /connections/{connection-name}/command | HTTP | Keep it deselected | POST | [http://x.x.x.xxx:8080/connections/{connection-name}/command-call](http://10.0.1.233:8080/connections/%7Bconnection-name%7D/command-call) |
| /connections/{connection-name}/data-queue/{library-name}/{data-queue-name} | HTTP | Keep it deselected | GET | [http://x.x.x.xxx:8080/connections/{connection-name}/data-queue/{library-name}/{data-queue-name}](http://10.0.1.233:8080/connections/%7Bconnection-name%7D/data-queue/%7Blibrary-name%7D/%7Bdata-queue-name%7D) |
| /connections/{connection-name}/data-queue/{library-name}/{data-queue-name} | HTTP | Keep it deselected | POST | [http://x.x.x.xxx:8080/connections/{connection-name}/data-queue/{library-name}/{data-queue-name}](http://10.0.1.233:8080/connections/%7Bconnection-name%7D/data-queue/%7Blibrary-name%7D/%7Bdata-queue-name%7D) |
| /connections/{connection-name}/invoke-program-call/{library-name}/{program-name} | Lambda | -na- | POST | Lambda Region: us-east-2Name of the Lambda function: InputTransformation |
| connections/{connection-name}/program-call/{library-name}/{program-name} | HTTP | Keep it deselected | POST | [http://x.x.x.xxx:8080/connections/{connection-name}/program-call/{library-name}/{program-name}](http://10.0.1.233:8080/connections/%7Bconnection-name%7D/program-call/%7Blibrary-name%7D/%7Bprogram-name%7D) |
| /connections/{connection-name}/close | HTTP | Keep it deselected | POST |
http:// x.x.x.xxx:8080/connections/{connection-name}/close |
| /connections/{connection-name}/reopen | HTTP | Keep it deselected | POST | http:// x.x.x.xxx:8080/connections/{connection-name}/reopen |

**Note:** Here x.x.x.xxx is the IP address of the EC2 public instance, 8080 is the port on which AS400 Connector Service API is running.


1. Create an SNS topic for publishing the received DTAQ events from the DTAQ
2. Create a Lambda function i.e. DQSNSEventProcessor, deploy (upload Jar) and subscribe to SNS topic


**Testing existing solution:**

The following text talks about the general approach to be followed for testing the existing solution.

1. Login / Sig into AWS Management Console
2. Choose region from picklist as US East (Ohio)us-east-2
3. **Managing EC2 Instances**
  1. On successful sign in, Search and click on EC2 instances from AWS services dashboard, shows EC2 services dashboard
  2. Click on Instances under instances from the left side menu
  3. Start the below instances one after the other through by selecting check mark next to instance name and then click on Instance Start from the Instance State pick list
    1. as400gateway – is an EC2 instance and acts as bastion / NAT gateway host, which is part of public subnet under VPC. By default, application starts running on port 8080.
    2. CommonAPIServer – is an Ec2 instance and hosts the AS400 connector API interfaces.
4. **Managing Service Configuration**
  1. Application properties can be changed from the file present in this location /opt/as400-common-api/config/application-dev.properties
5. **Managing AWS API Gateway Interface**
  1. Search and click on API Gateway from within the AWS console, API Gateway Dashboard gets displayed
  2. Click check box next to &quot;AS400 Common API&quot;, Shows API interfaces info in a tree structure format where lot of insights can be drawn in terms of API specification, configuration, and testing
  3. Currently API interfaces are deployed in the Test Stage environment. To get Test stage environment info, click on Stages in the left side menu. Here site URL info is available.

Ex. [https://46oht9t3f8.execute-api.us-east-2.amazonaws.com/test](https://46oht9t3f8.execute-api.us-east-2.amazonaws.com/test)

1. **Logs Verification**
  1. CommonAPIServer logs can be verified from this location /var/log/as400-common-api-service.log
  2. Lambda function logs
    1. Search and click on Lambda function from within the AWS console, Lambda dashboard gets displayed
    2. Click on Functions and can see the below available functions
      1. InputTransformation: Transforms Input json payload to as/400 compatible format
      2. DQSNSEventProcessor: AS/400 DTA Queue SNS Lambda Integration
    3. To view logs pertaining to Lambda \&gt; Functions \&gt; DQSNSEventProcessor
      1. Click on DQSNSEventProcessor, Displays dashboard with configuration, Permission and Monitoring
      2. Click on Monitoring tab, it shows Monitoring dashboard along with the CloudWatch metrics. Click on &quot;View logs in CloudWatch&quot;. This is the path where logs can be found CloudWatch \&gt; CloudWatch Logs \&gt;

Log groups \&gt; /aws/lambda/DQSNSEventProcessor

1. **Managing Application Service**

By using tool Putty SSH login to the public EC2 instance Ex.AS400Gateway and then use below command to login to private EC2 instance. **ssh ec2-user@x.x.x.xx**

where x.x.x.xx represents IP address of private EC2 instance where the actual service is hosted.

  1. To know the application status: sudo service as400-common-api-service status
  2. To stop the application: sudo service as400-common-api-service stop
  3. To start application: sudo service as400-common-api-service start

**License Management:**

The IBM i connector requires a license file &quot;as400-license.lic&quot; from Infoview to enable access to specific IBM i system(s).

Managing license in different ways by using different protocols such as S3, HTTP/HTTPS, FTP, FILE, SMB etc. and accessing it through these protocols in our application needs to configure required properties in **application-dev.properties** file.

Available Protocols to load license file/truststore file (HTTP,HTTPS, FTP, SMB, S3, FILE, CLASSPATH_)_

what protocol used to load license file/truststore file that need to be configured as below in application-dev.properties file as below.

licenseFileProtocol=S3

Truststore file is used to establish the secure connection with IBM i AS400 system. if secure connection property set as true then needs to configure truststore file protocol in application-dev.properties file as below.

truststoreFileProtocol=S3

Following table contains the properties related to protocols requires to be configure:

| **#** | **Protocol Name** | **Properties** |
| --- | --- | --- |
| 1 | S3 | s3.bucket=\&lt;path-to-bucket\&gt;<br>s3.region=us-east-2</br><br>s3.accessKey=ENC(\&lt;encrypted-access-key\&gt;)</br><br>s3.secretKey=ENC(\&lt;encrypted-secret-key\&gt;)|
| 2 | HTTP/HTTPS | http.url=\&lt;url-URL\&gt;<br>http.dir.path=\&lt;license-file-path\&gt;</br><br>http.username=ENC(\&lt;encrypted-user-name\&gt;)</br>http.password=ENC(\&lt;encrypted-pwd\&gt;)
 |
| 3 | FTP | ftp.host=\&lt;ftp-host\&gt;<br>ftp.dir.path=\&lt;path\&gt;</br><br>ftp.username=ENC(\&lt;encrypted-user-name\&gt;)</br><br>ftp.password=ENC(\&lt;encrypted-pwd\&gt; ) </br>|
| 4 | FILE/SMB | file.Path=\&lt;path-to-license-file\&gt;
 |
