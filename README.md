# data-preparation

this doc is aiming at describing the process of dealing with vast volume of raw data preparation for the downstream analytics.

## raw data pull from source
for customers from mainland China, Baidu Net Disk is highly probable to used to store the raw data. As a result, the first step in terms of data preparation is to pull the raw data that is stored on Baudi Net Disk to aws S3.

network route is designed as below to achieve an optimized route to transfer these data.

![image](https://user-images.githubusercontent.com/97269758/160224355-aadf9cbb-71e7-4b8e-8803-30b974e48e43.png)

It is essential to download these data from Baidu Netdisk manually to the windows EC2. After that, can use aws cli installed on windows EC2 to unzip the file in batch and push them to S3 bucket.

to unzip those files in batch:
```
unzip -d /dir/ *.zip
```
to transfer those unzipped files to specific s3 bucket:
```
aws s3 cp d:/data s3://<bucket name>/raw --recursive
```
the snapshot of s3 bucket as shown as below:

![1](https://user-images.githubusercontent.com/97269758/160239260-d6dee628-b269-48d9-b65f-11dfbfe54914.png)

![Screen Shot 2022-03-26 at 7 30 41 PM](https://user-images.githubusercontent.com/97269758/160237427-67374731-e2b6-45f9-8024-ff87a37be06f.png)

compression is very important for data processing, choosing the right compression algorithm greatly saves time and cost.
for this case, we choose parquet as the data format, snappy as the compression algorithm. Meanwhile, data partitioning is another important factor in terms of accelerating query speed as well as saving cost.
The steps below shows how to conver the raw data which is initially stored as txt to parquet, and partitioned by date field (in this case, is datadate field)

to create a meta data, a database namespace in glue console, and finds it in athena console afterwards.

![Screen Shot 2022-03-26 at 8 09 23 PM](https://user-images.githubusercontent.com/97269758/160238739-8f8dd6aa-959f-4d90-bd71-fdcd1d0430b4.png)

![Screen Shot 2022-03-26 at 8 09 46 PM](https://user-images.githubusercontent.com/97269758/160238755-4d4c1dab-7440-4772-b540-0a8226311b8b.png)

to execute the sql query in athena console to create a table mapping raw txt data.

```sql
CREATE EXTERNAL TABLE raw_txt (
	datadate string,
	ticker string,
	exchangecd string,
	shortnm string,
	secoffset string,
	bartime string,
	closeprice float,
	openprice float,
	highprice float,
	lowprice float,
	volume bigint,
	value float,
	vwap float
) 
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.OpenCSVSerde' 
WITH SERDEPROPERTIES ("separatorChar" = "\t")
LOCATION 's3://a-minsheng-raw/raw-txt/' 
TBLPROPERTIES ('skip.header.line.count' = '1');
```
to execute the sql query in athena console to create a table with datadate as partition and parquet as data format.

```sql
CREATE EXTERNAL TABLE raw_date_parquet (
	ticker string,
	exchangecd string,
	shortnm string,
	secoffset string,
	bartime string,
	closeprice float,
	openprice float,
	highprice float,
	lowprice float,
	volume bigint,
	value float,
	vwap float
) 
PARTITIONED BY (datadate string)
STORED AS parquet
LOCATION 's3://a-minsheng-raw/raw-date-parquet/'
```

to execute the sql query in athena console to convert the data in raw_txt to raw_date_parquet.

```sql
INSERT INTO raw_date_parquet
SELECT ticker,
	exchangecd,
	shortnm,
	secoffset,
	bartime,
	closeprice,
	openprice,
	highprice,
	lowprice,
	volume,
	value,
	vwap,
	datadate
FROM raw_txt;
```
the partitioned data is stored in s3 as below:

![Screen Shot 2022-03-26 at 8 16 23 PM](https://user-images.githubusercontent.com/97269758/160239032-76966a93-0ffa-4ea8-a124-9c19d1aed85b.png)

