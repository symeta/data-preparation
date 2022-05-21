# data-preparation

# Add Date Column to table, and Date as Table Partition

## Pre-requisite
Already have an athena table mapping to raw data stored in s3. a sample DDL of an athena table is as below:

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


# Transform Field Value mapping to different dictionary sets




