# Setup Splunk SIEM Lab on Linux

> **Goal:** Quickly stand up Splunk, ingest one log source, create a “failed-login” alert, and visualize it on a basic dashboard.

---

##  Table of Contents

1. [Overview](#overview)
2. [Architecture](#architecture)
3. [Prerequisites](#prerequisites)
4. [Setup](#setup)

   * [1. Install Splunk](#1-install-splunk)
   * [2. First Login](#2-first-login)
   * [3. Create the `lab_logs` Index](#3-create-the-lab_logs-index)
   * [4. Ingest Log Sources](#4-ingest-log-sources)
5. [Detections](#detections)
6. [Dashboard](#dashboard)
7. [Screenshots](#screenshots)
8. [Credits & Inspiration](#credits--inspiration)


## Overview

This repository documents my hands-on lab for **setting up a Splunk SIEM tool** on a Linux host:

* Installing a 60-day Splunk Enterprise trial [here](https://www.splunk.com/en_us/download.html).
* Shipping Windows & Linux security logs into a dedicated index  
* Creating three custom detection rules (failed logins, USB insertion, new admin user)  
* Building a 3-panel SOC dashboard  
* Exporting JSON objects so anyone can reproduce the lab in < 30 minutes  

The project is designed to setup a SIEM tool as for a SOC roles.



## Architecture

```text
┌────────────────────┐     Windows Event Forwarder
│  Windows 10 VM     │────(Security, System logs)
└────────────────────┘                │
                                       ▼
┌────────────────────┐     Linux Universal Forwarder
│  Kali (Host OS)    │────(auth.log, syslog)
└────────────────────┘                │
                                       ▼
          ┌──────────────────────────────────────────┐
          │       Splunk Enterprise (9.2.x)         │
          │  • Index: lab_logs                       │
          │  • Search & Reporting App                │
          │  • Alerts: 3 custom correlation searches │
          │  • Dashboard: Lab_Overview               │
          └──────────────────────────────────────────┘