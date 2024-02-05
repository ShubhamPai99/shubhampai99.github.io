---
title: "Label your Units and Pay Attention to Alarms"
date: 2024-02-04T19:19:14-05:00
tags: []
description: ""
omit_header_text: true
draft: true
---
A couple weeks ago I was approached by an application team whose Lambda alarm was constantly going off in our production alerts webex channel.

```
LAMBDA-NAME-prod-Lambda-Duration
Alarm Description: Lambda Duration > 80
------------------------------------------
Threshold crossed on 2 datapoints: [600, 1400] were >= the threshold [24.0]
```

First, I checked AWS cloudwatch alarm and, indeed, the lambda datapoints were well over the threshold. Then I went into the cloudwatch logs and the runtimes weren't erroring or giving any warn messages. 

I then went back to the error message and dissected it:
1. `Lambda Duration > 80`. Ok, 80 *what*? 80 seconds, 80 milliseconds, 80 percent?
2. `the threshold [24.0]` Ok, 24 *what*?
3. `2 datapoints [600, 1400] (...) threshold [24.0]` Why are the datapoints *WAY* higher than the threshold to begin with?

Having been thoroughly confused, I decided to analyze the module that generated this alarm. The module was set up so that the team can define everything for the lambda in one Terraform module block.
