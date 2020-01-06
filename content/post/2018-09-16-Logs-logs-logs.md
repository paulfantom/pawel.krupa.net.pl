---
title: "Logs, logs, logs"
description: "Logging system for heterogeneous geographically distributed environment"
date: 1970-01-01
thumbnailImagePosition: left
thumbnailImage: https://www.internetstaff.com/wp-content/uploads/2014/10/journal2-300x183.png
tags:
  - logs
  - journald
  - distributed
---

# Logging system for heterogeneous geographically distributed environment

You probably think this post will be about some large scale google-like infrastructure with thousands of
nodes. Surprise! It's about my simple setup of some droplets in DigitalOcean, couple of raspberry pis, and 2 NAS boxes
located at my apartment and my parents home. Pretty simple and not uncommon setup.

## Why?
Logs, why would someone care about creating a solution for storing logs in a small scale system? One could say:

> I have just 5 devices, why do I need to collect logs? I can log into each box and check what happened.

But there are good reasons for keeping your logs somewhere else. It usually comes down to:
- better log archival and rotation
- log parsing, filtering, and aggregation
- event correlation
- easier viewing and reporting

Imagine how would you analyze what happened to you system after someone got access to it or your main storage
failed? Seems almost impossible without having remote log storage.

As for me keeping logs stored somewhere else is important due to reduced I/O operations on SD card of raspberry pi's
(nice to have with SBC running 24/7) and easier system management since I don't have to log into each box.

## Collect!
Ok, so we know why remote log storage is important, but how can we ship them there? If you go to google and search for
"log collection" or "log management" you will probably find software like Logstash or Fluentd, but for my purpose I want
to use something that is available out of the box on on every one of my systems.

### systemd
All my systems run systemd which ships with its own local log management system called journald. This is used by most 
of system services running on host so it would make much sense to use it to forward logs somewhere else. However looking
over journald documentation I couldn't find any simple method. There is a module called 
[systemd-netlogd][netlogd] which description say that it "Forwards messages from the journal to other hosts over the 
network using syslog format RFC 5424". Perfect! That is exactly what I am looking for. However its development isn't 
very active and it isn't shipped in for of deb/rpm package. At that point I almost decided to install fluentd or 
fluent-bit everywhere as I already have a working ansible role for that [here][ansible-fluentd], but 
first I just had to satisfy my curiosity and check what is this [RFC 5424][rfc5424] about? Oh, a syslog protocol.

### rsyslog
Syslog? The same syslog invented in the 80s? Yes, why not?! Rsyslog is already enabled on almost all of my systems and
it comes with features crucial to me. Its integration with journald is cleanly described in [Red Hat system administrators guide for RHEL7][rhel7guide]
and it can securly ship logs over untrusted networks by using TLS for transport security and authetication. I don't
need anything else... apart from a place to send those logs to.

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

[netlogd][https://github.com/systemd/systemd-netlogd]
[rfc5424][https://tools.ietf.org/html/rfc5424]
[ansible-fluentd][https://github.com/cloudalchemy/ansible-fluentd]
[rhel7guide][https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/s1-interaction_of_rsyslog_and_journal]
