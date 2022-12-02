---
layout: post
title: AWS - Queries on Athena from SNS service (SMS logs)
categories: AWS
author: "David BEAURY"
tags: [aws, SNS, logs] 
---
## Introduction
---

<img src="https://cloud-davb.github.io/devops/images/post/2022-11-29-AWS-Queries-on-Athena-SNS-SMS-image6.png">

I wrote this article in order to follow the details of sending SMS via the SNS service of AWS, which for me is not detailed enough in the interface, it allowed me to understand why I had reached limits too quickly (I saw the number of SMS sent but the cost did not correspond to the calculation I had made). 

By analysing the logs I realised that I was sending several SMS for each sending.

I used the Athena service because the logs are generated in an S3 bucket with one file per day, so it is difficult to analyse. 

## How to do it
---
## Select the S3 bucket

Click on setting and manage:

<img src="https://cloud-davb.github.io/devops/images/post/2022-11-29-AWS-Queries-on-Athena-SNS-SMS-image1.png">

Select the S3 bucket:

<img src="https://cloud-davb.github.io/devops/images/post/2022-11-29-AWS-Queries-on-Athena-SNS-SMS-image2.png">

## Create database

Comme back to Editor and past the query and click to run:

```sql
create database S3_database_SNS_SMS_logs
```

<img src="https://cloud-davb.github.io/devops/images/post/2022-11-29-AWS-Queries-on-Athena-SNS-SMS-image3.png">

## Create table

Select the database on the left menu and past the query and click to run.

I put the date in string because the date format in the log file is not standard, so I prefer to transform it in the query.

```sql

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
<img src="https://cloud-davb.github.io/devops/images/post/2022-11-29-AWS-Queries-on-Athena-SNS-SMS-image4.png">

## Query between two date

Past the query (change the start date and the end date) and click to run:

```SQL

SELECT from_iso8601_timestamp(publishtimeutc) as publishtimeutc,messageid,MessageType,DeliveryStatus,PriceInUSD,PartNumber,TotalParts
FROM "s3_database_sns_sms_logs"."sns_sms_logs_table"

WHERE  parse_datetime(publishtimeutc,'yyyy-MM-dd''T''HH:mm:ss.SSSSSS''Z') 
     BETWEEN parse_datetime('2022-11-20T00:00:00.000Z','yyyy-MM-dd''T''HH:mm:ss.SSSSSS''Z') 
     AND parse_datetime('2022-11-28T23:59:59.999Z','yyyy-MM-dd''T''HH:mm:ss.SSSSSS''Z')
ORDER by publishtimeutc  

````
<img src="https://cloud-davb.github.io/devops/images/post/2022-11-29-AWS-Queries-on-Athena-SNS-SMS-image5.png">
