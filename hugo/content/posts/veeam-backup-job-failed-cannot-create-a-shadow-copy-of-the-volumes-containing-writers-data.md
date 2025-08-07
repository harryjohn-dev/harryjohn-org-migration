+++
date = '2013-02-26T00:00:00+00:00'
draft = false
title = "Veeam Backup job failed. Cannot create a shadow copy of the volumes containing writer's data."
description = 'Troubleshooting Veeam backup failures related to VSS writers and disk space issues'
tags = ['veeam', 'vmware', 'backup', 'vss', 'troubleshooting']
categories = ['uncategorized']
comments = true
+++

{{< figure src="/images/Backup-job-failed.png" alt="Backup job failed" caption="Backup job failed" class="alignright" width="156" height="86" >}}

Occasionally one of our Veeam backup jobs will fail and most of the time it is due to shadow copy or VSS writer issue which a simple reboot will resolve, this solution has become our default answer to a failed backup with an error mentioning the words "shadow copy" or "VSS". One of our SQL servers had a failed backup last night and gave us the error "Cannot create a shadow copy of the volumes containing writer's data", here is the full error message:

> Failed to prepare guest for hot backup. Error: VSSControl: -2147467259
> Backup job failed. Cannot create a shadow copy of the volumes containing writer's data. VSS asynchronous operation is not completed.
> Operation: [Shadow copies commit]. Code: [0x8004231f].
> Error: VSSControl: -2147467259 Backup job failed. Cannot create a shadow copy of the volumes containing writer's data. VSS asynchronous operation is not completed. Operation: [Shadow copies commit]. Code: [0x8004231f].

This time a reboot did not fix the problem so checking the event viewer reveals a clear message that I am running out of space on C:

> When preparing a new volume shadow copy for volume C:, the shadow copy storage on volume C: did not have sufficiently large contiguous blocks. Consider deleting unnecessary files on the shadow copy storage volume or use a different shadow copy storage volume.

Why did my monitoring not pick this up you may be asking! Well the C drive is a 12GB volume with 2.5GB free -- 20.8%, just above our alert threshold by 0.8%, apparently 2.5GB free was not enough to run the shadow copies on this server!

Further investigation showed the default user profile's temporary internet files had inflated to 400MB! After resolving this issue by following this guide here on how [to get rid of the huge default user temporary internet files](http://www.paessler.com/blog/2009/04/17/prtg-7/how-to-get-rid-of-huge-default-userlocal-settingstemporary-internet-filescontentie5-folders).

Backup still failed! Time to check the status of the VSS writers by using the command:

```bash
vssadmin list writers
```

For each failed writer you need to restart the appropriate service. The following is a list of the failed writers I had and the services I restarted to bring them back to "stable"

- **SqlServerWriter** -- SQL Server VSS Writer
- **IIS Metabase Writer** -- IIS Admin Service  
- **WMI Writer** -- Windows Management Instrumentation

As soon as I had cleared up some free space on C, restarted these services and all VSS writers showed as stable the backup succeeded. Two things to learn here; we should also have a threshold on 2.5GB as well as 20% free space for shadow copies and that "Cannot create a shadow copy of the volumes containing writer's data" means we need to look beyond VSS/backups and more towards the storage that shadow copies are enabled for (check event logs).
