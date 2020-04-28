---
layout: post
title:  "Splunk Integration with Python and Flask"
date:   2020-03-13 10:11:00
image:  
tags:   [Splunk, Logging, Python, Flask, Docker]
---

This article is to walk you through setting up splunk and flask integration and not how to use splunk.

I used [splunk-handler](https://pypi.org/project/splunk-handler/) originally, but the library was very specific and complicated. Hence, I simplified the splunk-handler to serve my needs. 


## Step 1: Splunk sign up
[Create an account with splunk](https://www.splunk.com/), if you haven't one yet. You may choose to sign-up for the 14-day free trial for the cloud instance, or download splunk directly. We will cover both setups in this article!

## Step 2: Configure Splunk
First you need to setup a token to use in your app or from the command line and this is how you do it:
- Visit your cloud instance "https://prd-p-YOUR-INSTANCE.cloud.splunk.com" or your [self-service setup](https://splk.it/38OT28D) "http://localhost:8000/", assuming you are running on localhost port 8000
- Click on "Settings"
- Under "DATA", choose "Data Inputs"
 
![](/img/splunk/setting_data_input.png){:width="300x"}

- Under "Local inputs", choose "HTTP Event Collector"

![](/img/splunk/http_event_collector.png){:width="300x"}

- Click on "New Token"
- Give your token a name and follow steps till you create a token
- Go back to "HTTP Event Collector" page and make sure the token in "Enabled": from "Data Inputs > HTTP Event Controller" choose "Global Settings"

![](/img/splunk/global_setting.png){:width="600x"}

- "Enable" the token

![](/img/splunk/enable_token.png){:width="300x"}

- Change the "Default Index" to "main"

![](/img/splunk/default_index.png){:width="300x"}


![](/img/splunk/token_status_enabled.png){:width="600x"}

## Step 3: Use command-line to make sure your token is working
Let's make send a simple "Hello, World!" log to splunk to make sure the token is working. Notice url is slightly different between the cloud instance and your local setup. We are looking for a success message: `{"text":"Success","code":0}`. Also, notice that by default Splunk's HTTP Event Collector (HEC) is running on port **8088**
- Self-service cloud instances: https://input-prd-p-XXXXXXXX.cloud.splunk.com:8088
- Managed cloud instances: https://http-inputs-XXXXXXXX.splunkcloud.com
- Enterprise instances (also your local setup "localhost"): https://my-self-hosted-splunk.com:8088

#### Self-service
- From your local machine: 
{% highlight bash %}
curl -k https://localhost:8088/services/collector/raw -H 'Authorization: Splunk YOUR_TOKEN' -d '{"event":"Hello, World!"}'
{% endhighlight %}
- From docker, accessing splunk on your local host machine (mac in this case): 
{% highlight bash %}
curl -k https://docker.for.mac.host.internal:8088/services/collector/raw -H 'Authorization: Splunk YOUR_TOKEN' -d '{"event":"Hello, World!"}'
{% endhighlight %}
#### Cloud
{% highlight bash %}
curl -k https://input-prd-p-XXXXXXXX.cloud.splunk.com:8088/services/collector/raw -H 'Authorization: Splunk YOUR_TOKEN' -d '{"event":"Hello, World!"}'
{% endhighlight %}


## Step 4: Choose a python library that serves your needs
Below is the simplified version of [splunk-handler](https://pypi.org/project/splunk-handler/). I named it `splunk-handler.py` and placed it in `app > lib`

{% highlight python %}
# Super light version of https://pypi.org/project/splunk-handler/

import json
import logging
import time
import traceback

import requests
from requests.adapters import HTTPAdapter
from requests.packages.urllib3.util.retry import Retry

instances = []  # For keeping track of running class instances


class SplunkHandler(logging.Handler):
    def __init__(self, host, token, index="main", debug=False, verify=True):
        """
        Args:
          host (str): The Splunk host param
          token (str): Authentication token
          index (str): Splunk index to write to
          debug (bool): Whether to print debug console messages
          verify (bool): Whether to perform ssl certificate validation
        """

        global instances
        instances.append(self)
        logging.Handler.__init__(self)

        self.host = host
        self.token = token
        self.index = index
        self.verify = verify
        self.timeout = 60
        self.debug = debug
        self.session = requests.Session()
        self.write_debug_log("Starting debug mode")
        self.write_debug_log("Preparing to override loggers")

        # prevent infinite recursion by silencing requests and urllib3 loggers
        logging.getLogger("requests").propagate = False
        logging.getLogger("urllib3").propagate = False

        # and do the same for ourselves
        logging.getLogger(__name__).propagate = False

        # disable all warnings from urllib3 package
        if not self.verify:
            requests.packages.urllib3.disable_warnings()

        # Set up automatic retry with back-off
        self.write_debug_log("Preparing to create a Requests session")
        retry = Retry(
            total=5,
            backoff_factor=2.0,
            method_whitelist=False,  # Retry for any HTTP verb
            status_forcelist=[500, 502, 503, 504],
        )
        self.session.mount("https://", HTTPAdapter(max_retries=retry))
        self.write_debug_log("Class initialize complete")

    # Handler subclass, emit needs to be implemented
    def emit(self, record):
        self.write_debug_log("emit() called")

        try:
            record = self.format_record(record)
        except Exception as e:
            self.write_log("Exception in Splunk logging handler: %s" % str(e))
            self.write_log(traceback.format_exc())
            return

        # Flush log immediately; is blocking call
        self._splunk_worker(payload=record)
        return

    #
    # helper methods
    #

    def write_log(self, log_message):
        print("[SplunkHandler] " + log_message)

    def write_debug_log(self, log_message):
        if self.debug:
            print("[SplunkHandler DEBUG] " + log_message)

    def format_record(self, record):
        self.write_debug_log("format_record() called")

        params = {
            "time": time.time(),
            "host": self.host,
            "index": self.index,
            "source": record.pathname,
            "sourcetype": "text",
            "event": self.format(record),
        }

        self.write_debug_log("Record dictionary created")
        formatted_record = json.dumps(params, sort_keys=True)
        self.write_debug_log("Record formatting complete")
        return formatted_record

    def _splunk_worker(self, payload=None):
        self.write_debug_log("_splunk_worker() called")

        if payload:
            self.write_debug_log("Payload available for sending")
            try:
                self.write_debug_log("Sending payload: " + payload)
                record = self.session.post(
                    self.host,
                    data=payload,
                    headers={"Authorization": "Splunk %s" % self.token},
                    verify=self.verify,
                    timeout=self.timeout,
                )
                record.raise_for_status()  # Throws exception for 4xx/5xx status
                self.write_debug_log("Payload sent successfully")

            except Exception as e:
                try:
                    self.write_log("Exception in Splunk logging handler: %s" % str(e))
                    self.write_log(traceback.format_exc())
                except Exception:
                    self.write_debug_log(
                        "Exception encountered,"
                        + "but traceback could not be formatted"
                    )

            self.log_payload = ""
{% endhighlight %}

## Step 5: Add environment variables
Add the following to your .env file.
Note:
- Self-service cloud instances: https://input-prd-p-XXXXXXXX.cloud.splunk.com:8088
- Managed cloud instances: https://http-inputs-XXXXXXXX.splunkcloud.com
- Enterprise instances (also your local setup "localhost"): https://my-self-hosted-splunk.com:8088

{% highlight bash %}
SPLUNK_EVENT_LOGGER_RAW:
SPLUNK_AUTH_TOKEN: token you created earlier
{% endhighlight %}


## Step 6: Modify your code to use SplunkHandler

Initialize `SplunkHandler` in main.py or any class you need to send logs to splunk.
Note that I do not want to verify ssl in the following example.

{% highlight python %}
import os
from lib.splunk_handler import SplunkHandler

app = Flask(__name__)

splunk = SplunkHandler(
    host=os.getenv("SPLUNK_EVENT_LOGGER_RAW"),
    token=os.getenv("SPLUNK_AUTH_TOKEN"),
    verify=False
  )
{% endhighlight %}

Send a log to splunk:

{% highlight python %}
@app.route("/")
def hello():    
    app.logger.addHandler(splunk)
    app.logger.info("Hello World!")
{% endhighlight%}

## Step 7: Moment of truth!
Now log into your splunk and you should see that a message got logged into splunk instantly!


Happy logging and let me know if you run into any issues with the steps above!