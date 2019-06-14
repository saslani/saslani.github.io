---
layout: post
title:  "Hosting TeamCity on AWS using Terraform - v2"
date:   2019-05-18 15:08:15
categories: Infrastructure as Code
---

Enhancements:
- Prevent s3 from being destroyed via ``` terraform destroy ``` command
- Prevent database from being destroyed via ``` terraform destroy ``` command


- backup db images into s3 bucket
- add disaster recovery notes on how to recover database
- state management

- add load balancer
- add ssl
