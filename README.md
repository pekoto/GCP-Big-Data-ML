# GCP-Big-Data-ML
Notes for the Google Cloud Platform Big Data and Machine Learning Fundamentals course.

https://www.coursera.org/learn/gcp-big-data-ml-fundamentals/home/welcome

https://console.cloud.google.com

## 1. Google Cloud Architecture

    -----------------------------------------
            2. Big Data/ML Products
    -----------------------------------------
     1. Compute Power | Storage | Networking
    -----------------------------------------
               0. Security
    -----------------------------------------


-2. Abstract away scaling, infrastructure

-1. Process, store, and deliver pipelines, ML models, etc.

-0. Auth

## 2. Creating a VM on Compute Engine

### 2.1 Setting up the VM

1. https://cloud.google.com
2. Compute --> Compute Engine --> VM Instances
3. Create a new VM

3.1 Allow full access to cloud APIs

3.2 Since we will access the VM through SSH, we don't need to allow HTTP or HTTPS traffic

4. Click SSH to connect

At this stage, the new VM has no software.

We can just install stuff like normal:

`sudo apt-get install git`

(APT: advanced package tool, package manager)

### 2.2 Sample earthquake data

After installing git, we can pull down the data from our repo:

`git clone https://www.github.com/GoogleCloudPlatform/training-data-analyst`

Course materials are in:

`training-data-analyst/courses/bdml_fundamentals`

Go to the Earthquake sample in `earthquakevm`.

The `ingest.sh` shell script contains the script to ingest data (`less ingest.sh`). This script basically just deletes existing data and then `wget`s a new CSV file containing the data.

Now, `transform.py` contains a Python script to parse the CSV file and create a PNG from it using `matplotlib` (
[matplotlib notes](https://nbviewer.jupyter.org/github/pekoto/MyJupyterNotes/blob/master/Python%20for%20Data%20Analysis.ipynb)).

([File details](https://github.com/GoogleCloudPlatform/datalab-samples/blob/master/basemap/earthquakes.ipynb))

Run `./install_missing.sh` to get the missing Python libraries, and run the scripts mentioned above to get the CSV and generate the image.

### 2.3 Transferring the data to bucket storage

Now, since we have generated the image. Let's get it off the VM and delete the VM.

To do this, we need to create some storage:
Storage > Storage > Browser > Create Bucket

Now, to view our bucket, we can use:

`gsutil ls gs://[bucketname]` (hence why bucket names need to be globally unique)

To copy out data to the bucket, we can use:

`gsutil cp earthquakes.* gs://[bucketname]`

### 2.4 Stopping/deleting the VM

So now we're finished with our VM, we can either:

__STOP__
Stop the machine. You will still pay for the disk, but not the processing power.

__DELETE__
Delete the machine. You won't pay for anything, but obviously you will lose all of the data.

### 2.5 Viewing the assets

Now, we want to make our assets in storage publically available.
To do this:
Select Files > Permissions > Add Members > `allUsers` > Grant `Storage Object Viewer` role.

Now, you can use the public link to [view your assets](https://storage.googleapis.com/earthquakeg/earthquakes.htm).

## 3. Storage

There are 4 storage types:

1. __Multiregional__: Optimized for geo-redundancy, end-user latency
2. __Regional__: High performance local access (common for data pipelines you want to run, rather than give global access)
3. __Nearline__: Data accessed less than once a month
4. __Coldline__: Data accessed less than once a year

## 4. Edge Network (networking)

Edge node receives the user's request and passes to the nearest Google data center.

Consider node types (Hadoop):

1. Master node: Controls which nodes perform which tasks. Most work is assigned to...
2. Worker node: Stores data and performs calculations
3. Edge node: Facilitate communication between users and master/worker nodes

__Edge computing__: brings data storage closer to the location where it's needed. In contrast to cloud computing, edge computing does decentralized computing at the edge of the network.

__Edge Node (aka. Google Global Cache - GGC)__: Points close to the user. Network operators and ISPs deploy Google servers inside their network. E.g., YouTube videos could be cached on these edge nodes.

__Edge Point of Prescence__: Locations where Google connects its network to the rest of the internet.

https://peering.google.com/#/

## 5. Security

Use Google IAM, etc. to provide security.

BigQuery data is encrypted.

## 6. Big Data Tool History

1. __GFS__: Data can be stored in a distributed fashion
2. __MapReduce__: Distributed processing, large scale processing across server clusters. Hadoop implementation
3. __BigTable__: Record high volume of data
4. __Dremel__: Breaks data into small chunks (shards) and compresses into columnar format, then uses query optimizer to process queries in parallel (service auto-manages data imbalances and scale)
5. __Colossus, TensorFlow__: And more...

## (Note on Columnar Databases)

Consider typical row-oriented databases. The data is stored in a row. This is great when you want to query multiple columns from a single row.

However, what if you want to get all of the data from all rows from a single column?

For this, we need to read every row, picking out just the columns we want. For example, we want the average age of all the people. This will be slower in row-oriented databases, because even an index on the age column wouldn't help. The DB will just do a sequential scan.

Instead we can store the data column-wise. How does it know which columns to join together into a single set? Each column has a link back to the row number. Or, in terms of implementation, each column could be stored in a separate file. Then, each column for a given row is stored at the same offset in a given file. When you scan a column and find the data you want, the rest of the data for that record will be at the same offset in the other files.


__Row oriented__
```
RowId 	EmpId 	Lastname 	Firstname 	Salary
001 	10 	    Smith 	    Joe      	40000
002 	12 	    Jones 	    Mary 	    50000
003 	11 	    Johnson 	Cathy 	    44000
004 	22 	    Jones 	    Bob 	    55000 
```

__Column oriented__
```
10:001,12:002,11:003,22:004;
Smith:001,Jones:002,Johnson:003,Jones:004;
Joe:001,Mary:002,Cathy:003,Bob:004;
40000:001,50000:002,44000:003,55000:004;
```

In terms of IO improvements, if you have 100 rows with 100 columns, it will be the difference between reading 100x2 vs. 100x100.

It also becomes easier to horizontally scale -- make a new file to store more of that column.

It is also easier to add a column -- just add a new file.

## Lab 1

__Public datasets__

Go to `BigQuery > Resources > Add Data > Explore public datasets` to add publically available datasets.

You can query the datasets using SQL syntax:

```
SELECT
  name, gender,
  SUM(number) AS total
FROM
  `bigquery-public-data.usa_names.usa_1910_2013`
GROUP BY
  name, gender
ORDER BY
  total DESC
LIMIT
  10
```

Before you run the query, the query validator in the bottom right will show much data is going to be run.

__Creating your own dataset__

Resources seem to have the following structure:

`Resources > Project > Dataset > Table`

To add your own dataset:

`Resources > click Project ID > Create dataset`
(For example babynames)

Then, we can add a table to this dataset.

`Click dataset name > Create table`

* Source > upload from file
* File format > CSV
* Table name > names_2014
* Schema > name:string,gender:string,count:integer

Creating the table will run a job. The table will be created after the job completes.

Click preview to see some of the data.

Then you can query the table like before.

SQL Syntax for BigQuery: https://cloud.google.com/bigquery/docs/reference/standard-sql/

## 7. GCP Approaches

Google Cloud Platform applications:

1. __Compute Engine__: Infrastructure as a service. Run virtual machines on demand in the cloud. 
2. __Google Kubernetes Engine (GKE)__: Clusters of machines running containers (code packages with dependencies)
3. __App Engine__: Platform as a service (PaaS). Run code in the cloud without worrying about infrastructure.
4. __Cloud Functions__: Serverless environment. Functions as a service (FaaS). Executes code in response to events.

App Engine: long-lived web applications that auto-scale to billions of users.

Cloud functions: Code triggered by event, such as new file uploaded.

__Serverless__

Although this can mean different things, typically it now means functions as a service (FaaS). I.e., application code is hosted by a third party, eliminating need for server and hardware management. Applications are broken into functions that are scaled automatically.

## 8. Recommendation Systems

A recommendation system requires 3 things:

* Data
* Model
* Training/serving infrastructure

_Point_: Train model on data, not rules.

__RankBrain__

ML applied to Google search. Rather than use heuristics (e.g., if in California and q='giants', show California giants), use previous data to train ML model and display results.

__Building a recommendation system__

1. Ingest existing data (input and output, e.g., user ratings of tagged data)
2. Train model to predict output (e.g., user rating)
3. Provide recommendation: rate all of the unrated products, show top n.

Ratings will be based on:

1. Who is this user like?
2. Is this a good product? (other ratings?)
3. Predicted rating = user-preference * item-quality

Models can typically be updated once a day or once a week. It does not need to be streaming.
Once computed, the recommendations can be stored in Cloud SQL.

So compute the ratings use a batch job (__Dataproc__), and store them in __Cloud SQL__.

__Storage systems__

Roughly:

1. __Cloud Storage__: Global file system _(unstructured)_
2. __Cloud SQL__: Relational database (transactional/relational data accessed through SQL) _(structured/transactional)_
2.1 __Cloud Spanner__: Horizontal scalability (+ more than a few GBs, need a few DBs)
3. __Datastore__: Transactional, NoSQL object-oriented database _(structured/transactional)_
4. __Bigtable__: High throughput, NoSQL, __append-only__ data (not transactional)  _(millisecond latency analytics)_
5. __BigQuery__: SQL data warehouse to power analytics _(seconds latency analytics)_

![image](https://raw.githubusercontent.com/pekoto/GCP-Big-Data-ML/master/images/storage-table.jpg)

__Hadoop Ecosystem__

1. __Hadoop__: MapReduce framework (HDFS)
2. __Pig__: Scripting language that can be compiled into MapReduce jobs
3. __Hive__: Data warehousing system and query language. Makes data on distributed file system look like an SQL DB.
4. __Spark__: Lets you run queries on your data. Also ML, etc.

__Storage__

HDFS is used for working storage -- storage during the processing of the job.
But all input and output data will be stored in Cloud Storage.
So because we store data off-cluster, the cluster only has to be available for a run of the job.

(Recall, you can shut down the compute nodes when not using them, so save your data in Cloud Storage, etc., instead of in the computer node disk.)

