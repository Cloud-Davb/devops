---
layout: post
title: AWS - How to repaire SSM services in Windows 2019 EC2
categories: AWS
author: "David BEAURY"
tags: [aws, SSM, UserData, EC2, Powershell] 
---
## Introduction

I have written this article to be sure SSM service is completly install and running on Windows 2019 EC2 in user data.
In fact most of time when I want to use SSM the service not work and we can't connect on it. To solve it at the end of my user data script I force to install again SSM.

## How to do it

This script EC2Launch pour être sur que SSM soit bien initialisé" add this line compatible in Powershell script:

```POWERSHELL
    ....
      UserData: !Base64 |
        <powershell>

            C:\ProgramData\Amazon\EC2-Windows\Launch\Scripts\InitializeInstance.ps1

        </powershell>

````