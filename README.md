# Setup Splunk SIEM Lab on Linux

> **Goal:** Quickly stand up Splunk Enterprise, ingest one log source, create a failed-login alert, and visualize it on a basic dashboard—all on a Linux host.

---

## Table of Contents

1. [Overview](#overview)  
2. [Architecture](#architecture)  
3. [Prerequisites](#prerequisites)  
4. [Setup](#setup)  
   - [1. Install Splunk](#1-install-splunk)  
   - [2. First Login](#2-first-login)  
   - [3. Create the `lab_logs` Index](#3-create-the-lab_logs-index)  
   - [4. Ingest Log Sources](#4-ingest-log-sources)  
5. [Detections](#detections)  
6. [Dashboard](#dashboard)  
7. [Screenshots](#screenshots)  
8. [Credits & Inspiration](#credits--inspiration)  


---

## Overview

This lab demonstrates the core tasks of a junior SOC analyst by guiding you through:

1. Installing a free 60-day trial of [**Splunk Enterprise**](https://www.splunk.com/en_us/download.html) on a Linux host.  
2. Ingesting Windows or Linux security logs into a dedicated index.  
3. Creating three custom detection rules (failed logins, USB insertion, new admin user).  
4. Building a simple 3-panel SOC dashboard.  
5. Exporting JSON objects so anyone can reproduce the lab in under 30 minutes.

Follow these steps to validate your SIEM skills and build a portfolio piece.

---

## Architecture

```text
┌────────────────────┐     Windows Event Forwarder
│  Windows 10 VM     │────(Security, System logs)
└────────────────────┘                │
                                       ▼
┌────────────────────┐     Linux Universal Forwarder
│  Linux host        │────(auth.log, syslog)
└────────────────────┘                │
                                       ▼
          ┌──────────────────────────────────────────┐
          │       Splunk Enterprise                  │
          │  • Index: lab_logs                       │
          │  • Search & Reporting App                │
          │  • Alerts: 3 custom correlation searches │
          │  • Dashboard: Lab_Overview               │
          └──────────────────────────────────────────┘
```


## Setup

Follow these steps one by one.

### 1. Install Splunk

We need Splunk Enterprise to collect and search logs. Here’s how:

1. **Download Splunk**  
   - Open your web browser and go to [Splunk Download](https://www.splunk.com/en_us/download/splunk-enterprise.html).  
   - Click **Linux .deb** and save the file to your `Downloads` folder.

2. **Install Splunk**  
   ```bash
   cd ~/Downloads
   sudo apt install ./splunk-*.deb
   sudo /opt/splunk/bin/splunk start --accept-license --answer-yes --no-prompt
