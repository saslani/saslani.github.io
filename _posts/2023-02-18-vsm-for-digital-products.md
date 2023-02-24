---
layout: post
title:  "VSM for Digital Products"
date:   2023-02-18 21:40:00
image:  vsm/tbd.jog
tags:   [Value Stream Mapping, Tool Making]
---

Value Stream Mapping (VSM) is a powerful tool for organizations looking to improve their efficiency and customer satisfaction by identifying and eliminating waste in their processes. It is a visual representation of the flow of materials and information required to bring a product or service from concept to customer. VSM helps teams to identify and eliminate bottlenecks in their processes.
This article is the first in series in which I will go over how to apply VSM for your digital product, with some examples.

Take any business where time is money in a true sense, for example, trading. Imagine building a trading platform where latency is measured in milliseconds and nanoseconds. When working within such tight constraints, you need to consider everything from the API language and technology of choice, to every algorithm you are running, the operating system where the API will be running in, the location of the server(s), length of the cables, etc. In such an environment, how do you find waste? Applying VSM lessons at a different scale would help you find and eliminate waste (or memory leaks) just like in physical products.

Local optimizations that might make sense to one team sometimes don’t create systemic improvements. Sometimes, the current process exists because it was the highest paid person in the room “feeling” strongly that it’s the right thing to do, the HiPPO (Highest Paid Person’s Opinion) Process. In these scenarios the order comes from the “top” and teams might feel it’s “above their pay grade” to question it. The orders often come with a timestamp attached to it and the teams need to deliver or else… often without considering the team’s skills and other priorities.

Rushing to pick something and get started without an understanding of VSM could lead to disappointment, a failed mission, money and time loss, loss of trust in management, burnout, layoffs, etc. Instead of the HiPPO process, spend a couple of days laying out the high-level VSM of a product family or the process you need to improve on. If you are new to the team, just ask the team members about the product they are building to get started:
- Inputs:
  - What data do you consume?
  - Where is the data you are consuming originating from?
  - Which team(s) are you dependent on?
- Outputs:
  - What data do you produce?
  - Who consumes your data?
  - Which team(s) depend on you?

The process of VSM begins by mapping the current state of the value stream, which includes all activities required to design, produce, and deliver a product or service. It is important to understand the current state in order to identify areas where waste is present. Once the current state is understood, teams can identify areas where waste is present, such as excess inventory, unnecessary steps, or delays. Once these areas of waste have been identified, the next step is to create a future state map that eliminates the waste and improves the flow of materials and information. By implementing the future state, organizations can reduce lead times, increase efficiency, and improve overall customer satisfaction.

When building a VSM for products in which latency is important, such as our trading system:
- Start by logging and benchmarking
- Use dashboards i.e. Datadog, Grafana, Prometheus, etc to visualize the data at every state of the process
- Verify that the metrics are correct 
- Understand what’s the norm, what falls into 99 and 99.99 percentile
- Diversify: this could be considering how the data change:
  - During a special time of the year
  - Time of the day
  - Certain location
  - News events etc.
- Look for patterns

You might think the steps above will take more than just a couple of days and you are right. But yet again, if you don’t have this information, you won’t be able to have evidence of where the waste is happening. So, start somewhere!

Implementing VSM can be a challenging process, but the benefits are well worth it. By identifying and eliminating waste in your processes, you can reduce lead times, increase efficiency, and improve overall customer and stakeholders’ satisfaction. Value Stream Mapping is not a one-time process. Iterative feedback and response is important to keep the product up-to-date. Once you have the first VSM created, make it easy for the team to maintain it. The best way for this is to assign a person to always take the responsibility to ensure the map reflects accurate information. 


In Part II of this series, I’ll cover a value stream map for a trading platform.
