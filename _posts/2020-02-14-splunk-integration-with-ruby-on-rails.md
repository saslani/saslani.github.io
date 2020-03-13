---
layout: post
title:  "Splunk Integration with Ruby on Rails (dokcer or not!)"
date:   2020-01-10 10:11:00
image:  
tags:   [Splunk, Logging, Ruby, Rails, Docker]
---

This article is to walk you through setting up splunk and rails integration and not how to use splunk.

**Stack and Versions:**
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


## Step 1: Splunk sign up
[Create an account with splunk](https://www.splunk.com/), if you haven't one yet. You may choose to sign-up for the 14-day free trial for the cloud instance, or download splunk directly. We will cover both setups in this article!

## Step 2: Install gems
We're using `rails_sementic_logger` for logging and `puma` compiler

## Step 3: Configure Splunk
First you need to setup a token to use in your app or from the command line and this is how you do it:
- Visit your cloud instance "https://prd-p-YOUR-INSTANCE.cloud.splunk.com" or your self-service setup "http://localhost:8000/"
- Click on "Settings"
- Under "DATA", choose "Data Inputs"
- Under "Local inputs", choose "HTTP Event Collector"
- Click on "New Token"
- Give your token a name and follow steps till you create a token
- Go back to "HTTP Event Collector" page and make sure the token in "Enabled"

## Step 4: Use command-line to make sure your token is working
Notice url is slightly different between the cloud instance and your local setup. We are looking for a success message: `{"text":"Success","code":0}`
- Self-service cloud instances: https://input-prd-p-XXXXXXXX.cloud.splunk.com:8088
- Managed cloud instances: https://http-inputs-XXXXXXXX.splunkcloud.com
- Enterprise instances (also your local setup "localhost"): https://my-self-hosted-splunk.com:8088

#### Self-service
- From your local machine: `curl -k https://localhost:8088/services/collector/raw -H 'Authorization: Splunk YOUR_TOKEN' -d '{"event":"Hello, World!"}'`
- From docker, accessing splunk on your local host machine: `curl -k https://docker.for.mac.host.internal:8088/services/collector/raw -H 'Authorization: Splunk YOUR_TOKEN' -d '{"event":"Hello, World!"}'`

#### Cloud
`curl -k https://input-prd-p-XXXXXXXX.cloud.splunk.com:8088/services/collector/raw -H 'Authorization: Splunk YOUR_TOKEN' -d '{"event":"Hello, World!"}'`

## Step 5: Configure logs in development.rb
Note: 
- You may choose to put the following block in `if ENV["RAILS_LOG_TO_STDOUT"].present?` if you want the ability to turn logs _on_ or _off_
- You can disable the logs for a block of code using:

{% highlight ruby %}
Rails.logger.silence do
  # Your code here
end
{% endhighlight %}

Minimal configuration for splunk in _development.rb_

{% highlight ruby %}
if ENV["RAILS_LOG_TO_STDOUT"] == true.to_s
  config.rails_semantic_logger.format = :json
  config.rails_semantic_logger.started = true
  config.rails_semantic_logger.add_file_appender = false

  # SPLUNK_EVENT_LOGGER_RAW: i.e. https://localhost:8088/services/collector/raw
  # SPLUNK_AUTH_TOKEN: token you created earlier
  # SPLUNK_VERIFY_SSL: If you need to verify a certificate

  SemanticLogger.add_appender(
    appender: :splunk_http,
    url: ENV["SPLUNK_EVENT_LOGGER_RAW"],
    token: ENV["SPLUNK_AUTH_TOKEN"],
    ssl: { verify_mode: ENV["SPLUNK_VERIFY_SSL"] == true.to_s ? OpenSSL::SSL::VERIFY_PEER : OpenSSL::SSL::VERIFY_NONE },
    compress: false,
    level: config.log_level,
    environment: ENV["RAILS_ENV"],
    application: "XYZ"
  )
end
{% endhighlight %}

## Step 6: Open a worker for SemanticLogger
[So put this in puma.rb](https://rdoc.info/gems/rails_semantic_logger/1.5.0/frames)

{% highlight ruby %}
  # Re-open appenders after forking the process
  SemanticLogger.reopen
{% endhighlight %}


## Step 7: Moment of truth!
Log a message anywhere in your code. For example `logger.info("Yo Ho Let's Go!")`. How log into your splunk and you should be seeing a message got logged into splunk instantly!


Happy logging and let me know if you run into any issues with the steps above!