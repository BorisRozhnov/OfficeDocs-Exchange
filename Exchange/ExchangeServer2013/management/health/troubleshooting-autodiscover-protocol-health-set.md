﻿---
title: Troubleshooting Autodiscover.Protocol Health Set
TOCTitle: Troubleshooting Autodiscover.Protocol Health Set
ms:assetid: 06afdcc8-7920-4e88-b85a-98e67a19d221
ms:mtpsurl: https://technet.microsoft.com/en-us/library/ms.exch.scom.autodiscover.protocol(v=EXCHG.150)
ms:contentKeyID: 49720718
ms.date: 10/08/2015
mtps_version: v=EXCHG.150
---

<div data-xmlns="http://www.w3.org/1999/xhtml">

<div class="topic" data-xmlns="http://www.w3.org/1999/xhtml" data-msxsl="urn:schemas-microsoft-com:xslt" data-cs="http://msdn.microsoft.com/en-us/">

<div data-asp="http://msdn2.microsoft.com/asp">

# Troubleshooting Autodiscover.Protocol Health Set

</div>

<div id="mainSection">

<div id="mainBody">

<span> </span>

_**Applies to:** Exchange Server 2013_

_**Topic Last Modified:** 2015-03-09_

The Autodiscover.Protocol health set monitors the Autodiscover communications protocol on the Mailbox server.

If you receive an alert that specifies that Autodiscover.Protocol is unhealthy, this indicates an issue that may prevent users from accessing their mailboxes.

<span id="EXP"></span>

<div>

## Explanation

The Autodiscover.Protocol service is monitored by using the following probes and monitors.


<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 25%" />
<col style="width: 25%" />
<col style="width: 25%" />
</colgroup>
<thead>
<tr class="header">
<th>Probe</th>
<th>Health Set</th>
<th>Dependencies</th>
<th>Associated Monitors</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p>AutodiscoverSelfTestProbe</p></td>
<td><p>Autodiscover.Protocol</p></td>
<td><p>Active Directory</p></td>
<td><p>AutodiscoverSelfTestMonitor</p></td>
</tr>
</tbody>
</table>


For more information about probes and monitors, see [Server health and performance](https://technet.microsoft.com/en-us/library/jj150551\(v=exchg.150\)).

</div>

<div>

## Common issues

This probe can fail for any of the following common reasons:

  - The Autodiscover application pool (MSExchangeAutodiscoverAppPool) that is hosted on the monitored Client Access server (CAS) is not responding. Or, the Autodiscover application pool that is hosted on one or more Mailbox servers is not responding.

  - The Domain Controllers are not responding.

</div>

<div>

## User Action

It's possible that the service recovered after it issued the alert. Therefore, when you receive an alert that specifies that the health set is unhealthy, first verify that the issue still exists. If the issue does exist, perform the appropriate recovery actions outlined in the following sections.

<span id="verify"></span>

<div>

## Verifying the issue still exists

1.  Identify the health set name and the server name in the alert.

2.  The message details provide information about the exact cause of the alert. In most cases, the message details provide sufficient troubleshooting information to identify the root cause. If the message details are not clear, do the following:
    
    1.  Open the Exchange Management Shell, and then run the following command to retrieve the details of the health set that issued the alert:
        
            Get-ServerHealth <server name> | ?{$_.HealthSetName -eq "<health set name>"}
        
        For example, to retrieve the Autodiscover.Protocol health set details about server1.contoso.com, run the following command:
        
            Get-ServerHealth server1.contoso.com | ?{$_.HealthSetName -eq "Autodiscover.Protocol"}
    
    2.  Review the command output to determine which monitor reported the error. The **AlertValue** value for the monitor that issued the alert will be `Unhealthy`.
    
    3.  Rerun the associated probe for the monitor that’s in an unhealthy state. Refer to the table in the Explanation section to find the associated probe. To do this, run the following command:
        
            Invoke-MonitoringProbe <health set name>\<probe name> -Server <server name> | Format-List
        
        For example, assume that the failing monitor is **AutodiscoverSelfTestMonitor**. The probe associated with that monitor is **AutodiscoverSelfTestProbe**. To run that probe on server1.contoso.com, run the following command:
        
            Invoke-MonitoringProbe Autodiscover.Protocol\AutodiscoverSelfTestProbe -Server server1.contoso.com | Format-List
    
    4.  In the command output, review the **Result** value of the probe. If the value is **Succeeded**, the issue was a transient error, and it no longer exists. Otherwise, refer to the recovery steps outlined in the following sections.

</div>

<span id="TestMonitors"></span>

<div>

## AutodiscoverSelfTestProbe Recovery Actions

When you receive an alert from a health set, the email message contains the following information:

  - Name of the Mailbox server that sent the alert

  - Name of Mailbox server being monitored

  - Time and date when the alert occurred

  - Authentication mechanism used, and credential information

  - Full exception trace of the last error, including diagnostic data and specific HTTP header information

You can use the information in the full exception trace to help troubleshoot the issue. The exception generated by the probe contains a Failure Reason that describes why the probe failed.

To troubleshoot this issue, follow these steps:

1.  Review protocol logs on the Mailbox servers. By default, Protocol log files on the Mailbox server are located in the ***\<exchange server installation directory\>*\\Logging\\Autodiscover** folder.

2.  Create a test user account, and then log on to the Mailbox server by using the test user account in the address. For example, log on by using: https://*\<servername\>*:444/autodiscover/autodiscover.xml.
    
    If the test user account name passes, an issue may affect the mailbox server that’s hosting the monitored mailbox.

3.  Try to repeat the previous steps by using a test account on the Mailbox server.

4.  Check for alerts on the Autodiscover.Proxy Health Set that may indicate an issue that affects a specific Mailbox server. For more information, see [Troubleshooting Autodiscover.Proxy Health Set](troubleshooting-autodiscover-proxy-health-set.md).

5.  Check for alerts on the Autodiscover Health Set that may indicate a problem that affects specific Mailbox servers. For more information, see [Troubleshooting Autodiscover Health Set](troubleshooting-autodiscover-health-set.md).

6.  Start IIS Manager, and then connect to the Mailbox server that is reporting the issue. Verify that the **MSExchangeAutodiscoverAppPool** application pool is running on the Mailbox server.

7.  In IIS Manager, click **Application Pools**, and then recycle the **MSExchangeAutodiscoverAppPool** application pool by running the following command from the Shell:
    
        %SystemRoot%\System32\inetsrv\Appcmd recycle MSExchangeAutodiscoverAppPool

8.  Rerun the associated probe as shown in step 2c in the Verifying the issue still exists section.

9.  If the issue still exists, recycle the IIS service using the IISReset utility or by running the following command:
    
        Iisreset /noforce

10. Rerun the associated probe as shown in step 2c in the Verifying the issue still exists section.

11. If the issue still exists, restart the server.

12. After the server restarts, rerun the associated probe as shown in step 2c in the Verifying the issue still exists section.

13. If the probe continues to fail, you may need assistance to resolve this issue. Contact a Microsoft Support professional to resolve this issue. To contact a Microsoft Support professional, visit the [Exchange Server Solutions Center](http://go.microsoft.com/fwlink/p/?linkid=180809). In the navigation pane, click **Support options and resources** and use one of the options listed under **Get technical support** to contact a Microsoft Support professional. Because your organization may have a specific procedure for directly contacting Microsoft Product Support Services, be sure to review your organization's guidelines first.

</div>

</div>

<div>

## For More Information

[What's new in Exchange 2013](https://technet.microsoft.com/en-us/library/jj150540\(v=exchg.150\))

[Exchange 2013 cmdlets](https://technet.microsoft.com/en-us/library/bb124413\(v=exchg.150\))

</div>

</div>

<span> </span>

</div>

</div>

</div>
