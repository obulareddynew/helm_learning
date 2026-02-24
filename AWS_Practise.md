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

-  
-  
