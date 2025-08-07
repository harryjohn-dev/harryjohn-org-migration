---
title: "Deploy multiple VMs from template with PowerCLI"
date: 2014-01-06T00:00:00Z
draft: false
tags: ["powercli", "vmware", "powershell", "automation", "deployment", "citrix", "xenapp", "xendesktop"]
categories: ["powercli", "vmware"]
author: "Harry John"
comments: true
---

![Deploy multiple VMs from template](/images/deploy-multiple-vms-from-template-300x211.png)

I frequently need to deploy multiple VMs from a template in large numbers, most often a bunch of Citrix XenApp or XenDesktop servers following an update to the gold image. Without an automated method deployment can become a headache and would take days to deploy multiple VMs, especially when taking into account the post-deploy tasks such as activating Windows, activating Office, installing Citrix, putting the computer into the right OU etc.

Rather than consuming large amounts of paracetamol I have written a (fairly long) PowerCLI script to remove as many of the manual and mundane tasks as I could. In this blog post I am going to run through the script and explain how it works. You may download the complete script and use it “out-the-box” or make your own modifications, otherwise read on to understand how I deploy multiple VMs from template using PowerCLI.

The script currently does the following:

- Deploys multiple VMs from template
- Chooses the best datastores to deploy multiple VMs based on capacity and number of VMs/datastore
- Powers on the VM
- Puts the VM in the correct OU within AD
- Activates Windows
- Activates Office
- Changes IP address
- Starts a custom service (I used this to start IMA service which adds the VM to Citrix AppCenter)

The complete script is currently approximately 700 lines in total and you can [download the complete script](http://www.harryjohn.org/wp-content/uploads/2014/01/DeployMultipleVMs1.zip), however for the educational purpose of this article I will just go through some major snippets. Hopefully this will help give you some ideas or to understand the basics of how this works.

In this blog post I will cover the following aspects of the script:

- How to deploy multiple VMs from template simultaneously
- Keeping track of VMs as they go through the build process
- Changing the IP address of a VM once deployed

## Deploy multiple VMs from template simultaneously

To deploy multiple VMs from template simultaneously I make use of [PowerShell Jobs](http://blogs.technet.com/b/heyscriptingguy/archive/2012/12/31/using-windows-powershell-jobs.aspx). For a complete understanding of how this works read through ScriptingGuy's excellent blog post. I will be using just two of these commands; Start-Job and Get-Job

Now, when invoking the Start-Job command a new hidden PowerShell instance will be launched in the background and runs the job within here. There are a few things to note;

1. A new PowerCLI instance means a new session to vCenter, you will need to forward the current session to each instance/job you start
2. If you have a PowerShell profile script, this script will run for each instance/job you start

I am not going to cover [Multi-threading PowerCLI](http://velemental.com/2012/03/11/multithreading-powercli/) as this has been explained very well on the vElemental blog, I suggest you check it out for a full understanding of how this works. For now just know that I will be using two .ps1 files -- the "deploy VM script" which will be invoked by what I like to call the "control script", every time we want to deploy another VM simultaneously.

## The deploy VM script

This script is simple. Other than the bit of work required in passing the vCenter session credentials over from the "control script", all we need to do here is create a mini PowerCLI script to deploy one VM. So first, lets gather some parameters about the VM we are to deploy:

```powershell
Param (
    $session = $(throw "missing -session parameter"),
    $vcserver,
    $name,
    $datastore,
    $ipaddr,
    $template,
    $folderName
)
```

There are just a few more parameters I will need, the host/cluster I am to deploy to, the customisation spec and the true VMware folder location:



```powershell
#========================================================================
# PARAMETERS
#========================================================================
# host/cluster
$vmhost = "cluster1"

# customisation spec
$custSpecName = "W2K8-R2-X64"

# location
$folder = Get-Folder -Location "Virtual Machines" -Name $folderName
```

There is something important to note about customisation specs. I have set my customisation spec to use DHCP and I will be applying a static IP once the VM has deployed, powered on and been customised. One method many people have discussed is to clone your main customisation spec into a new “temporary spec”, add the IP address details into this temporary spec and delete after you have deployed a VM with it. I just didn’t get along with this method. It would be nice if there was a way we could supply the details within PowerCLI for which you are prompted for when using the vSphere Client, as far as I know this is not possible.

Finally, we need the command to deploy a VM using the parameters we have collected above.

```powershell
try {
$vm = New-VM -Name $name `
	-VMHost $vmhost `
	-Template $template `
	-OSCustomizationSpec $custSpec `
	-Datastore $datastore `
	-Location $folder `
	-ErrorAction:Stop
}
catch
{
# your catch method....
}
```

That’s it for the deploy VM script, I told you it was simple!

## The control script

Getting slightly more complicated, the control script is the “brain” of the process and, you guessed it, controls the deployment of multiple VMs as well as completing any other post-deployment tasks. Lets just cover the basics.

First we must gather some information about the VMs we are to deploy. Some information I like to “hard code” in parameters at the top of the script such as the template name, DNS servers and gateway address. Other information I like to collect interactively when at run-time such as VM names, IP addresses, number of VMs. Here is an example:

```powershell
# HARD CODED PARAMETERS
# vm settings
$folder = "Citrix-NG Servers"
$template = 'Citrix XenApp 6.5 Farm Member Template (Citrix Installed)'

# network variables
$networkSubnet = "255.255.0.0"
$networkGateway = "172.16.0.1"
$networkDns = "172.16.0.2","172.16.0.3"

# GATHER FROM USER PARAMETERS
# get VM name prefix
$vmFirstName = Read-Host "Enter the name of the first VM (e.g. MAILSERVER01)"

# get number of VMs to build
[int]$vmQuantity = Read-Host "How many VMs are you deploying?"

# get IP Address 
$ipAddress = Read-Host "You need a block of $vmQuantity adjacent IP addresses, enter the first"
```

You might have already noticed, when I deploy multiple VMs they will have the same name but a different number at the end, for example; MAILSERVER01, MAILSERVER02 etc. I also deploy VMs with a sequence of adjacent IP addresses for example; 172.16.0.101, 172.16.0.102 etc. This makes deployment and this script very simple because all I need to know is the first VM name, the first IP address, and how many VMs to deploy. From this information I can create a nice PowerShell array with each VM name and its IP address. Lets split up the first IP address we have collected to do this:

```powershell
# split up IP address
$ipSplit = $ipAddress.Split(".")
$ipPrefix = $ipSplit[0] + "." + $ipSplit[1] + "." + $ipSplit[2] + "." 
[int]$ipSuffix = $ipSplit[3]
```

Now I have the IP address prefix, such as 172.16.0 and the first IP address suffix, such as 101. If I put together the prefix and the suffix I get the complete IP address: 172.16.0.101. I can add one to the suffix, put them together again and have the IP address of the next VM: 172.16.0.102. Simples!

Again, I need to do this with the VM name, so take the last two characters of the name and you have the first VM number (suffix), take the other characters and you have the VM name (prefix):

```powershell
# split up VM name
[int]$vmFirstNumber = $vmFirstName.Substring($vmFirstName.length - 2,2)
$vmNamePrefix = $vmFirstName.Substring(0,$vmFirstName.length - 2)
```

## Keeping track of VMs as they go through the build process

Now we have collected all the info we need to deploy multiple VMs, I like to create an array which will keep track of the VMs, tasks and their status. So lets create a loop to build the VM name and the IP address using the prefixes and suffixes we created earlier and add each one to the array. I also add a few other properties for each VM to keep track of the build status, such as Clone Initiated, Cloned, Powered On and Re-IP.

```powershell
###################
# Build $vmStatus array to keep track of VMs and completed tasks
###################

# create an object collection holding status info for all VMs being built
$vmStatus = @()

# create counters for VM name and IP address/datastore
$vmNumber = $vmFirstNumber
$vmCounter = 0

1..$vmQuantity | foreach {
	# if VM number is less than 10 add a 0 onto the server name so we dont have MAILSERVER1 for example
	if ($vmNumber -lt 10) {
		$vmName = $vmNamePrefix + "0" + $vmNumber
	} else {
		$vmName = $vmNamePrefix + $vmNumber
	}

	# compile IP address
        $vmIpAddress = $ipPrefix + ($ipSuffix + ($vmCounter))

	# setup object properties
	$properties = [ordered]@{
		MachineName = $vmName
		IpAddress = $vmIpAddress
		CloneInitiated = 0
		Cloned = 0
		PoweredOn = 0
		ReIP = 0
	}

	# create object and add to collection
	$obj = New-Object -TypeName PSObject -Property $properties
	$vmStatus += $obj

	# increase counters
	$vmNumber++
	$vmCounter++
}
```

This is a good start, we now have a list of VMs, their IP addresses and the status of deployment. You could modify the above to perhaps import a list from CSV, rather than ask for the VM details interactively. A good reason to do this would be if you wanted to deploy multiple VMs without following my naming convention or adjacent IP addresses.

Prepare the command to deploy a VM using the deploy VM script we created earlier. I also clear the current job list in case any are still in memory from a previous time I ran the script without reloading my PowerShell session:

```powershell
###################
# Prepare to start deploying VMs
###################

# prepare VM deploy job command
$job= {
	Set-Location $args[0]
	powershell -command ".\DeployVM.ps1 -session $($args[1]) -vcserver $($args[2]) -name $($args[3]) -datastore $($args[4]) -ipaddr $($args[5]) -template `'$($args[6])`' -folder `'$($args[7])`'"
}

# clear job list
Get-Job | Remove-Job
```

Time to start deploying multiple VMs! The following loop will continue looping until all tasks have been completed, that is all the properties for each VM within $vmStatus have a value of 1. Notice in the next few examples before each task I select the VMs which have not yet had that task completed and where previous tasks have been completed using a where query on $vmStatus. If the task then completes successfully, I set that property to 1.

```powershell
while ($vmsCompleted.Count -lt $vmQuantity) {

	###################
	# Initiate VM cloning, using $maxConcurrentJobs as the limit
	###################

	# do not start job if $maxConcurrentJobs are already running, do not start if all VMs are now cloned (clone initiated) - skip this task
	if (($vmsCloneInitiated.Count -lt $vmQuantity) -and ($runningJobCount -lt $maxConcurrentJobs)) {

        # select VMs which have not yet been cloned - and only the first $maxConcurrentJobs VMs
        $vmStatus | where {($_.Cloned -eq 0)} | Select-Object -first $maxConcurrentJobs | foreach {

		    # create job
		    Start-Job -Name $_.MachineName -ScriptBlock $job -ArgumentList $currentPath, $session, $vcserver, $_.MachineName, $_.Datastore, $_.IpAddress, $template, $folder		

		    # update vmStatus object collection
		    $_.CloneInitiated = 1
        }
	}
```

There are a few variables in the above example I have not yet mentioned, such as $vmsComplete.Count, $vmsCloneInitiated.Count and $runningJobCount. Lets update these

```powershell
	###################
	# Check running job count and look for VMs which have finished cloning
	###################

	# count current running jobs
	$jobs = Get-Job 
	$runningJobs = $jobs | ? { $_.state -eq "Running" }
	$runningJobCount = $runningJobs.count

	# check for vms which have finished cloning
	$completedJobs = $jobs | ? { $_.state -eq "Completed" }	

	# update $vmStatus
	foreach ($completedJob in $completedJobs) {

		# get index of object and set cloned to 1
		$vmStatusIndex = 0..($vmStatus.Count -1) | where {$vmStatus[$_].MachineName -eq $completedJob.Name}
		$vmStatus[$vmStatusIndex].Cloned = 1
	}
```

Finally, the rest of the loop to complete the remaining tasks. I have stripped out much of the code within each task to keep this looking simple for the purpose of the article.


```powershell
	###################
	# Power on VMs which have finished cloning
	###################

	$vmStatus | where {($_.Cloned -ne 0) -and ($_.PoweredOn -eq 0)} | foreach {

		# start the VM
		Start-VM $_.MachineName

		# update $vmStatus
		$_.PoweredOn = 1
	}	

	###################
	# Set correct IP address for VMs and start Citrix service
	###################

	$vmStatus | where {($_.Cloned -ne 0) -and ($_.PoweredOn -ne 0) -and ($_.ReIP -eq 0)} | foreach {

		# check machine is online
		if (Test-Connection $_.MachineName -Count 1 -ErrorAction SilentlyContinue) {

            # code to set IP address - mentioned later in blog post
		}
	}

	###################
	# Update counters and sleep
	###################

	# update counters
	[ARRAY]$vmsCloneInitiated = $vmStatus | ? CloneInitiated -eq 1
	[ARRAY]$vmsCompleted = $vmStatus | ? {($_.Cloned -ne 0) -and ($_.PoweredOn -ne 0) -and ($_.ReIP -ne 0)}

	# sleep for $cycleTime seconds
	Start-Sleep $cycleTime
}
```


## Changing the IP address of a VM once deployed

I mentioned earlier I didn’t get along with the guest customisation method of setting the VM’s IP address. In the example above you can see where I set the IP address – one of the last things I do. I removed the code snippet for clarity in the article but here it is. Note, I also flush DNS and wait 15 seconds before continuing:

```powershell
$vmStatus | where {($_.Cloned -ne 0) -and ($_.PoweredOn -ne 0) -and ($_.InAD -ne 0) -and ($_.ActivateWin -ne 0) -and ($_.ActivateOffice -ne 0) -and ($_.ReIP -eq 0)} | foreach {

		# check machine is online
		if (Test-Connection $_.MachineName -Count 1 -ErrorAction SilentlyContinue) {

			Write-Host "$($_.MachineName): Online, changing IP address"

			# change IP settings
			try {
				Get-VM -Name $_.MachineName | Get-VMGuestNetworkInterface `
					-Guestuser $localAdminUser `
					-GuestPassword $localAdminPassword | `
						? { $_.name -like $localNICNameQuery } | `
							Set-VMGuestNetworkInterface `
								-Guestuser $localAdminUser `
								-GuestPassword $localAdminPassword `
								-IPPolicy static `
								-IP $_.IpAddress `
								-Netmask $networkSubnet  `
								-Gateway $networkGateway `
								-DNS $networkDns

				Write-Host "$($_.MachineName): IP Address changed"

				# flush dns
				ipconfig /flushdns
                Start-Sleep 15

        		# update $vmStatus	
				$_.ReIP = 1
			}
			catch {
                $null
			}
		}
	}
```

## Conclusion

That’s all folks! As I mentioned previously I have published the entire script for download but hopefully the above will give you a better understanding of how this works than reading the script alone. There are a few extra features in the full script which are not explained above, such as selecting an appropriate datastore for each VM auto-magically, activating Windows and Office and starting a Windows service. You can disable these features using the $enableFeature parameters at the top of script.

Did you find this useful? I’d love to hear your feedback, or if you are having any issues running the script or a variant of your own leave a commend and I will do my best to help!