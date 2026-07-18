 ## Azure VM Monitoring Project: AMA, DCR, Log Analytics, Alerts, and Grafana

## Project Overview

This project demonstrates a complete Azure VM monitoring setup using **Azure Monitor Agent (AMA)**, **Data Collection Rules (DCR)**, **Log Analytics Workspace (LAW)**, **Azure Monitor Alerts**, **Action Groups**, and **Azure Monitor Dashboards with Grafana**.

The lab focuses on monitoring an Azure Windows virtual machine named **VM1**. The monitoring data is collected by AMA, controlled by a DCR, stored in a Log Analytics Workspace, queried using KQL, used for alerting, and finally visualized in Grafana dashboards.

---

## Architecture


Azure Virtual Machine: VM1
        |
        v
Azure Monitor Agent: AMA
        |
        v
Data Collection Rule: datacollectionrule
        |
        v
Log Analytics Workspace: law-vm-monitoring-lab
        |
        +--> KQL Queries
        +--> Azure Monitor Alert Rules
        +--> Grafana Dashboard Panels
```

---

## What This Project Covers

- Azure VM monitoring using Azure Monitor Agent.
- Creating and associating a Data Collection Rule.
- Sending Windows Event Logs and Performance Counters to Log Analytics Workspace.
- Running KQL queries to validate collected data.
- Creating an Azure Monitor alert for missing AMA heartbeat.
- Creating an Action Group for email notification.
- Creating Grafana dashboard panels for heartbeat and CPU usage.
- Understanding the difference between Azure Monitor Metrics and Log Analytics Workspace.
- Troubleshooting common Grafana and KQL issues.

---

## Azure Services Used

| Azure Service | Purpose |
|---|---|
| Azure Virtual Machine | Target machine to monitor |
| Azure Monitor Agent | Collects guest OS monitoring data |
| Data Collection Rule | Defines what data to collect and where to send it |
| Log Analytics Workspace | Stores logs and performance counter data |
| Azure Monitor Alerts | Creates alert rules from metrics or logs |
| Action Group | Sends alert notifications such as email |
| Dashboards with Grafana | Visualizes Azure Monitor Logs using Grafana panels |

---

## Lab Resource Names

The following resource names were used in this project. These can be changed based on your environment.

```text
Virtual Machine: VM1
Resource Group: Test
Log Analytics Workspace: law-vm-monitoring-lab
Data Collection Rule: datacollectionrule
Alert Rule: HeartbeatRule
Action Group: Actiongroup
Grafana Dashboard: Dashboard-07-18-2026-20-36
```

---

## Prerequisites

Before starting, ensure that you have:

- An active Azure subscription.
- A Windows Azure VM created.
- Permission to create and manage:
  - Log Analytics Workspace
  - Data Collection Rule
  - Azure Monitor Agent extension
  - Alert Rule
  - Action Group
  - Grafana Dashboard
- Basic knowledge of Azure Portal.
- Basic understanding of KQL.

---

# Step 1: Create Log Analytics Workspace

1. Open **Azure Portal**.
2. Search for **Log Analytics workspaces**.
3. Click **Create**.
4. Enter the following details:

```text
Resource Group: Test
Workspace Name: law-vm-monitoring-lab
```

5. Click **Review + Create**.
6. Click **Create**.

The Log Analytics Workspace stores the monitoring data collected from the VM.

---

# Step 2: Create a Data Collection Rule

1. Open **Azure Portal**.
2. Search for **Monitor**.
3. Go to **Data Collection Rules**.
4. Click **Create**.
5. Enter the following details:

```text
Rule Name: datacollectionrule
Resource Group: Test
Platform Type: Windows
```

6. Continue to the **Resources** tab.

---

# Step 3: Add VM to the DCR

1. In the DCR creation wizard, go to **Resources**.
2. Click **Add resources**.
3. Select the Azure VM.

```text
VM Name: VM1
```

4. Click **Apply**.

This associates the VM with the DCR. AMA uses the associated DCR to understand what data should be collected.

---

# Step 4: Configure Data Sources in DCR

## 4.1 Windows Event Logs

Add Windows Event Logs as a data source.


Destination:

```text
Destination Type: Log Analytics Workspace
Workspace: law-vm-monitoring-lab
```

---

## 4.2 Performance Counters

Add Performance Counters as another data source.


Destination:

```text
Destination Type: Log Analytics Workspace
Workspace: law-vm-monitoring-lab
```

Important note:

```text
If Performance Counters are sent to Azure Monitor Metrics, the data will not appear in the Log Analytics Perf table.
If you want to query performance counters using KQL and visualize them in Grafana Logs panels, send Performance Counters to Log Analytics Workspace.
```

---

# Step 5: Verify AMA Installation on VM

Go to:

```text
Azure Portal → Virtual Machines → VM1 → Extensions + applications
```

Verify that the following extension is installed:

```text
AzureMonitorWindowsAgent
```

Expected status:

```text
Provisioning succeeded
```

If the extension is visible and provisioning succeeded, AMA is installed on the VM.

---

# Step 6: Verify DCR Association

Go to:

```text
Azure Portal → Monitor → Data Collection Rules → Select your DCR → Resources
```

Confirm that **VM1** is listed under associated resources.

---

# Step 7: Verify Data in Log Analytics Workspace

Go to:

```text
Azure Portal → Log Analytics Workspace → law-vm-monitoring-lab → Logs
```

Run the following KQL queries.

---

## KQL Query: Check AMA Heartbeat

```kusto
Heartbeat
| where TimeGenerated > ago(24h)
| where Category == "Azure Monitor Agent"
| project TimeGenerated, Computer, Category, OSType, Version
| order by TimeGenerated desc
```

Expected result:

```text
VM1 should appear in the query output.
```

---


## KQL Query: Check Which Tables Are Receiving Data

```kusto
search *
| where TimeGenerated > ago(24h)
| summarize Count=count() by $table
| order by Count desc
```

Useful tables for this lab:

```text
Heartbeat
Event
Perf
```

---

## KQL Query: Check Windows Event Logs

```kusto
Event
| where TimeGenerated > ago(24h)
| project TimeGenerated, Computer, EventLog, Source, EventID, EventLevelName, RenderedDescription
| order by TimeGenerated desc
```


# Step 8: Create Alert for Missing AMA Heartbeat

## Alert Objective

Create an alert when VM1 does not send AMA heartbeat for 5 minutes.

Go to:

```text
Log Analytics Workspace → law-vm-monitoring-lab → Alerts → Create alert rule
```


Alert logic:

```text
Measure: Table rows
Operator: Less than
Threshold: 1
Evaluation frequency: 1 minutes
Lookback period: 5 minutes
```

Meaning:

```text
If no heartbeat row is found for VM1 in the last 5 minutes, trigger the alert.
```

Alert rule details:

```text
Alert Rule Name: HeartbeatRule
Severity: 3 - Warning
Enable upon creation: Yes
```

---

# Step 9: Create Action Group

During alert creation:

1. Go to **Actions**.
2. Click **Create action group**.
3. Enter:

```text
Action Group Name: Actiongroup
Display Name: Email
Resource Group: Test
```

4. Notification type:

```text
Email/SMS message/Push/Voice
```

5. Add an email address.
6. Enable **Common alert schema**.
7. Click **Create**.

The action group sends an email when the heartbeat alert fires.

---

# Step 10: Create Grafana Dashboard

Go to:

```text
Azure Portal → Monitor → Dashboards with Grafana
```

Create a new dashboard:

```text
New → New dashboard → Add visualization
```

Select the following data source settings:

```text
Data Source: Azure Monitor
Service: Logs
Resource: law-vm-monitoring-lab
Logs Mode: Analytics
Time Column: TimeGenerated
```

---

# Grafana Panel 1: AMA Heartbeat Count

Visualization:

```text
Time series
```

Panel title:

```text
VM-AMA-LAW-HEARTBEAT
```

KQL query:

```kusto
Heartbeat
| where $__timeFilter(TimeGenerated)
| where Category == "Azure Monitor Agent"
| where Computer =~ "VM1"
| summarize HeartbeatCount=count() by bin(TimeGenerated, 5m)
| order by TimeGenerated asc
```

This panel shows heartbeat count from VM1 over time.

---

# Grafana Panel 2: CPU Usage

First, discover the actual CPU counter name:

```kusto
Perf
| where $__timeFilter(TimeGenerated)
| where Computer =~ "VM1"
| summarize Count=count() by ObjectName, CounterName, InstanceName
| order by Count desc
```



This panel shows average CPU usage over time.

---


Key point:

```text
If performance counters are sent to Azure Monitor Metrics, they will not appear in the Perf table.
If performance counters are sent to Log Analytics Workspace, they can be queried using the Perf table.
```

---

# Troubleshooting

## Issue 1: Perf Query Shows No Data

Check the following:

```text
DCR → Data Sources → Performance Counters
DCR → Destination → Log Analytics Workspace
DCR → Resources → VM1 associated
Time range → Last 24 hours
```

Run:

```kusto
Perf
| where TimeGenerated > ago(24h)
| where Computer =~ "VM1"
| summarize Count=count() by ObjectName, CounterName, InstanceName
| order by Count desc
```

---

## Issue 2: Grafana Shows “Data is missing a time field”

Cause:

```text
The query does not return TimeGenerated.
```

Wrong query example:

```kusto
Perf
| where $__timeFilter(TimeGenerated)
| summarize Count=count() by ObjectName, CounterName, InstanceName
```

Fix:

Use `bin(TimeGenerated, 5m)` in the summarize statement.

Correct query example:

```kusto
Perf
| where $__timeFilter(TimeGenerated)
| where Computer =~ "VM1"
| where ObjectName == "Processor Information"
| where CounterName == "% Processor Time"
| where InstanceName == "_Total"
| summarize AvgCPU=avg(CounterValue) by bin(TimeGenerated, 5m)
| order by TimeGenerated asc
```

---

## Issue 3: Grafana Shows “Data is missing a number field”

Cause:

```text
The query returns text/log rows but no numeric value.
```

Fix:

Use numeric aggregation such as:

```text
count()
avg()
sum()
min()
max()
```

Correct heartbeat query:

```kusto
Heartbeat
| where $__timeFilter(TimeGenerated)
| where Category == "Azure Monitor Agent"
| where Computer =~ "VM1"
| summarize HeartbeatCount=count() by bin(TimeGenerated, 5m)
| order by TimeGenerated asc
```

---

## Issue 4: Alert Does Not Fire

Check:

```text
Query returns correct data in Log Analytics
Alert scope is Log Analytics Workspace
Condition is Count less than 1
Action Group is attached
Alert rule is enabled
Evaluation frequency and lookback period are configured correctly
```

---

# Final Dashboard Output

The final Grafana dashboard contains:

```text
CPU Usage
VM-AMA-LAW-HEARTBEAT
Optional Available Memory
Optional Disk Free Space
Optional Error and Critical Events
Optional Recent Windows Events
```

---

# Final Validation Checklist

| Check | Status |
|---|---|
| AMA installed on VM | Completed |
| DCR created | Completed |
| VM associated with DCR | Completed |
| Logs sent to LAW | Completed |
| Heartbeat data visible | Completed |
| Perf data visible | Completed |
| Alert rule created | Completed |
| Action group configured | Completed |
| Grafana dashboard created | Completed |
| CPU panel working | Completed |
| Heartbeat panel working | Completed |

---


### Key Learnings

- Azure Monitor Agent is used to collect guest OS data from Azure VMs.
- AMA requires a Data Collection Rule to know what data to collect and where to send it.
- Log Analytics Workspace stores VM monitoring data such as Heartbeat, Event, and Perf.
- KQL is used to validate and analyze collected monitoring data.
- Azure Monitor Alerts can detect missing heartbeat or other log-based conditions.
- Action Groups are used to send alert notifications.
- Grafana can visualize Azure Monitor Logs using KQL queries.
- Performance Counters must be sent to Log Analytics Workspace if we want to query them in the Perf table.
- Azure Monitor Metrics and Log Analytics Workspace are different monitoring destinations.


# Conclusion

This project successfully demonstrates an end-to-end Azure VM monitoring solution using Azure Monitor Agent, Data Collection Rules, Log Analytics Workspace, Azure Monitor Alerts, Action Groups, and Grafana dashboards.

The implementation validates the complete monitoring flow:

```text
VM1 → AMA → DCR → LAW → KQL → Alerting → Grafana
```

This lab is useful for learning Azure observability, Windows VM monitoring, KQL-based log analysis, Azure alerting, and operational dashboarding with Grafana.
