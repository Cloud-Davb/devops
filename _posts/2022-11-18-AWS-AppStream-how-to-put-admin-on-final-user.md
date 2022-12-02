---
layout: post
title: AWS - AppStream 2.0 Allow end users to have admin rights to AppStream end users
categories: AWS
author: "David BEAURY"
tags: [aws, AppStream 2.0, Admin] 
---
## Introduction

I have written this article to allow the end user to have administrator rights in the Appstream desktop on Windows 2019

## How to do it

1.	Within the Image Builder Admin (or an account that has administrator rights on the Image Builder), launch the "Computer Management" MMC tool. (Easiest way is to click start, then type in "Computer", and Computer Management will show up)
2.	From the left nav, expand "Local Users and Groups", then select "Groups"
3.	Open the Administrators group
4.	Click Add, then enter in "interactive" in the search box, and hit OK. "NT AUTHORITY\INTERACTIVE (S-1-5-4)" should now appear in the Administrators group
5.	Continuing installing and configuring your applications, and create the image
6.	Apply the newly created image to the fleet
