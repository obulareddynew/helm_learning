# Amazon S3(Simple Storage Service)

It stores the Object(data+metadata(metadata is a set of name value pairs that describe the object)) 
- S3 buckets are containers for objects stored in S3.
- An Object is uniquely identified within a bucket by a key.
- An Image file is uploaded to a uniquely named S3 bucket after file uploaded it becomes an object which contains image data and any added metadata such as time created and location.
- Amazon S3 offers industry leading scalability, data availability, security and performance.
- Customers of all sizes and industries can use it to store and protect any amount of data for a range of use cases such as websites, mobile applications, backup and restore, archive, enterprise applications, IOT devices, and big data analytics.
- Amazon S3 can easily meet fluctuating demands by automatically scaling your storage resources up and down without upfront investments or resoure procurement cycles.
- Amazon S3 is designed for 11 9s of data durability because it automatically creates and stores copies of all S3 objects across multiple availability zones.
- Your data is available when needed and protected against failures, errors and threats.
- Amazon S3 also offers leading security, compliance and audit capabilities.it secures data from unauthorized access with encryption features and access management tools.
- S3 allows you to block public access to all of your objects at the bucket or the account level with S3 block public access.
- You should enable block public access for all accounts and buckets that you do not want publicly accessible S3 maintains compliance programs such as PCI-DSS , HIPAA/HITECH, FedRAMP, EU Data protection Directive, and FISMA.
- To help you meet regulatory requirements, AWS also supports numerous auditing capabilities to monitor access requests to your S3 resources.
- Amazon S3 is widely supported and integarted with many applications and services.
- You can choose to work with a partner from the AWS partner network or APN which is the largest community of technology and consulting service providers.
- Migration partners can help you to transfer data to Amazon S3.
- Storage partners offer integarated solutions for primary storage, backup and restore, archives, and disaster recovery.
- You can also purchase an AWS integarted solution directly from the AWS MarketPlace which lists over 250 storage-specific offerings.

### Amazon S3 More Features:

Amazon S3 also provides a wide range of storage classes. 
- **S3 Standard** : S3 standard is for general purpose storage of frequenctly accessed data it delivers low latency and high throughput and can access data in milliseconds.
- **S3 Intelligent tiering** : S3 intelligent tiering is for data with unknown or changing access patterns. It automatically moves your data between a frequent access and an infrequent access tier based on changing access patterns - without performance impact or operational overhead.
- **S3 standard infrequent access, or IA**: S3 standard IA and s3 one zone IA are for long lived but less frequently accessed data.
- **S3 one zone infrequent access, or S3 one zone IA**: S3 standard IA and s3 one zone IA are for long lived but less frequently accessed data. Since S3 one zone infrequent access only stores data in one availability zone, it fits best for data that can be recreated.
- **and the three Amazon S3 Glacier storage classes**: S# Glacier storage classes are for long term archive and digital preservation. S3 Intelligent Tiering, S3 standard IA and S3 one zone IA also have the same low latency and high throughput performance as S3 standard. S3 glacier instant retieval storage class delivers the lowest cost storage with milliseconds retrieval. It is designed for archive data that needs immediate access.For archive data that does not require immediate access, choose S3 Glacier flexible retrieval, with retrieval in minutes, or free bulk retrievals in 5 to 12 hours. To save even more on long lived archive storage, choose S3 glacier Deep archive, the lowest cost storage, with data retrieval from 12 to 48 hours.
     - S3 Glacier Instant Retrieval,
     - S3 Glacier Flexible Retrieval,
     - S3 Glacier Deep Archive
-  Easch S3 storage class supports a specific data access level
-  with wide range of storage classes offered by S3. You can choose which ones best fits your data profile and use cases.
-  To clasify, manage and report on your data, amazon S3 also provides easy to use management tools for granular data control.
-  You can use S3 storage class analysis to discover data that should move to a lower cost storage class based on access patterns.
-  Then you can use this information to configure an S3 lifecycle policy that makes the data transfer.
-  S3 lifecycle policy defines when objects transition to another storage class and when objects expire. It helps to manage your objects so that they are stored cost effectively throughout their life cycle.
-  In addition, you can use S3 cross-region replication to replicate data into other AWS regions. You can also replicate data within the same AWS region.
-  S3 Object Lock. Objects are locked for the duration of the retention period that you define and are prptected from deletion.
-  You can enforce, write-once, read-many policies with S3 object lock.
-  S3 Inventory report lists your stored objects, their metadata and encryption status.
-  You can also use S3 batch operations to change object properties and perform storage management tasks for billions of objects.
-  With S3 batch operations and a single S3 API request, or a few clicks in the Amazon S3 management console, you can perform batch operations such as, copy objects between buckets, replace object tag sets, modify access controls and restore archived objects from Amazon S3 Glacier.
-  Since Amazon S3 works with AWS Lambda, you can log activities, definie alerts and automate workflows without managing additional infrastruture.
-  With query in-place services, you can run big data analytices directly across you S3 objects and other data sets in AWS. This means you can run big data analytics directly on your data stored in Amazon S3 that allows you to query data without needing to copy and load it into a seperate analytics platform or data warehouse.
-  Amazon S3 is also compatible with AWS analytics services, Amazon Athena and Amazon redshift spectrum.
-  You can use Amazon Athena to query S3 data with standard SQL expressions.
-  Amazon Redshift spectrum also runs SQL queries directly against data at rest in Amazon S3, and is more appropriate for complex queries and large data sets up to exabytes.
-  You can also use S3 select to retrive subsets of object data instead of the entire object, which can be up to five terabytes in size. it is designed to increase query performance by up to 400 percent and reduce querying costs as much as 80%.
-  Versioning is means of keeping multiple variants of an object in the same bucket.
-  Buckets can be in one of three states: un-versioned, whick is the default, versioning enabled, or versioning suspended.
-  When versioning is suspended it will suspend the creation of version ID for new objects but preserve all the existing object versions.
-  Once you enable versioning the combination of a bucket key and version ID uniquely identify each object.
-  After the bucket is version enabled, it can never return to an un-versioned state. you can, howver, suspend versioning on that bucket.
-  You can use versioning to preserve, retrive and restore every version of every object stored in your Amazon S3 bucket.
-  Versioning enabled buckets enable you to recover objects from accidental deletion or overwrite.
-  If you delete an object it is not permanently deleted, Amazon S3 inserts a delete marker.
-  If you overwrite an object, it results in a new object version in the bucket. you can restore the previous version as long as the object has not been deleted.

 #### More Information:
- When Configured for a static website hosting, the S3 bucket receives a dedicated URL Requests to this URL promt Amazon S3 to serve the buckets designated root object(typically the main HTML file)
- Access tp the S3 bucket and its content is controlled through permissions, which are defined in a bucket policy.
- A bucket policy, written in JSON format, specifies who can access the bucket and what operations they can perform.

### Amazon S3 Access Management:
- By default all Amazon S3 resources, such as buckets and objects are private.
- Only resource owner an AWS account that created it, can access the resource.
- The resource owner can optionally grant access. or permissions to others by writing an access policy
- Amazon S3 offers access policy options that can be broadly categorized as **resource-based policies and user policies**.
- **Resource based policies** are access policies that you attach to your resources such as buckets and objects Bucket policies and access control lists, or ACLs(access Control Lists) are resource-based policies, you can also authenticate certain types of data access using query-string authentication to grant temporary access to others with time limited URLs.
- **User policies** are access policies that you attach to users. Access can be granted to users using AWS IAM policies.
- You may choose to use resource-based policies, user policies, or some combination of these to manage permissions to your Amazon S3 resources.
- **Access Control list** is a legacy access control mechanism as a general rule. AWS recommends using S3 bucket policies or IAM policies for access control.
- When Amazon S3 receives a request it must evaluate all the access policies to determine whether to authorize or deny the request.

 #### Resource Based policies:
 **Access Control Lists**
 - Each bucket and object has an access control list. an ACL associated with it.
 - Access control lists use Amazon S3-specific-XML schemaIt defines which AWS accounts or groups are granted access and the type of access they have.
 - When you create a bucket or an object, Amazon S3 creates a default ACL. that grants the resource owner full control over the resource.
 - You can use access control lists to grant basic read or write permissions to other AWS accounts.
 - A Bucket policy, written in JSON, is attached to an S3 bucket.It provides access permissions to the S3 bucket and all objects in it.
 - Here is an example of an S3 bucket policy this S3 bucket policy allows the principal the root account, and the IAM user Alice under that account to perform any S3 operations.The wildcard astrisk character after the S3 in the action statement indicates that all S3 operations are allowed. Particularly, you can perform S3 operations only on specific resources as shown in the resource statement.In this case, only the S3 bucket with name  my_bucket and all objects in it are allowed As you can see, you can use the slash and asterisk after the bucket name to indicate all objects in the bucket.

**User Policies**

- User policies control who has access to your Amazon S3 resources and is written in JSON. You can use AWS Identity and Access Management, or AWS IAM to create users, groups, and roles in your account and attach access policies to grant them access to Amazon S3.
- Lets look at an IAM policy example unlike bucket policies , IAM policy does not require a principal element because principal is by default the entity that the IAM policy is attached to that first part of the policy specifies bucket-level permissions It allows the ListBucket action so that applications. can list all objects in the test bucket. the second part specifies object level permissions It allows PutObject, GetObject, and DeleteObject actions so that applications can write, read and delete any objects in the test bucket.


 #### More Information:
 - JSON is a standardized data format that is both human readable and machine readable, widely used across AWS services and applications.
 - When city residents access the web portal for beach wave information, their browsers send GET requests to the static webpages URL, which serves the index.html root object.
 - The root object can be renamed from index.html to waves.html with S# bucket settings updated accordinly to reference the new filename.


# EC2 Instances

## AWS Global infrastructure Overview:

- The AWS Global Infrastructure delivers cloud solutions that companies depend on, no matter their size, changing needs or challenges.
- The infrastructure is designed and built to deliver flexible, relaiable, scalable and secure cloud computing environment with high quality global network performance.
- AWS provides an extensive global footprint, helping make sure your worldwide customers are served. you can rapidly expand operations to virtually any geographic region or country.
- As of 2025, AWS now spans over 100 availability zones within over 35 AWS regions around the world.
- The footprint is continuously expanding with announced plans for even more availability zones and regions.
- This expansion is paired with commitment to continue using renewable energy sources.
- The AWS global infrastructure consists of regions and availability zones or AZs. AWS cloud computing resources are hosted in multiple world wide locations referred to as AWS regions. Each region is a seperate geographic area and has multiple isolated locations known as availability zones.
- A region has at least two AZs. Most have three,including all new regions, and several have as many as six.
- AZs are fully isolated and are many kilometers apart from each other for complete redundancy.
- Each AZ consists of one or more discrete data centers, typically three.at full scale these data centers can include hundreds of thousands of servers.
- To help provide maximum reliability,each data center is housed in separate facilities with redundant power and dedicated metro fiber connectivity.
- AWS network each AZ data center and AWS region is interconnected through a purpose-built, highly available, low latency private global network with a fully redundant parallel fiber network.
- Points of Presence(PoP) with its fully managed Amazon CloudFront service.
- AWS provides a global content delivery network or CDN that you can use to securely deliver data to end users.
- CloudFront provides low latency and high transfer speeds through a world wide network of global point of presence, or POP, locations. This network consists of edge locations and regional Edge cache servers, partioning, Azs help with partitionaning which helps protect your application from natural disasters such as lightning strikes, tornadoes and earthquakes.
- For example you might deploy an application across multiple Azs and use a load balancer to distribute traffic across them. if one AZ experience an outage, traffic is routed to another AZ, keeping the application up and running.

### Amazon EC2 Overview:

Amazon Elastic Compute Cloud(EC2):
- Amazon Elastic Compute Cloud - or EC2 - is a web service that provides secure, resizable compute capacity in the cloud. It eliminates your need for up-front hardware investment so you can develop and deploy applications faster.
- You can use Amazon EC2 to launch as many or as few virtual servers as you need.
- Amazon EC2 enables you to scale up or down within minutes to handle changes or spikes in requirements, reducing your need to forecast traffic.
- It provides you with complete control of your computing resources and lets you run on Amazon's proven, reliable, and scalable computing environment.
- Amazon EC2 is hosted in multiple locations world wide comprised of AWS regions, which consistes of atleast 3 availability zones.
- This computing environment provides high availability, with service-level agreement, SLA, commitment, of 99.99% availability for each Amazon EC2 region.
- In addition, security at AWS is our top priority. AWS is built to meet the requirements of the most security sensitive organizations.

##### First, lets go through some of the Amazon EC2 basics.
**What is an Instance**
- An instance is a virtual server in the cloud .
- Amazon EC2 provides a wide selection of instance types to enable you to choose the CPU, memory, storage, networking capacity, and GPU for your applications.
- Here are the instance type categories
     - **General Purpose** instances are instances, that provide a balance, of compute, memory, and networking resources. Eg: Web Server
     - **Compute Optimized** instances are optimized for compute intensive workloads. Eg: Game
     - **Memory Optimized Instances** These instances are for memory-intensive workloads that process large data sets in memory: Eg: Big Data Analytics
     - **Storage Optimized Instances** They are designed for workloads that require sequencial read and write access to very large data sets on local storage. Eg: Data Warehousing
     - **Accelerated computing instances** These are GPU instances that use hardware accelerators. Eg: Machine or Deep Learning
     - **HPC optimized instances** These instances are optimized for high performance computing workloads. Eg: complex Simulations.
- above instance types are optimized to fit different use cases. given just one of the many examples for each instance types.
- In addition, each instance type includes one or more instance sizes, allowing you to scale your resources to the requirements of your target workload.

**AMI(Amazon Machine Image)**:
- An AMI provides the information required to launch an instance. It is a tempplate that contains a software configuration such as an operating system, applications, and an application server.
- You must specify an AMI, When you launch an instance.
- You can kaunch a single instance or multiple instances from an AMI.

### AWS Global Infrastructure Benefits:
- AWS Global Infrastrure benefits:
     - Performance(High Quality and uninterrupted performance)
          - It offers customers a high performing low latency, high scalable, high avaialibilty, secure cloud infrastructure and responds to changing needs fast without any drop in performance. AWS provides customers with highest levels of availability and reliability.
          - AWS availability zones are designed for physical redundancy and provide resilience enabiling uninterrupted performance with no signle points of failure. Every component of the AWS infrastructure is designed and built for redundancy and reliability from regions to networking links to load balancers to routers and firmware.
          - In addition, The AWS Global oInfrastrure is built on Amazon's own custom hardware such as custom purpose built compute servers, load balancers, routers, and silicon.
          - AWS optimizes this hardware for only one set of requirements - workloads run by AWS customers.
          - By using custom hardware, AWS provides customers with the highest levels of reliability, the fastest pace of innovation all at the lowest possible cost.
     - Security:
          - our core infrastrure is designed to meet the most stringent security requirements in the world and is built to satisfy the security requirements for military, global banks, and other high sensitivity organizations.
          - our infrastructure is monitored 24/7 to help ensure the confidentiality, integritry and availability of our customers data.
     - Scalability:
          - The AWS Global Infrastructure enables companies to be extremely flexible and take advantage of the conceptually infinite scalability of the cloud.
          - Customers can provision the amount if resources that they actuvally need knowing they can instantly scale up or down along with the needs of their business.
          - Companies can quickly spin up resources as they need them, deploying hundreds or even thousands of servers in minutes.
     - Low Cost
          - With the industryts most extensive data center footprint, the AWS Global infrastructure enables more customers to benfit from cloud economics of scale and reduce the total cost of ownership(T.C.O) of their overall I.T Infrastructure.

## Amazon Elastic Block Store(EBS):
- is a service that provides block-level storage volumes for use with Amazon Elastic Compute Cloud, or EC2 Instances.
- EBS volumes behave like raw, unformatted block devices in the cloud.
- You can mount EBS volumnes as a device on your EC2 instance.
- Each EBS volume can be attached to only one instance at a time but multiple valumes of EBS can be on the same instance.
- Since EBS volumes are mounted to the instances, they can provide single-digit milliseconds latency, which is extremely low latency between where the data is stored and where it might be used on the instance.
- EBS volumes are designed for both throughtput and transaction intensive workloads at any scale.
- Amazon EBS offers data persistence. All the data on the attached EBS volume remains available even after you stop or terminate an Amazon EC2 instance.
- Amazon EBS provides two major storage volume types.
     - **Solid State drive or SSD-backed storage**: is optimized for transactional workloads, which involve frequent read/write operations with small I/O size. Some use case examples include databases and boot volumes.SSD-backed volumes include the highest-performance provisioned IOPS SSD, io2 and io1, for latency-sensitive transactional workloads. general-purpose SSD volumes, gp3 and gp2 balance price and performance for a wide variety of transactional data. gp3 volumes enable customers to provision performance independent of storage capacity. The dominet performance for SSD-backed volumes depends primarily on IOPS (input/output operations per sec)
     - **Hard disk drive or HDD backed storage**: HDD-backed storage is for throughput-intensive workloads such as MapReduce,data warehouse, and log proicessing . HDD-backed volumes include throughout-optimized HDD. st1, for frequenctly accessed throughput-intensive workloads. The lowest cost cold HDD, sc1, is for less frequently accessed data. Dominent performance for HDD-backed volumes depends primarily on throughput measured in megabytes per second.

 ## DNS (Domain Name System):
- Welcome to an overview of DNS, or Domain Name System
- Computers on the internet commi=unicate with one another by using numbers known as IP addresses. When you visit a website you don't have to remember a long number, instead, you can enter a domain name like www.example.com that is easier to remember. A DNS service such as **Amazon Route 53** is globally distributed service that translates human readable names like www.example.cominto a numeric addresses like 192.0.2.1 that computers use to connect to each other.
- The interner's DNS (system) works much like a phone book by managing the mapping between names and numbers.
- DNS servers translate requests for names into IP addresses, controlling which server an end user will reach when they type a domain name into their web server. Tese requests are called queries. Let's take a look at how DNS queries work
     - First, the host sends a DNS request to the local DNS server.
     - Second, the DNS server which is preconfigured with the list if root name servers, randomly selects one root name server for the "A" record www.example.com
     - An "A" record maps a domain name like example.com to an IP address of the computer that hosts the domain.
     - Third, the root name server responds with a list of authoritative name servers for the .com zone, along with a list of the IP addresses.
     - Fourth, the DNS server randomly selects one of the authoritative name servers returned in step three and sends another DNS query for the "A" record example.com to the top-level somain server.
     - Fifth, the top-level domain name server responds with a list if name servers. that are authoritative for the domain example.com.
     - Sixth, the DNS server randomly selects ione of the authoritative name servers returned in step five and sends another DNS query for the "A" record example.com
     - Seventh, since the name server receiving the query in step six is authoritative for the domain example.com, the name server responds to the DNS server with the IP address value of the "A" record for example.com.
     - And for the final step, the DNS server sends the DNS response with the IP address back to the host, which enables the host computer to connect to www.example.com

## AWS support overview
- AWS provides multiple tiers of support to help customers succeed with their AWS implementations.
- The support plans include basic, which is free and automatic for all customers, Developer, Business, Enterprise On-Ramp, and Enterprise.
- The Basic plan offers customer service for account and billing questions, support forums, service health checks and documentation.
- The other paid plans provide all of these options plus unlimited technical support cases with monthly pricing and no long-term contracts.

#### Technical resources and documentation overview 
- Numerous other AWS technical resources are available across different environments and repositories. You can use the AWS documenation website for find helpful resources, such as user guides, whitepapers, tutorials, code samples, software developemnt kits and references.
- AWS perscriptive Gidance provides time-tested strategies, guides, and patterns to help accelerate your cloud migration, modernization and optimization projects.
- These resources were developed by AWS technology experts and the global community of AWS partners, based on their years of experience helping customers realize their business objectives on AWS.
- AWS re:Post provides several types of knowledge resources, such as knowledge center articles, community articles, and questions and answers. The AWS Trust and Safety Center provides information on how to report activity or content on AWS that you suspect is abusive, how to address an abuse notice that you receive from AWS Trust anhd Safety ,
- AWS services that you can use to protect your applicaions and best practices for digital messaging.

#### AWS Trusted Advisor:
- moniters AWS infrastructure services, identifies customer configurations, compares them to known best practices, and notifies customers about oportunities to save money, improve system performance or close security gaps.
- Trusted Advisor evaluates AWS accounts by using checks across five key categories, cost optimization, security, fault tolerance, performance and service limits
- Basic support and Developer support custometrs get access to 6 security checks and 50 service limkit checks, while Business support and Enterprise support customers have access to all 115 trusted Advisor checks. 
 


