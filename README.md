# AWS-Getting-started-with-Big-Data

In this project we have created a sample EMR(Elastic MapReduce) cluster using Amazon Web Service Console. The project is Basically "the Analysis of Log requests". For this project we have used sample data from Amazon CloudFront log files and also used sample Hive script to analyze the logs. 
The sample cluster that you create will be running in a live environment and you will be charged for the resources that you use. This tutorial should take an hour or less, so the charges that you incur should be minimal.

# Step 1
1. you must have AWS account. You can also create a account if you don't have one using the following link
http://aws.amazon.com/ and choose create account.
 
2. Create Amazon S3 bucket. 
S3-simple storage service. S3 is used to store out log files, output and we can also store inputs, scripts and we can store as much as data we want. To create a bucket, simple got to services from AWS console homePage and choose S3. Then create a bucket.Once you created the bucket you can enter the bucket and create folders like output and logs. For this project we are using only logs and output folders. Make sure the output folder is empty. 

3. Create EC2 KeyPairs 
EC2 is a virtual server on AWS to run applications. We must have keypairs for this project to connect to the nodes in your cluster over a secure channel using the Secure Shell (SSH) protocol.To create EC2, go to EC2 homePage and on the left panel, select Network and Security. From there, select keypairs and create. If you have already created keypairs then skip this step. Note: The keypairs which we download are the only copy and keep it in secure place.

4. Launch Sample Cluster
Sign in to the AWS Management Console and open the Amazon EMR console at https://console.aws.amazon.com/elasticmapreduce/.
  Choose Create cluster.
  On the Quick cluster configuration page, accept the default values except for the following fields:
  For S3 folder, choose the folder icon to select the path to the logs folder that you created in Create an Amazon S3 Bucket.
  For EC2 key pair, choose the key pair that you created in Create an Amazon EC2 Key Pair.
  Choose Create cluster.
  Proceed to the next step.

5. Sample Data and Scripts
For this project, we are taking sample data and hive scripts from AWS.The sample data is a series of Amazon CloudFront web distribution log files. The data is stored in Amazon S3 at s3://region.elasticmapreduce.samples where region is your region.

Each entry in the CloudFront log files provides details about a single user request in the following format:

  "2014-07-05 20:00:00 LHR3 4260 10.0.0.15 GET eabcd12345678.cloudfront.net /test-image-1.jpeg 200 -        Mozilla/5.0%20(MacOS;%20U;%20Windows%20NT%205.1;%20en-US;%20rv:1.9.0.9)%20Gecko/2009040821%20IE/3.0.9
  
The sample script calculates the total number of requests per operating system over a specified timeframe. The script uses HiveQL, which is a SQL-like scripting language for data warehousing and analysis. The script is stored in Amazon S3 at s3://region.elasticmapreduce.samples/cloudfront/code/Hive_CloudFront.q where region is your region.

The sample Hive script does the following:

  Creates a Hive table named cloudfront_logs.
  Reads the CloudFront log files from Amazon S3 using EMRFS and parses the CloudFront log files using the regular expression  serializer/deserializer (RegEx SerDe).
  Writes the parsed results to the Hive table cloudfront_logs.
  Submits a HiveQL query against the data to retrieve the total requests per operating system for a given time frame.
  Writes the query results to your Amazon S3 output bucket.
  The Hive code that creates the table looks like the following:
  CREATE EXTERNAL TABLE IF NOT EXISTS cloudfront_logs ( 
	Date Date, 
	Time STRING, 
	Location STRING, 
	Bytes INT, 
	RequestIP STRING, 
	Method STRING, 
	Host STRING, 
	Uri STRING, 
	Status INT, 
	Referrer STRING, 
	OS String, 
	Browser String, 
	BrowserVersion String 
)


  The Hive code that parses the log files using the RegEx SerDe looks like the following:

ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.RegexSerDe' 
WITH SERDEPROPERTIES ( "input.regex" = "^(?!#)([^ ]+)\\s+([^ ]+)\\s+([^ ]+)\\s+([^ ]+)\\s+([^ ]+)\\s+([^ ]+)\\s+([^ ]+)\\s+([^ ]+)\\s+([^ ]+)\\s+([^ ]+)\\s+[^\(]+[\(]([^\;]+).*\%20([^\/]+)[\/](.*)$" ) LOCATION 's3://us-west-2.elasticmapreduce.samples/cloudfront/data/';

The HiveQL query looks like the following:

SELECT os, COUNT(*) count FROM cloudfront_logs WHERE date BETWEEN '2014-07-05' AND '2014-08-05' GROUP BY os;

6. Process the sample Cluster
Use the Add Step option to submit your Hive script to the cluster using the console. The Hive script and sample data used by the script have been uploaded to Amazon S3 for you.

To submit your Hive script as a step

Open the Amazon EMR console at https://console.aws.amazon.com/elasticmapreduce/.

In Cluster List, select the name of your cluster.

Scroll to the Steps section and expand it, then choose Add step.

In the Add Step dialog:

For Step type, choose Hive program.
For Name, accept the default name (Hive program) or type a new name.
For Script S3 location, type s3://region.elasticmapreduce.samples/cloudfront/code/Hive_CloudFront.q
Replace region with your region. For example, for US West (Oregon) type s3://us-west-2.elasticmapreduce.samples/cloudfront/code/Hive_CloudFront.q
For Input S3 location, type s3://region.elasticmapreduce.samples
Replace region with your region. For example, for US West (Oregon) type s3://us-west-2.elasticmapreduce.samples
For Output S3 location, type or browse to the output bucket that you created in Create an Amazon S3 Bucket.
For Arguments, include the following argument to allow column names that are the same as reserved words:
-hiveconf hive.support.sql11.reserved.keywords=false
For Action on failure, accept the default option Continue.
Choose Add. The step appears in the console with a status of Pending.

The status of the step changes from Pending to Running to Completed as the step runs. To update the status, choose Refresh above the Actions column. The step runs in approximately 1 minute.

7. View results

After the step completes successfully, the query output produced by the Hive script is stored in the Amazon S3 output folder that you specified when you submitted the step.

To view the output of the Hive script
Open the Amazon S3 console at https://console.aws.amazon.com/s3/.

In the Amazon S3 console, select the bucket that you used for the output data; for example, s3://myemrbucket/
Select the output folder.

The query writes results into a separate folder. Choose os_requests.
The Hive query results are stored in a text file. To download the file, right-click it, choose Download, open the context (right-click) menu for Download, choose Save Link As, and then save the file to a suitable location.

Open the file using a text editor such as WordPad (Windows), TextEdit (Mac OS), or gEdit (Linux). In the output file, you should see the number of access requests by operating system.

Proceed to the next step.

8. Reset your environment

Delete the amazon S3 bucket.
Terminate the sample EMR cluster
  To terminate your Amazon EMR cluster
  Open the Amazon EMR console at https://console.aws.amazon.com/elasticmapreduce/.
  On the Cluster List page, select your cluster and choose Terminate.
    By default, clusters created using the console are launched with termination protection enabled, so you must disable it. In the           Terminate clusters dialog, for Termination protection, choose Change.
  Choose Off and then confirm the change.
  Choose Terminate.
