---
title: "Office 2016 Install -- \"We need to remove some older apps\""
date: 2015-09-22T00:00:00Z
draft: false
description: "Office 2016 was made available today to the Office 365 subscribers running on PC -- it has been available to Mac users since July. However the installation might not go as smoothly as you would expect, especially if you have some older non-365 Office type applications installed."
tags: ["office", "microsoft", "office365", "installation", "troubleshooting"]
categories: ["uncategorized"]
author: "Harry John"
comments: true
---

## "We need to remove some older apps"

If you're reading this you've probably already download the installer and have noticed a scary warning when starting the install: "We need to remove some older apps".

![Office 2016 Install - We need to remove some older apps](/images/Office-2016-Remove-Some-Older-Apps-300x234.png)

If you haven't see this warning yet here's the deal. Apparently some older apps, such as Visio Professional 2013 and Skype for Business 2015 don't work with Office 2016. If you continue the installation, these "incompatible" apps will be removed and you won't be able to install the old version again.

## What now?

Microsoft's official support on the matter is here: https://support.office.com/en-ie/article/-We-need-to-remove-some-older-apps-error-a225347f-e102-4ea6-9796-5d1ac5220c3b

This seems to mostly apply to Visio Professional 2013. However this morning I have seen a case where Skype and OneDrive must also be removed. Worse, you are not able to reinstall them following the upgrade. Further more, once you've upgraded, how do you go back to Office 2013?

In my case, I was able to install Visio 2016 as I am licensed through Office 365. What about users who are not, and purchased a retail copy of Visio 2013 in the past? Have Microsoft really forced those users to either or re-purchase Visio or miss out on Office 2016? Also, will Microsoft force Office 2013 users to upgrade to 2016 at some point, based on the Office 365 subscription model?

(Update: It seems Microsoft will start pushing updates in October via Windows Update -- optionally, however users will have one year to update. Read this ZDNet article about [the new rules for the rollout](http://www.zdnet.com/article/microsofts-office-2016-the-new-rules-for-the-rollout-starting-september-22/) and this Microsoft article for administrators: [Prepare to update Office 365 ProPlus](https://technet.microsoft.com/en-us/library/mt422981.aspx))

There are some questions which need answering here. I currently have a support case open with Microsoft specifically around the Skype/Onedrive issue. They have informed me that no-one else has reported this and will get back to me in the next 24 hours with a solution. I will of course update here with the solution when I hear back.

## Update: Microsoft confirmed an issue with Skype for Business

Microsoft have responded to my support case around Skype for Business. In brief, Skype for Business is not bundled with Office 2016 and is installed separately, however, the link they provide to install Skype for Business gives you the installer for the 2015 version, which is not compatible with Office 2016. This is their mistake, they are currently looking to update the link to provide the version compatible with 2016.

**They have provided a workaround to get the newest Skype installed until they have fixed the link routing, available here: https://community.office365.com/en-us/w/lync/skype-for-business-is-removed-when-you-upgrade-to-office-2016**

Here is their response in full:

> Current Status: Engineers have validated the solution will mitigate user impact and will be initiating the fix deployment process.
>
> User Experience: Affected users who have upgraded to Office 2016 are unable to install Skype for Business 2016. The Skype for Business 2016 installation URL located on the Office 365 homepage is not routing affected users to the newest version of Skype for Business. As a result, some users may receive the error message: "We have detected that you have a newer version of Office installed on your device. If you want to install an older version of Office, please remove these newer products and try again."
>
> As a workaround, the affected users can download the Skype for Business 2016 client at https://community.office365.com/en-us/w/lync/skype-for-business-is-removed-when-you-upgrade-to-office-2016
>
> Customer Impact: Impact is isolated to customers who have purchased an Office 365 business Premium subscription and have upgraded to Office 2016. Analysis indicates that the scope of customers experiencing impact appears to be very limited with only a few customers reporting the issue.

How did your Office 2016 upgrade go? If anyone else has had similar issues, I'd love to hear from you. Leave a comment below. 