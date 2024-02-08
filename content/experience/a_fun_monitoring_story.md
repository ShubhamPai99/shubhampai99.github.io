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
1. `Lambda Duration > 80`. Ok, 80 *what*? seconds, milliseconds, percent?
2. `the threshold [24.0]` 24 *WHAT* -__-?
3. `2 datapoints [600, 1400] (...) threshold [24.0]` Why are the datapoints *WAY* higher than the threshold to begin with?

Having been thoroughly confused, I decided to analyze the module that generated this alarm. The module was set up so that the team can define everything for the lambda in one Terraform module block. One input for the lambda module was `timeout` and this parameter was measured in **seconds**. This timeout is what the duration alarms threshold was built on. The equation for the duration alarm threshold was the following:
$ lambda_duration_threshold = lambda_function_timeout * a_user_defined_percent $
Which brought light to why we're seeing the error message. In the above example, 80 meant 80% and the threshold was just the lambda's timeout (in this case 30s) * 80% = 24.

But that didn't explain how the lambda was running for datapoints *way* larger than the threshold. Besides, if the timeout is 30, how is the lambda somehow running for 600s or 1400s? 

:confounded: it's because the threshold is in milliseconds :confounded: 

So it turns out that our threshold (what we thought was 24 seconds) was actually 24 milliseconds. Off by a factor of 1000. In fact, other teams had already found this out and changed their local alarms. But for the rest of Digital, everyone seemed to be ignoring the alarms in their webex channels. 

Let this be a lesson- in high school and college our teachers yelled at us to include our units. Let's keep doing that, and let's also pay attention to alarms when things go off :stuck_out_tongue_winking_eye:
