---
title: Azure Automation Hybrid Runbook Workers
description: This article provides information on installing and using Hybrid Runbook Worker which is a feature of Azure Automation that allows you to run runbooks on machines in your local datacenter or cloud provider.
services: automation
ms.service: automation
ms.component: process-automation
author: georgewallace
ms.author: gwallace
ms.date: 04/25/2018
ms.topic: article
manager: carmonm
---
# Automate resources in your data center or cloud with Hybrid Runbook Worker

Runbooks in Azure Automation may not be able to access resources in other clouds or in your on-premises environment since they run in the Azure cloud. The Hybrid Runbook Worker feature of Azure Automation allows you to run runbooks directly on the computer hosting the role and against resources in the environment to manage those local resources. Runbooks are stored and managed in Azure Automation and then delivered to one or more designated computers.

This functionality is illustrated in the following image:

![Hybrid Runbook Worker Overview](media/automation-hybrid-runbook-worker/automation.png)

## Hybrid Runbook Worker groups

Each Hybrid Runbook Worker is a member of a Hybrid Runbook Worker group that you specify when you install the agent. A group can include a single agent, but you can install multiple agents in a group for high availability.

When you start a runbook on a Hybrid Runbook Worker, you specify the group that it runs on. Each worker in the group polls Azure Automation to see if any jobs are available. If there is a job available then the first worker to get the job takes it. You cannot specify a particular worker.

## Installing a Hybrid Runbook Worker

The process to install a Hybrid Runbook worker is different depending on the OS. The following table contains links to the different methods you can use to install a Hybrid Runbook Worker. To install and configure a Windows Hybrid Runbook Worker, there are two methods available. The recommended method is using an Automation runbook to completely automate the process required to configure a Windows computer. The second method is following a step-by-step procedure to manually install and configure the role. For Linux machines, you run a Python script to install the agent on the machine

|OS  |Deployment Types  |
|---------|---------|
|Windows     | [PowerShell](automation-windows-hrw-install.md#automated-deployment)<br>[Manual](automation-windows-hrw-install.md#manual-deployment)        |
|Linux     | [Python](automation-linux-hrw-install.md#installing-linux-hybrid-runbook-worker)        |

> [!NOTE]
> To manage the configuration of your servers supporting the Hybrid Runbook Worker role with Desired State Configuration (DSC), you need to add them as DSC nodes. For more information about onboarding them for management with DSC, see [Onboarding machines for management by Azure Automation DSC](automation-dsc-onboarding.md).
>
>If you enable the [Update Management solution](automation-update-management.md), any computer connected to your Log Analytics workspace is automatically configured as a Hybrid Runbook Worker to support runbooks included in this solution. However, it is not registered with any Hybrid Worker groups already defined in your Automation account. The computer can be added to a Hybrid Runbook Worker group in your Automation account to support Automation runbooks as long as you are using the same account for both the solution and Hybrid Runbook Worker group membership. This functionality has been added to version 7.2.12024.0 of the Hybrid Runbook Worker.

Review the [information for planning your network](#network-planning) before you begin deploying a Hybrid Runbook Worker. After you have successfully deployed a runbook worker, review [run runbooks on a Hybrid Runbook Worker](automation-hrw-run-runbooks.md) to learn how to configure your runbooks to automate processes in your on-premises datacenter or other cloud environment.

## Removing Hybrid Runbook Worker

You can remove one or more Hybrid Runbook Workers from a group or you can remove the group, depending on your requirements. To remove a Hybrid Runbook Worker from an on-premises computer, perform the following steps:

1. In the Azure portal, navigate to your Automation account.
2. From the **Settings** blade, select **Keys** and note the values for field **URL** and **Primary Access Key**. You need this information for the next step.

### Windows

Open a PowerShell session in Administrator mode and run the following command. Use the **-Verbose** switch for a detailed log of the removal process.

```powershell
Remove-HybridRunbookWorker -url <URL> -key <PrimaryAccessKey>
```

To remove stale machines from your Hybrid Worker group, use the optional `machineName` parameter.

```powershell-interactive
Remove-HybridRunbookWorker -url <URL> -key <PrimaryAccessKey> -machineName <ComputerName>
```

### Linux

```bash
sudo python onboarding.py --deregister --endpoint="<URL>" --key="<PrimaryAccessKey>" --groupname="Example" --workspaceid="<workspaceId>"
```

> [!NOTE]
> This does not remove the Microsoft Monitoring Agent from the computer, only the functionality and configuration of the Hybrid Runbook Worker role.

## Remove Hybrid Worker groups

To remove a group, you first need to remove the Hybrid Runbook Worker from every computer that is a member of the group using the procedure shown earlier, and then you perform the following steps to remove the group.

1. Open the Automation account in the Azure portal.
1. Under **Process Automation**, select **Hybrid worker groups**. Select the group you wish to delete. After selecting the specific group, the **Hybrid worker group** properties page is displayed.

   ![Hybrid Runbook Worker Group page](media/automation-hybrid-runbook-worker/automation-hybrid-runbook-worker-group-properties.png)

1. On the properties page for the selected group, click **Delete**. A message appears asking you to confirm this action, select **Yes** if you are sure you want to proceed.

   ![Delete Group Confirmation Dialog](media/automation-hybrid-runbook-worker/automation-hybrid-runbook-worker-confirm-delete.png)

   This process can take several seconds to complete and you can track its progress under **Notifications** from the menu.

## <a name="network-planning"></a>Configure your network

### Hybrid Worker role

For the Hybrid Runbook Worker to connect to and register with Log Analytics, it must have access to the port number and the URLs that are described in this section. This is in addition to the [ports and URLs required for Microsoft Monitoring Agent](../log-analytics/log-analytics-agent-windows.md) to connect to Log Analytics.

If you use a proxy server for communication between the agent and the Log Analytics service, ensure that the appropriate resources are accessible. If you use a firewall to restrict access to the internet, you must configure your firewall to permit access.

The following port and URLs are required for the Hybrid Runbook Worker role to communicate with Automation:

* Port: Only TCP 443 is required for outbound internet access.
* Global URL: *.azure-automation.net
* Global URL of US Gov Virginia: *.azure-automation.us
* Agent Service: https://\<workspaceId\>.agentsvc.azure-automation.net

If you have an Automation account that's defined for a specific region, you can restrict communication to that regional datacenter. The following table provides the DNS record for each region.

| **Region** | **DNS record** |
| --- | --- |
| West Central US | wcus-jobruntimedata-prod-su1.azure-automation.net</br>wcus-agentservice-prod-1.azure-automation.net |
| South Central US |scus-jobruntimedata-prod-su1.azure-automation.net</br>scus-agentservice-prod-1.azure-automation.net |
| East US 2 |eus2-jobruntimedata-prod-su1.azure-automation.net</br>eus2-agentservice-prod-1.azure-automation.net |
| Canada Central |cc-jobruntimedata-prod-su1.azure-automation.net</br>cc-agentservice-prod-1.azure-automation.net |
| West Europe |we-jobruntimedata-prod-su1.azure-automation.net</br>we-agentservice-prod-1.azure-automation.net |
| North Europe |ne-jobruntimedata-prod-su1.azure-automation.net</br>ne-agentservice-prod-1.azure-automation.net |
| South East Asia |sea-jobruntimedata-prod-su1.azure-automation.net</br>sea-agentservice-prod-1.azure-automation.net|
| Central India |cid-jobruntimedata-prod-su1.azure-automation.net</br>cid-agentservice-prod-1.azure-automation.net |
| Japan East |jpe-jobruntimedata-prod-su1.azure-automation.net</br>jpe-agentservice-prod-1.azure-automation.net |
| Australia South East |ase-jobruntimedata-prod-su1.azure-automation.net</br>ase-agentservice-prod-1.azure-automation.net |
| UK South | uks-jobruntimedata-prod-su1.azure-automation.net</br>uks-agentservice-prod-1.azure-automation.net |
| US Gov Virginia | usge-jobruntimedata-prod-su1.azure-automation.us<br>usge-agentservice-prod-1.azure-automation.us |

For a list of region IP addresses instead of region names, download the [Azure Datacenter IP address](https://www.microsoft.com/download/details.aspx?id=41653) XML file from the Microsoft Download Center.

> [!NOTE]
> The Azure Datacenter IP address XML file lists the IP address ranges that are used in the Microsoft Azure datacenters. Compute, SQL, and storage ranges are included in the file.
>
>An updated file is posted weekly. The file reflects the currently deployed ranges and any upcoming changes to the IP ranges. New ranges that appear in the file aren't used in the datacenters for at least one week.
>
> It's a good idea to download the new XML file every week. Then, update your site to correctly identify services running in Azure. Azure ExpressRoute users should note that this file used to update the Border Gateway Protocol (BGP) advertisement of Azure space the first week of each month.

### Update Management

In addition to the standard addresses and ports that the Hybrid Runbook Worker requires, the following addresses are required specifically for Update Management. Communication to these addresses is done over port 443.

|Azure Public  |Azure Government  |
|---------|---------|
|*.ods.opinsights.azure.com     |*.ods.opinsights.azure.us         |
|*.oms.opinsights.azure.com     | *.oms.opinsights.azure.us        |
|*.blob.core.windows.net|*.blob.core.usgovcloudapi.net|

## Troubleshooting

The Hybrid Runbook Worker depends on the Microsoft Monitoring Agent to communicate with your Automation account to register the worker, receive runbook jobs, and report status. If registration of the worker fails, here are some possible causes for the error:

1. The hybrid worker is behind a proxy or firewall.

   Verify the computer has outbound access to *.azure-automation.net on port 443.

2. The computer the hybrid worker is running on has less than the minimum hardware requirements.

   Computers running the Hybrid Runbook Worker should meet the minimum hardware requirements before designating it to host this feature. Otherwise, depending on the resource utilization of other background processes and contention caused by runbooks during execution, the computer becomes over utilized and cause runbook job delays or timeouts.

   Confirm the computer designated to run the Hybrid Runbook Worker feature meets the minimum hardware requirements. If it does, monitor CPU and memory utilization to determine any correlation between the performance of Hybrid Runbook Worker processes and Windows. If there is memory or CPU pressure, this may indicate the need to upgrade or add additional processors, or increase memory to address the resource bottleneck and resolve the error. Alternatively, select a different compute resource that can support the minimum requirements and scale when workload demands indicate an increase is necessary.

3. The Microsoft Monitoring Agent service is not running.

   If the Microsoft Monitoring Agent Windows service is not running, this prevents the Hybrid Runbook Worker from communicating with Azure Automation. Verify the agent is running by entering the following command in PowerShell: `get-service healthservice`. If the service is stopped, enter the following command in PowerShell to start the service: `start-service healthservice`.

4. In the **Application and Services Logs\Operations Manager** event log, you see event 4502  and EventMessage containing **Microsoft.EnterpriseManagement.HealthService.AzureAutomation.HybridAgent** with the following description: *The certificate presented by the service \<wsid\>.oms.opinsights.azure.com was not issued by a certificate authority used for Microsoft services. Please contact your network administrator to see if they are running a proxy that intercepts TLS/SSL communication. The article KB3126513 has additional troubleshooting information for connectivity issues.*
    This can be caused by your proxy or network firewall blocking communication to Microsoft Azure. Verify the computer has outbound access to *.azure-automation.net on ports 443.

Logs are stored locally on each hybrid worker at C:\ProgramData\Microsoft\System Center\Orchestrator\7.2\SMA\Sandboxes. You can check if there are any warning or error events written to the **Application and Services Logs\Microsoft-SMA\Operations** and **Application and Services Logs\Operations Manager** event log that would indicate a connectivity or other issue affecting onboarding of the role to Azure Automation or issue while performing normal operations.

For additional steps on how to troubleshoot issues with Update Management, see [Update Management - troubleshooting](automation-update-management.md#troubleshooting)

## Next steps

Review [run runbooks on a Hybrid Runbook Worker](automation-hrw-run-runbooks.md) to learn how to configure your runbooks to automate processes in your on-premises datacenter or other cloud environment.
