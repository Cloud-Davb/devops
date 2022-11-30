---
layout: post
title: AWS - query on Athena SNS SMS 
---
This article is write to do some query on AWS athena to extract logs from S3 bucket (generate by SNS SMS).

## Create database

create database S3_database_SNS_SMS_logs

```create table

CREATE EXTERNAL TABLE IF NOT EXISTS `s3_database_sns_sms_logs`.`SNS_SMS_logs_table` (
  `PublishTimeUTC` string,
  `MessageId` string,
  `DestinationPhoneNumber` string,
  `MessageType` string,
  `DeliveryStatus` string,
  `PriceInUSD` string,
  `PartNumber` int,
  `TotalParts` int
) COMMENT "SNS_SMS_logs_table"
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.OpenCSVSerde'
WITH SERDEPROPERTIES (
   'separatorChar' = ',',
   'quoteChar' = '"',
   'escapeChar' = '\\'
   )
STORED AS TEXTFILE
LOCATION 's3://buckets3logsns/SMSUsageReports/eu-west-3/2022/'
TBLPROPERTIES ("skip.header.line.count"="1")
```

## Query between two date

SELECT from_iso8601_timestamp(publishtimeutc) as publishtimeutc,messageid,MessageType,DeliveryStatus,PriceInUSD,PartNumber,TotalParts
FROM "s3_database_sns_sms_logs"."sns_sms_logs_table"

WHERE  parse_datetime(publishtimeutc,'yyyy-MM-dd''T''HH:mm:ss.SSSSSS''Z') 
     BETWEEN parse_datetime('2022-11-20T00:00:00.000Z','yyyy-MM-dd''T''HH:mm:ss.SSSSSS''Z') 
     AND parse_datetime('2022-11-28T23:59:59.999Z','yyyy-MM-dd''T''HH:mm:ss.SSSSSS''Z')
ORDER by publishtimeutc  