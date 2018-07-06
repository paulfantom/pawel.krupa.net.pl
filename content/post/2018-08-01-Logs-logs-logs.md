---
title: "Logs, logs, logs"
date: 2018-08-01
tags: ["logs", "journald", "distributed"]
---

# Logging system for heterogeneous geographically distributed environment

What a subject! You probably think this post will be about some large scale google-like infrastructure with thousands of
nodes. Surprise! It's about my simple setup of some droplets in DigitalOcean, couple of raspberry pis, and 2 NAS boxes
located at my apartment and my parents home. Pretty simple and common setup.

## Why?
Logs, why would someone care about creating a solution for storing logs in a small scale system? One could say:

> I have just 5 devices, why do I need to collect logs? I can log into each box and check what happened.

But there are good reasons for keeping your logs somewhere else. It usually comes down to:
- better log archival and rotation
- log parsing, filtering, and aggregation
- event correlation
- easier viewing and reporting

Imagine how would you analyze what happened to you system after someone got access to it or your main storage
failed? Of course without having remote log storage?

As for me keeping logs stored somewhere else is important due to reduced I/O operations on SD card of raspberry pi's
(nice to have with SBC running 24/7) and easier system management since I don't have to log into each box.

## Collect!


### systemd


### rsyslog


## Store!


### Self-hosted solutions


### Someone else problem AKA "cloud"


### LogDNA



## Sum up


## MOAR

If you want more information about logging systems I highly recommend to read NIST
[Guide to Computer Security Log Management](https://ws680.nist.gov/publication/get_pdf.cfm?pub_id=50881). And if are
even more curious about logs, their aggregation, processing, analysis, and real world examples go read
[Logging and Log Management](https://www.amazon.com/Logging-Log-Management-Authoritative-Understanding/dp/1597496359).
