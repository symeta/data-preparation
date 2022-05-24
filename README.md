# data-preparation

# Add Date Column to table, and Date as Table Partition
The objective of this data transformation is to add a Data column to the original table which does not have a Date column initially.
And makes the Date field as the table partition key.

## Pre-requisite

Already have an athena table mapping to raw data stored in s3. A sample DDL of the source table is as below:

```sql
CREATE EXTERNAL TABLE `raw0515`(
  `datastatus` string COMMENT 'from deserializer', 
  `tradeindex` string COMMENT 'from deserializer', 
  `tradechan` string COMMENT 'from deserializer', 
  `securityid` string COMMENT 'from deserializer', 
  `tradtime` string COMMENT 'from deserializer', 
  `tradprice` string COMMENT 'from deserializer', 
  `tradvolume` string COMMENT 'from deserializer', 
  `trademoney` string COMMENT 'from deserializer', 
  `tradebuyno` string COMMENT 'from deserializer', 
  `tradesellno` string COMMENT 'from deserializer', 
  `tradebsflag` string COMMENT 'from deserializer', 
  `bizindex` string COMMENT 'from deserializer', 
  `localtime` string COMMENT 'from deserializer', 
  `seqno` string COMMENT 'from deserializer')
ROW FORMAT SERDE 
  'org.apache.hadoop.hive.serde2.OpenCSVSerde' 
WITH SERDEPROPERTIES ( 
  'separatorChar'=',') 
STORED AS INPUTFORMAT 
  'org.apache.hadoop.mapred.TextInputFormat' 
OUTPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION
  's3://msjy/raw0515/'
```

Set up an empty destination table which contains the Date column, and makes the Date field as the table partition key.
A sample DDL of the destination table is as below:

```sql
CREATE EXTERNAL TABLE destination0516 ( 
	DataStatus string, 
	TradeIndex string, 
	TradeChan string, 
	SecurityID string,
	TradTime string,
	TradPrice string,
	TradVolume string,
	TradeMoney string,
	TradeBuyNo string,
	TradeSellNo string,
	TradeBSFlag string,
	BizIndex string,
	LocalTime string,
	SeqNo string
) partitioned by (date string)
stored as parquet
LOCATION 's3://msjy/destination0516/';
```

## Define a data transformation job through Glue Console
The data transformation job aims at transforming the source table to the destination table by adding the Date column as well as the Date partition key.

### Open Glue console, and click the Jobs (legacy) tab

<img width="264" alt="Screen Shot 2022-05-21 at 12 56 14 PM" src="https://user-images.githubusercontent.com/97269758/169636069-d2dac5ad-017d-46a1-8dc1-4244c835d091.png">

### Create the glue job

<img width="481" alt="Screen Shot 2022-05-21 at 1 03 16 PM" src="https://user-images.githubusercontent.com/97269758/169636235-50f346a0-a806-4929-b60e-3c369974d880.png">

mandatory IAM role policies:

<img width="899" alt="Screen Shot 2022-05-21 at 1 05 39 PM" src="https://user-images.githubusercontent.com/97269758/169636417-5ca0b51d-0b33-4de0-9803-a33349319fbe.png">

choose the source table:

<img width="977" alt="Screen Shot 2022-05-21 at 1 14 12 PM" src="https://user-images.githubusercontent.com/97269758/169636605-6ef0da33-48b3-4c20-999a-941e9617efc4.png">

choose the destination table:

<img width="977" alt="Screen Shot 2022-05-21 at 1 14 51 PM" src="https://user-images.githubusercontent.com/97269758/169636627-8676641f-4b78-47a7-8bdb-890b5735669d.png">

<img width="985" alt="Screen Shot 2022-05-21 at 1 15 53 PM" src="https://user-images.githubusercontent.com/97269758/169636665-72a90cbe-4799-4f95-8635-0488fcda269e.png">

<img width="1754" alt="Screen Shot 2022-05-21 at 1 17 17 PM" src="https://user-images.githubusercontent.com/97269758/169636701-ef093413-283f-4622-a5ee-d026213a852f.png">

the job transformation script is shown as below:

```py
import sys
from awsglue.transforms import *
from awsglue.utils import getResolvedOptions
from pyspark.context import SparkContext
from awsglue.context import GlueContext
from awsglue.job import Job

## @params: [JOB_NAME]
args = getResolvedOptions(sys.argv, ['JOB_NAME'])

sc = SparkContext()
glueContext = GlueContext(sc)
spark = glueContext.spark_session
job = Job(glueContext)
job.init(args['JOB_NAME'], args)

datasource0 = glueContext.create_dynamic_frame.from_catalog(database = "msjyetlnew", table_name = "raw0515", transformation_ctx = "datasource0")


from pyspark.sql.functions import input_file_name
from pyspark.sql.functions import substring

df = datasource0.toDF().withColumn("date", substring(input_file_name(), 19, 8))

datasource1 = datasource0.fromDF(df, glueContext, "datasource1")

applymapping1 = ApplyMapping.apply(frame = datasource1, mappings = [("datastatus", "string", "datastatus", "string"), ("tradeindex", "string", "tradeindex", "string"), ("tradechan", "string", "tradechan", "string"), ("securityid", "string", "securityid", "string"), ("tradtime", "string", "tradtime", "string"), ("tradprice", "string", "tradprice", "string"), ("tradvolume", "string", "tradvolume", "string"), ("trademoney", "string", "trademoney", "string"), ("tradebuyno", "string", "tradebuyno", "string"), ("tradesellno", "string", "tradesellno", "string"), ("tradebsflag", "string", "tradebsflag", "string"), ("bizindex", "string", "bizindex", "string"), ("localtime", "string", "localtime", "string"), ("seqno", "string", "seqno", "string"),("date", "string", "date", "string")], transformation_ctx = "applymapping1")

selectfields2 = SelectFields.apply(frame = applymapping1, paths = ["datastatus", "tradeindex", "tradechan", "securityid", "tradtime", "tradprice", "tradvolume", "trademoney", "tradebuyno", "tradesellno", "tradebsflag", "bizindex", "localtime", "seqno", "date"], transformation_ctx = "selectfields2")

resolvechoice3 = ResolveChoice.apply(frame = selectfields2, choice = "MATCH_CATALOG", database = "msjyetlnew", table_name = "destination0516", transformation_ctx = "resolvechoice3")

resolvechoice4 = ResolveChoice.apply(frame = resolvechoice3, choice = "make_struct", transformation_ctx = "resolvechoice4")

datasink5 = glueContext.write_dynamic_frame.from_catalog(frame = resolvechoice4, database = "msjyetlnew", table_name = "destination0516", additional_options = {"partitionKeys":["date"]},transformation_ctx = "datasink5")

job.commit()
```
After the job has been succefully executed, switch to athena console and execute the below command:

```sql
msck repair table destination0516;
```

### debug

Debugging is achieved through Dev endpoints

<img width="253" alt="Screen Shot 2022-05-21 at 1 26 02 PM" src="https://user-images.githubusercontent.com/97269758/169636936-28da63d9-d1af-42c3-86dc-1678bd75fb52.png">


<img width="905" alt="Screen Shot 2022-05-21 at 1 34 27 PM" src="https://user-images.githubusercontent.com/97269758/169637228-d0c1f646-2998-45e4-b7af-fcde2faa24e8.png">


<img width="906" alt="Screen Shot 2022-05-21 at 1 34 44 PM" src="https://user-images.githubusercontent.com/97269758/169637233-5bb6742c-2f3b-4912-9077-3f32eac6abfd.png">

<img width="921" alt="Screen Shot 2022-05-21 at 1 35 29 PM" src="https://user-images.githubusercontent.com/97269758/169637251-b9f4d0e0-43f3-4268-bde3-aa084abb2683.png">



SSH public key could be acquired by executing the following command in ec2 instance:

```ssh
sudo -u ec2-user -i
cat /home/ec2-user/.ssh/authorized_keys
```

Get access the the endpint through local terminal by executing the following command:

```ssh
ssh -i ~/Desktop/mar28.pem glue@ec2-52-81-95-176.cn-north-1.compute.amazonaws.com.cn -t gluepyspark3
```

# Transform Field Value mapping to different dictionary sets


```py
from pyspark.sql.functions import input_file_name
from pyspark.sql.functions import substring
from pyspark.sql.functions import when

df = datasource0.toDF().withColumn("date",substring(input_file_name(), 19, 8))

df2 = df.withColumn("new_gender", when(df.gender == "M","Male")
                                 .when(df.gender == "F","Female")
                                 .when(df.gender.isNull() ,"")
                                 .otherwise(df.gender))

datasource1 = datasource0.fromDF(df2, glueContext, "datasource1")
```

# Guidance
https://docs.aws.amazon.com/glue/latest/dg/aws-glue-programming-etl-partitions.html
https://docs.aws.amazon.com/glue/latest/dg/aws-glue-api-crawler-pyspark-extensions-dynamic-frame-writer.html
