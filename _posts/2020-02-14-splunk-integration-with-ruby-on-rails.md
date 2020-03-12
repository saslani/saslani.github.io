---
layout: post
title:  "Splunk Integration with Ruby on Rails (dokcer or not!)"
date:   2020-01-10 10:11:00
image:  
tags:   [Splunk, Logging, Ruby, Rails, Docker]
---

This article is to walk you through setting up splunk and rails integration and not how to use splunk.

### Stack and Versions:
- Docker: 19.03.5
- Ruby: 2.6.4
- Rails: 6.0
- Gems:
    + Required:
        * rails_semantic_logger: 4.4 - 4.4.3
    + Related:
        * puma: 4.3.1
- Splunk: 
    + Locally hosted or Cloud (we will cover both)


### Step 1: Splunk sign up
[Create an account with splunk](https://www.splunk.com/), if you haven't one yet. You may choose to sign-up for the 14-day free trial for the cloud instance, or download splunk directly. We will cover both setups in this article!

### Step 2: Install gems
Install the gems, we're using `rails_sementic_logger` for logging and `puma` compiler

### Step 3: Configure Splunk
First you need to setup a token to use in our app, or from the command line and this is how you do it:
- Visit your cloud instance "https://prd-p-YOUR-INSTANCE.cloud.splunk.com" or your self-service setup "http://localhost:8000/"
- Click on "Settings"
- Under "DATA", choose "Data Inputs"
- Under "Local inputs", choose "HTTP Event Collector"
- Click on "New Token"
- Give your token a name and follow steps till you create a token
- Go back to "HTTP Event Collector" page and make sure the token in "Enabled"

### User command-line to make sure your token is working
Notice url is slightly different between the cloud instance and your local setup. We are looking for a success message: `{"text":"Success","code":0}`
- Self-service cloud instances: https://input-prd-p-XXXXXXXX.cloud.splunk.com:8088
- Managed cloud instances: https://http-inputs-XXXXXXXX.splunkcloud.com
- Enterprise instances (also your local setup "localhost"): https://my-self-hosted-splunk.com:8088

#### Self-service
- From your local machine: `curl -k https://localhost:8088/services/collector/event -H 'Authorization: Splunk YOUR_TOKEN' -d '{"event":"Hello, World!"}'`
- From docker, accessing splunk on your local host machine: `curl -k https://docker.for.mac.host.internal:8088/services/collector/event -H 'Authorization: Splunk YOUR_TOKEN' -d '{"event":"Hello, World!"}'`

#### Cloud
`curl -k https://input-prd-p-XXXXXXXX.cloud.splunk.com:8088/services/collector/event -H 'Authorization: Splunk YOUR_TOKEN' -d '{"event":"Hello, World!"}'`

### Step 4: Configure logs in application.rb
You may choose to put this configuration in `config > application.rb` or you environment specific configuration file. (i.e. `config > environments > development.rb`)

You may choose to put the following block in `if ENV["RAILS_LOG_TO_STDOUT"].present?` if you want the ability to turn logs _on_ or _off_

_application.rb_

