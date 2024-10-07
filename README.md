# Collecting Windows Firewall Logs with Azure AMA

**Collect firewall logs with Azure Monitor Agent.**

https://learn.microsoft.com/en-us/azure/azure-monitor/agents/data-sources-firewall-logs

This lab provides step-by-step instructions on how to collect Windows Firewall logs using Azure services, including creating a Windows VM, setting up Log Analytics, and configuring data collection.

[Install Azure CLI on Windows](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-windows?tabs=azure-cli)


**az login** 

**az account show --query "name"**


## Step 1: Create a Resource Group

First, create a resource group to organize all the resources needed for this lab.

```cmd
:: Define parameters
set resourceGroup=mohibul-Lab-RG
set location=WestUS

:: Create resource group
az group create --name "%resourceGroup%" --location "%location%"
```

## Step 2: Create a Windows Virtual Machine

Create a Windows VM that will be used to generate firewall logs.

```cmd
:: Define parameters
set image=Win2019Datacenter
set size=Standard_B2s
set vmName=mohibul-Lab-VM
set adminUsername=mohibul
set adminPassword=YourPassword123!

:: Create the VM
az vm create ^
  --resource-group %resourceGroup% ^
  --name %vmName% ^
  --image %image% ^
  --admin-username %adminUsername% ^
  --admin-password %adminPassword% ^
  --size %size%
```

### Configure Windows Firewall Logging

After creating the VM, configure Windows Firewall logging.

[Verify that Firewall Logs are Being Created](https://learn.microsoft.com/en-us/azure/azure-monitor/agents/data-sources-firewall-logs#verify-that-firewall-logs-are-being-created)


```cmd
:: Enable logging for allowed and dropped connections
netsh advfirewall set allprofiles logging allowedconnections enable
netsh advfirewall set allprofiles logging droppedconnections enable

:: Verify logging configuration
netsh advfirewall show allprofiles

:: Open Group Policy Editor to further configure logging policies
gpedit
```

Ensure the log file is created at:
```
C:\windows\system32\logfiles\firewall\pfirewall.log
```

## Step 3: Set Up Log Analytics Workspace

Create a Log Analytics Workspace to collect and store firewall logs.

```cmd
:: Define parameters
set workspaceName=mohibul-Lab-Workspace

:: Create the Log Analytics workspace
az monitor log-analytics workspace create ^
  --resource-group %resourceGroup% ^
  --workspace-name %workspaceName% ^
  --location %location%
```

## Step 4: Add Firewall table to Log Analytics Workspace

[Add Firewall Table to Log Analytics Workspace](https://learn.microsoft.com/en-us/azure/azure-monitor/agents/data-sources-firewall-logs#add-firewall-table-to-log-analytics-workspace)



## Step 5: Create Data Collection Endpoint

Create a Data Collection Endpoint (DCE) to facilitate data flow.

```cmd
:: Define parameters
set endpointName=mohibul-Lab-DCEndpoint
set publicNetworkAccess=Enabled

:: Create the Data Collection Endpoint
az monitor data-collection endpoint create ^
  -g %resourceGroup% ^
  -l %location% ^
  --name %endpointName% ^
  --public-network-access %publicNetworkAccess%
```

## Step 6: Create Data Collection Rule (DCR)


[Create a Data Collection Rule to Collect Firewall Logs](https://learn.microsoft.com/en-us/azure/azure-monitor/agents/data-sources-firewall-logs#create-a-data-collection-rule-to-collect-firewall-logs)



## Step 7: Test the Firewall Logs in Log Analytics

To verify if firewall logs are being ingested, query the "WindowsFirewall" table in Log Analytics.

[Sample Log Queries for Firewall Logs](https://learn.microsoft.com/en-us/azure/azure-monitor/agents/data-sources-firewall-logs#sample-log-queries)


```kql
WindowsFirewall
| take 10
```

## Step 8: Clean Up Resources

Once done, delete the resource group to clean up all resources.

```cmd
:: Delete the resource group
az group delete --name "%resourceGroup%" --yes --no-wait
```

