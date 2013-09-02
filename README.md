# nutch-aws

## Overview
This project intends to document the pitfals and tricks of running Nutch in a AWS EMR Hadoop Cluster.


## Getting Started
### Get a Amazon Linux AMI Instance running on EC2
The first step to get one machine set up with the basic tools and configuration. I find that the simplest thing to do is to launch a t1.micro EC2 instance [Amazon Linux AMI](http://aws.amazon.com/amazon-linux-ami/) as it comes with most of the tools I need ([ami-cli](http://aws.amazon.com/cli/) among them), it is very easy to replicate and very unexpensive. I won't cover this step here as it is well documented on the web. (hint:The Amazon Linux AMI is the first choice in the Quick Start tab at the web based "Classic Wizard" EC2 launcher.) Make sure you have security rights to ssh to this machine. 

### Configuration
1. ssh to the Amazon Linux AMI instance, create a working folder, e.g., ~/nutch-aws, we will refer to it as NUTCH\_AWS\_HOME from now on.
1. scp your key-pair (.pem) file to this instance under NUTCH\_AWS\_HOME
1. ssh back to the Amazon Linux AMI instance
1. Yum install ant

		sudo yum install ant -y

1. Get the Makefile* from github into NUTCH\_AWS\_HOME

		wget https://raw.github.com/eleflow/nutch-aws/master/Makefile

1. Fill in the blanks in the Makefile

		ACCESS_KEY_ID = 					## YOUR ACCESS KEY ID
		SECRET_ACCESS_KEY = 				## YOUR ACCESS KEY SECRET
		EC2_KEY_NAME = 						## YOUR ACCESS KEY NAME
		KEYPATH	= ${HOME}/${EC2_KEY_NAME}.pem ## YOUR ACCESS KEY  FILE (IF IT"S DIFFERENT THAN ${HOME}/${EC2_KEY_NAME}.pem)
		S3_BUCKET = 						## THE S3 BUCKET WHERE FILES WILL BE READ FROM AND WRITTEN TO
		CLUSTERSIZE	= 3						## NUMBER OF MACHINES IN THE CLUSTER
		DEPTH = 3							## HOW MANY LINK HOPS WILL THE CRAWLER GO
		TOPN = 5							## HOW MANY OUTLINKS WILL BE FOLLOWED
		MASTER_INSTANCE_TYPE = m1.small 
		SLAVE_INSTANCE_TYPE = m1.small

1. Checking the configuration:

		make s3.list

	This should list the S3 buckets associated with your account if the configuration is correct

1. Create a NUTCH\_AWS\_HOME\urls\seed.txt file with the urls that will be a starting point to the crawler.

## Running

### Copying nutch job and seed files to S3

		make bootstrap

### Launching a cluster

		make create

This make target will do these:

1. download the Nutch 1.6 source code 
1. build the nutch 1.6 map reduce job jar.
1. copy the nutch 1.6 map reduce job jar to s3://S3_BUCKET/lib
1. copy the contents from the NUTCH\_AWS\_HOME\urls folder to s3://S3_BUCKET/url
1. start a emr cluster and run theese mr jobs:
	1. run the Nutch [crawl](http://wiki.apache.org/nutch/Crawl) job
	1. run the Nutch [mergesegs](http://wiki.apache.org/nutch/bin/nutch_mergesegs) job
	1. copy the crawldb, linkdb, and merged segments folders from hdfs://users/hadoop/crawl to s3://S3_BUCKET/crawl
1. copy the logs to s3://S3_BUCKET/logs

Note: the cluster is launched with "keep_job_flow_alive_when_no_steps" set to false which means it will be destroyed after the steps are completed. 

## Checking the master node

		make ssh

This will ssh into the master node and will give access to the [hadoop command line tool](http://hadoop.apache.org/docs/r1.0.4/commands_manual.html).

## Destroying 

		make destroy

This will kill any job that the cluster may be running and terminate the cluster.


[*]Yeah it's a Makefile. I based it on [Karan's](http://github.com/lila/SimpleEMR/blob/master/Makefile) and it may not the best tool for the job it was an easy way to get things rolling quick. 


