---
layout: post
title: AWS - Backup SQL-RDS Database to S3
categories: AWS
author: "David BEAURY"
tags: [aws, RDS, SQL, S3] 
---
## Introduction
---

<img src="https://cloud-davb.github.io/devops/images/post/2023-01-24-AWS-Backup-SQL-RDS-Database-To-S3-image1.png">

I wrote this article in order to make an automatic backup of a SQL Express database under RDS: Indeed the backup of the database is made by snapshot and the recovery is very long, while a recovery by .bak file is fast. 

## How to do it
---
## Create option group for the RDS backup to allow backup

Go to OptionGroup on RDS database and add a new one

<img src="https://cloud-davb.github.io/devops/images/post/2023-01-24-AWS-Backup-SQL-RDS-Database-To-S3-image2.png">

Select SQLSERVER_BACKUP_RESTORE
Create a new Role to allow backup
Select on witch bucket you want to backup

<img src="https://cloud-davb.github.io/devops/images/post/2023-01-24-AWS-Backup-SQL-RDS-Database-To-S3-image3.png">

This is the result

<img src="https://cloud-davb.github.io/devops/images/post/2023-01-24-AWS-Backup-SQL-RDS-Database-To-S3-image4.png">

Modify your database and select the parameter group and option group (just created before)




## Create a powershell script file

This powershell script backup the SQL database in RDS into S3 bucket.
We have the function to wait the result of the backup "Wait-RdsNativeBackupSuccess": Success or Error. 
Create this file.ps1 in a backup directory: in my sample it is in c:\SQL\backupSQLtoS3.ps1

Configure the parameters in this file:

$server = 'db-xxxxx-prod.xxxxxx.eu-west-3.rds.amazxxxxxs.com'
$bucketS3 ='arn:aws:s3:::cliexxxxxxx/rd/'
$region = 'eu-west-3'
$database = 'databasename'
$username = 'user'
$password = 'password'

```Powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass -Force


function Wait-RdsNativeBackupSuccess($server, $database, $user, $pwd)
{
    $startDate = Get-Date
    $timeOutminutes = 10
    $retryIntervalSeconds = 10

    do {
        
            $awsResponse = SQLCMD -U $user -P $pwd -S $server -Q "select top 1 lifecycle from msdb.dbo.rds_fn_task_status(NULL,0) order by task_id desc"
            
            #I look for success in the answer 
            $testSuccess= $awsResponse[2].Contains('SUCCESS')
            
            if($testSuccess -eq 'True') {
                fonction-log "Backup SQL and copy to S3 bucket OK "
                break}
            
            #I look for ERROR in the answer
            $testSuccess= $awsResponse[2].Contains('ERROR')

            if($testSuccess -eq 'True') {
                fonction-log "Backup ERROR: " + $awsResponse 
                break
                
                }

            
            start-sleep -seconds $retryIntervalSeconds

    } while ($startDate.AddMinutes($timeOutminutes) -gt (Get-Date))

}


function fonction-log([string]$logtext){
            
                Write-Host "[DEBUG] [" + (Get-Date -UFormat "%Y-%m-%d-%T") + "] [$logtext]"
            
            }
            $timestamp = get-date -format dd-MM-yyyy--HH-mm-ss
            
            Start-Transcript -Path "c:\Sql\logs\$timestamp-backupSQL.log"
			
			fonction-log "******** start *********"


$server = 'db-xxxxx-prod.xxxxxx.eu-west-3.rds.amazxxxxxs.com'
$bucketS3 ='arn:aws:s3:::cliexxxxxxx/rd/'
$region = 'eu-west-3'
$database = 'databsename'
$username = 'user'
$password = 'password'

$timestamp = get-date -format dd-MM-yyyy--HH-mm-ss

$fileName =  "$bucketS3$database-$timestamp.bak"
	
try
    {
        SQLCMD -U $username -P $password -S $server -Q "exec msdb.dbo.rds_backup_database @source_db_name='$database', 
        @s3_arn_to_backup_to='$fileName', @overwrite_S3_backup_file=1;"

        #attente du retour du backup
        Wait-RdsNativeBackupSuccess -server $server -database dbName -user $username -pwd $password

        }
        catch
        {
           Write-Host $_.Exception.Message -ForegroundColor Red
		   fonction-log "backup SQL Nok : $_.Exception.Message"
        }
	

fonction-log "******** stop *********"

Stop-Transcript
            

            
```



## Create cheduler


