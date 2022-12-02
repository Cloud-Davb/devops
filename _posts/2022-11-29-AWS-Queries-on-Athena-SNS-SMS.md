---
layout: post
title: AWS - Queries on Athena from SNS service (SMS logs)
categories: AWS
author: "David BEAURY"
tags: [aws, SNS, logs] 
---
<h2>
  <img style="border-radius:50%; width: 40px; height: 40px;" src="https://media-exp1.licdn.com/dms/image/C5603AQHc4Hgz7Lh-8Q/profile-displayphoto-shrink_400_400/0/1517593121277?e=1675296000&v=beta&t=G2VbG5Xsl6RVw0OUayLe3PsmHbeTRfymzXWsLiWvFhA" alt="Avatar">
      {{ author | default: page.author | escape }}
  <a target="_blank" href="https://www.linkedin.com/in/david-beaury-0740b462/">
    <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-linkedin" viewBox="0 0 16 16">
      <path d="M0 1.146C0 .513.526 0 1.175 0h13.65C15.474 0 16 .513 16 1.146v13.708c0 .633-.526 1.146-1.175 1.146H1.175C.526 16 0 15.487 0 14.854V1.146zm4.943 12.248V6.169H2.542v7.225h2.401zm-1.2-8.212c.837 0 1.358-.554 1.358-1.248-.015-.709-.52-1.248-1.342-1.248-.822 0-1.359.54-1.359 1.248 0 .694.521 1.248 1.327 1.248h.016zm4.908 8.212V9.359c0-.216.016-.432.08-.586.173-.431.568-.878 1.232-.878.869 0 1.216.662 1.216 1.634v3.865h2.401V9.25c0-2.22-1.184-3.252-2.764-3.252-1.274 0-1.845.7-2.165 1.193v.025h-.016a5.54 5.54 0 0 1 .016-.025V6.169h-2.4c.03.678 0 7.225 0 7.225h2.4z"/>
    </svg>
  </a>
</h2>
## Introduction
I wrote this article in order to follow the details of sending SMS via the SNS service of AWS, which for me is not detailed enough in the interface, it allowed me to understand why I had reached limits too quickly (I saw the number of SMS sent but the cost did not correspond to the calculation I had made). 

By analysing the logs I realised that I was sending several SMS for each sending.

I used the Athena service because the logs are generated in an S3 bucket with one file per day, so it is difficult to analyse. 

## How to do it

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
