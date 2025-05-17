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
5. [Detections](#detections)  
6. [Dashboard](#dashboard)  
7. [Screenshots](#screenshots)  
8. [Credits](#credits)  


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
┌────────────────────┐    Linux Universal Forwarder
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
---

We need Splunk Enterprise to collect and search logs. Here’s how:

1. **Download Splunk**  
   - Open your web browser and go to [Splunk Download](https://www.splunk.com/en_us/download/splunk-enterprise.html).  
   - Click **Linux .deb** and save the file to your `Downloads` folder.

2. **Install Splunk**  
   ```bash
   cd ~/Downloads
   sudo apt install ./splunk-*.deb
   sudo /opt/splunk/bin/splunk start --accept-license --answer-yes --no-prompt
   ```
3. Set an **admin password** when prompted.
   - When prompted by the start command, create a secure password fro the Admin user and please remember it.



## 2. First Login

1. Open your web browser and navigate to 
```
   https://localhost:8000
```  
2. Log in as **admin** using the password you configured.


## 3. Create the `lab_logs` Index

1. In Splunk Web, go to **Settings** > **Indexes**.  
2. Click **New Index**, enter `lab_logs` as the **Index Name**, then click **Save**.
 


 ##  Detections

Create three simple correlation searches (alerts) in Splunk Web:

1. **Failed logins**  
   - **Search**:
     ```spl
     index=lab_logs EventCode=4625
     | stats count by src_ip
     | where count >= 5
     ```
   - **Trigger:** when count ≥ 5 within the search window  
   - **Export:** save this alert in **Settings ▸ Searches, reports, and alerts**, then click **Export** to JSON and save as `detections/failed_logins.json`

2. **USB mass-storage insertion**  
   - **Search**:
     ```spl
     index=lab_logs EventCode=2003
     ```
   - **Trigger:** when any event appears  
   - **Export:** save as `detections/usb_insertion.json`

3. **New local admin user**  
   - **Search**:
     ```spl
     index=lab_logs (EventCode=4720 OR AuditEvent=useradd) group="Administrators"
     ```
   - **Trigger:** when any event appears  
   - **Export:** save as `detections/new_admin_user.json`



##  Dashboard

Build a dashboard called **Lab_Overview** with three panels:

1. **Event Volume Over Time**  
   - **Visualization:** Timechart  
   - **Search:**
     ```spl
     index=lab_logs | timechart span=1h count
     ```
   - **Export:** JSON → `dashboards/event_volume.json`

2. **Triggered Alerts Table**  
   - **Visualization:** Table  
   - **Search:** use your **Failed logins** saved search  
   - **Export:** JSON → `dashboards/alerts_table.json`

3. **Top 10 Source IPs**  
   - **Visualization:** Bar chart or table  
   - **Search:**
     ```spl
     index=lab_logs | top limit=10 src_ip
     ```
   - **Export:** JSON → `dashboards/top_ips.json`


##  Screenshots

Capture and commit these images under `diagrams/`:

| Step                  | File                         |
| --------------------- | ---------------------------- |
| First Login           | `diagrams/first-login.png`   |
| Index Created         | `diagrams/index-created.png` |
| Failed-Login Alert    | `diagrams/alert-fired.png`   |
| Dashboard (3 panels)  | `diagrams/dashboard.png`     |

---

##  Credit

- **Splunk Security Content** – official detection rules and dashboards:  
  https://github.com/splunk/security-content  


> All searches, dashboards, and configuration files in this repo were created from scratch for learning purposes.

---