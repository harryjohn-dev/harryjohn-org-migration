:::::::::: {#branding role="banner"}
:::::: {#inner-header .clearfix}
:::: {#site-heading}
::: {#site-title}
[Harry John](http://www.harryjohn.org/ "Harry John"){rel="home"}
:::
::::

<div>

Search for:

</div>
::::::

::::: {#inner-nav .clearfix}
# Main menu {#main-menu .assistive-text .section-heading}

::: {.skip-link .screen-reader-text}
[Skip to content](#content "Skip to content")
:::

::: menu
-   [Home](http://www.harryjohn.org/)
-   [About](http://www.harryjohn.org/about/)
:::
:::::
::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: {#container}
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: {#content .clearfix}
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: {#main .col620 .clearfix role="main"}
:::: post-meta-style
::: post-date-style
[Jan 06,
2014](http://www.harryjohn.org/deploy-multiple-vms-from-template/){rel="bookmark"}
:::

by [Harry
John](http://www.harryjohn.org/author/harry/ "View all posts by Harry John"){.url
.fn .n rel="author"}

in [PowerCLI](http://www.harryjohn.org/category/powercli/){rel="tag"},
[VMware](http://www.harryjohn.org/category/vmware/){rel="tag"}

[[Comment]{.white}
(1)](http://www.harryjohn.org/deploy-multiple-vms-from-template/#comments)
::::

::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: content-wrap
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: post-wrap
::: entry-header
# Deploy multiple VMs from template with PowerCLI {#deploy-multiple-vms-from-template-with-powercli .entry-title}
:::

::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.entry-content .post_content}
:::::::::: e-mailit_top_toolbox
::::::::: {.e-mailit_toolbox .square .size32 emailit-url="http://www.harryjohn.org/deploy-multiple-vms-from-template/" emailit-title="Deploy multiple VMs from template with PowerCLI"}
::: e-mailit_btn_Facebook
:::

::: e-mailit_btn_Twitter
:::

::: e-mailit_btn_Send_via_Email
:::

::: e-mailit_btn_Pinterest
:::

::: e-mailit_btn_LinkedIn
:::

::: e-mailit_btn_EMAILiT
:::
:::::::::
::::::::::

[![Deploy multiple VMs from
template](http://www.harryjohn.org/wp-content/uploads/2014/01/deploy-multiple-vms-from-template-300x211.png){.alignright
.size-medium .wp-image-231 width="300" height="211"
srcset="http://www.harryjohn.org/wp-content/uploads/2014/01/deploy-multiple-vms-from-template-300x211.png 300w, http://www.harryjohn.org/wp-content/uploads/2014/01/deploy-multiple-vms-from-template-624x439.png 624w, http://www.harryjohn.org/wp-content/uploads/2014/01/deploy-multiple-vms-from-template.png 749w"
sizes="(max-width: 300px) 100vw, 300px"}](http://www.harryjohn.org/wp-content/uploads/2014/01/deploy-multiple-vms-from-template.png)I
frequently need to deploy multiple VMs from a template in large numbers,
most often a bunch of Citrix XenApp or XenDesktop servers following an
update to the gold image. Without an automated method deployment can
become a headache and would take days to deploy multiple VMs, especially
when taking into account the post-deploy tasks such as activating
Windows, activating Office, installing Citrix, putting the computer into
the right OU etc.

Rather than consuming large amounts of paracetamol I have written a
(fairly long) PowerCLI script to remove as many of the manual and
mundane tasks as I could. In this blog post I am going to run through
the script and explain how it works. You may download the complete
script and use it "out-the-box" or make your own modifications,
otherwise read on to understand how I deploy multiple VMs from template
using PowerCLI.[]{#more-204}

The script currently does the following:

-   Deploys multiple VMs from template
-   Chooses the best datastores to deploy multiple VMs based on capacity
    and number of VMs/datastore
-   Powers on the VM
-   Puts the VM in the correct OU within AD
-   Activates Windows
-   Activates Office
-   Changes IP address
-   Starts a custom service (I used this to start IMA service which adds
    the VM to Citrix AppCenter)

The complete script is currently approximately 700 lines in total and
you can [download the complete
script](http://www.harryjohn.org/wp-content/uploads/2014/01/DeployMultipleVMs1.zip),
however for the educational purpose of this article I will just go
through some major snippets. Hopefully this will help give you some
ideas or to understand the basics of how this works.

In this blog post I will cover the following aspects of the script:

-   How to deploy multiple VMs from template simultaneously
-   Keeping track of VMs as they go through the build process
-   Changing the IP address of a VM once deployed

## Deploy multiple VMs from template simultaneously

To deploy multiple VMs from template simultaneously I make use of
[PowerShell
Jobs](http://blogs.technet.com/b/heyscriptingguy/archive/2012/12/31/using-windows-powershell-jobs.aspx). For
a complete understanding of how this works read through ScriptingGuy's
excellent blog post. I will be using just two of these commands;
Start-Job and Get-Job

Now, when invoking the Start-Job command a new hidden PowerShell
instance will be launched in the background and runs the job within
here. There are a few things to note;

1.  A new PowerCLI instance means a new session to vCenter, you will
    need to forward the current session to each instance/job you start
2.  If you have a PowerShell profile script, this script will run for
    each instance/job you start

I am not going to cover [Multi-threading
PowerCLI](http://velemental.com/2012/03/11/multithreading-powercli/) as
this has been explained very well on the vElemental blog, I suggest you
check it out for a full understanding of how this works. For now just
know that I will be using two .ps1 files -- the "deploy VM script" which
will be invoked by what I like to call the "control script", every time
we want to deploy another VM simultaneously.

## The deploy VM script

This script is simple. Other than the bit of work required in passing
the vCenter session credentials over from the "control script", all we
need to do here is create a mini PowerCLI script to deploy one VM. So
first, lets gather some parameters about the VM we are to deploy:

:::::::::::::::::::: {#crayon-5a56046b0f12c953320872 .crayon-syntax .crayon-theme-classic .crayon-font-monaco .crayon-os-pc .print-yes .notranslate settings=" minimize scroll-mouseover wrap" style=" margin-top: 12px; margin-bottom: 12px; font-size: 12px !important; line-height: 15px !important;"}
:::::::::::::::: {.crayon-toolbar settings=" mouseover overlay hide delay" style="font-size: 12px !important;height: 18px !important; line-height: 18px !important;"}
[]{.crayon-title}

::::::::::::::: {.crayon-tools style="font-size: 12px !important;height: 18px !important; line-height: 18px !important;"}
:::: {.crayon-button .crayon-nums-button title="Toggle Line Numbers"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-plain-button title="Toggle Plain Code"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-wrap-button title="Toggle Line Wrap"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-expand-button title="Expand Code"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-copy-button title="Copy"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-popup-button title="Open Code In New Window"}
::: crayon-button-icon
:::
::::

[PowerShell]{.crayon-language}
:::::::::::::::
::::::::::::::::

::: {.crayon-info style="min-height: 16.8px !important; line-height: 16.8px !important;"}
:::

::: crayon-plain-wrap
:::

::: {.crayon-main style=""}
+-----------------------------------+-----------------------------------+
| :::: {.crayon-nums-content        | :::: {.crayon-pr                  |
| style="font-size: 12px !important | e style="font-size: 12px !importa |
| ; line-height: 15px !important;"} | nt; line-height: 15px !important; |
| ::: {.crayon-num line="           |  -moz-tab-size:4; -o-tab-size:4;  |
| crayon-5a56046b0f12c953320872-1"} | -webkit-tab-size:4; tab-size:4;"} |
| 1                                 | ::: {#crayon-5a560                |
| :::                               | 46b0f12c953320872-1 .crayon-line} |
| ::::                              | [Param]{.crayon-st}[              |
|                                   | ]{.crayon-h}[(]                   |
|                                   | {.crayon-sy}[\$session]{.crayon-v |
|                                   | }[=]{.crayon-o}[\$]{.crayon-sy}[( |
|                                   | ]{.crayon-sy}[throw]{.crayon-st}[ |
|                                   | ]{.crayon-h}[\"missing -session   |
|                                   | parameter\"]{.                    |
|                                   | crayon-s}[)]{.crayon-sy}[,]{.cray |
|                                   | on-sy}[\$vcserver]{.crayon-v}[,]{ |
|                                   | .crayon-sy}[\$name]{.crayon-v}[,] |
|                                   | {.crayon-sy}[\$datastore]{.crayon |
|                                   | -v}[,]{.crayon-sy}[\$ipaddr]{.cra |
|                                   | yon-v}[,]{.crayon-sy}[\$template] |
|                                   | {.crayon-v}[,]{.crayon-sy}[\$fold |
|                                   | erName]{.crayon-v}[)]{.crayon-sy} |
|                                   | :::                               |
|                                   | ::::                              |
+-----------------------------------+-----------------------------------+
:::
::::::::::::::::::::

There are just a few more parameters I will need, the host/cluster I am
to deploy to, the customisation spec and the true VMware folder
location:

:::::::::::::::::::: {#crayon-5a56046b0f137046219097 .crayon-syntax .crayon-theme-classic .crayon-font-monaco .crayon-os-pc .print-yes .notranslate settings=" minimize scroll-mouseover" style=" margin-top: 12px; margin-bottom: 12px; font-size: 12px !important; line-height: 15px !important;"}
:::::::::::::::: {.crayon-toolbar settings=" mouseover overlay hide delay" style="font-size: 12px !important;height: 18px !important; line-height: 18px !important;"}
[]{.crayon-title}

::::::::::::::: {.crayon-tools style="font-size: 12px !important;height: 18px !important; line-height: 18px !important;"}
:::: {.crayon-button .crayon-nums-button title="Toggle Line Numbers"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-plain-button title="Toggle Plain Code"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-wrap-button title="Toggle Line Wrap"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-expand-button title="Expand Code"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-copy-button title="Copy"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-popup-button title="Open Code In New Window"}
::: crayon-button-icon
:::
::::

[PowerShell]{.crayon-language}
:::::::::::::::
::::::::::::::::

::: {.crayon-info style="min-height: 16.8px !important; line-height: 16.8px !important;"}
:::

::: crayon-plain-wrap
:::

::: {.crayon-main style=""}
+-----------------------------------+-----------------------------------+
| ::::                              | :::::::::::::: {.crayon-pr        |
| :::::::::: {.crayon-nums-content  | e style="font-size: 12px !importa |
| style="font-size: 12px !important | nt; line-height: 15px !important; |
| ; line-height: 15px !important;"} |  -moz-tab-size:4; -o-tab-size:4;  |
| ::: {.crayon-num line="           | -webkit-tab-size:4; tab-size:4;"} |
| crayon-5a56046b0f137046219097-1"} | ::: {#crayon-5a560                |
| 1                                 | 46b0f137046219097-1 .crayon-line} |
| :::                               | [#==================              |
|                                   | ================================= |
| ::: {.cray                        | =====================]{.crayon-c} |
| on-num .crayon-striped-num line=" | :::                               |
| crayon-5a56046b0f137046219097-2"} |                                   |
| 2                                 | ::: {#                            |
| :::                               | crayon-5a56046b0f137046219097-2 . |
|                                   | crayon-line .crayon-striped-line} |
| ::: {.crayon-num line="           | [\# PARAMETERS]{.crayon-c}        |
| crayon-5a56046b0f137046219097-3"} | :::                               |
| 3                                 |                                   |
| :::                               | ::: {#crayon-5a560                |
|                                   | 46b0f137046219097-3 .crayon-line} |
| ::: {.cray                        | [#==================              |
| on-num .crayon-striped-num line=" | ================================= |
| crayon-5a56046b0f137046219097-4"} | =====================]{.crayon-c} |
| 4                                 | :::                               |
| :::                               |                                   |
|                                   | ::: {#                            |
| ::: {.crayon-num line="           | crayon-5a56046b0f137046219097-4 . |
| crayon-5a56046b0f137046219097-5"} | crayon-line .crayon-striped-line} |
| 5                                 | [\# host/cluster]{.crayon-c}      |
| :::                               | :::                               |
|                                   |                                   |
| ::: {.cray                        | ::: {#crayon-5a560                |
| on-num .crayon-striped-num line=" | 46b0f137046219097-5 .crayon-line} |
| crayon-5a56046b0f137046219097-6"} | [\$vmhost]{.crayon-v}[            |
| 6                                 | ]{.crayon-h}[=]{.crayon-o}[       |
| :::                               | ]{.c                              |
|                                   | rayon-h}[\"cluster1\"]{.crayon-s} |
| ::: {.crayon-num line="           | :::                               |
| crayon-5a56046b0f137046219097-7"} |                                   |
| 7                                 | ::: {#                            |
| :::                               | crayon-5a56046b0f137046219097-6 . |
|                                   | crayon-line .crayon-striped-line} |
| ::: {.cray                        |                                   |
| on-num .crayon-striped-num line=" | :::                               |
| crayon-5a56046b0f137046219097-8"} |                                   |
| 8                                 | ::: {#crayon-5a560                |
| :::                               | 46b0f137046219097-7 .crayon-line} |
|                                   | [\# customisation                 |
| ::: {.crayon-num line="           | spec]{.crayon-c}                  |
| crayon-5a56046b0f137046219097-9"} | :::                               |
| 9                                 |                                   |
| :::                               | ::: {#                            |
|                                   | crayon-5a56046b0f137046219097-8 . |
| ::: {.crayo                       | crayon-line .crayon-striped-line} |
| n-num .crayon-striped-num line="c | [\$custSpecName]{.crayon-v}[      |
| rayon-5a56046b0f137046219097-10"} | ]{.crayon-h}[=]{.crayon-o}[       |
| 10                                | ]{.cray                           |
| :::                               | on-h}[\"W2K8-R2-X64\"]{.crayon-s} |
|                                   | :::                               |
| ::: {.crayon-num line="c          |                                   |
| rayon-5a56046b0f137046219097-11"} | ::: {#crayon-5a560                |
| 11                                | 46b0f137046219097-9 .crayon-line} |
| :::                               |                                   |
| ::::::::::::::                    | :::                               |
|                                   |                                   |
|                                   | ::: {#c                           |
|                                   | rayon-5a56046b0f137046219097-10 . |
|                                   | crayon-line .crayon-striped-line} |
|                                   | [\# location]{.crayon-c}          |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#crayon-5a5604               |
|                                   | 6b0f137046219097-11 .crayon-line} |
|                                   | [\$folder]{.crayon-v}[            |
|                                   | ]{.crayon-h}[=]{.crayon-o}[       |
|                                   | ]{.                               |
|                                   | crayon-h}[Get-Folder]{.crayon-r}[ |
|                                   | ]{.                               |
|                                   | crayon-h}[-Location]{.crayon-cn}[ |
|                                   | ]{.crayon-h}[\"Virtual            |
|                                   | Machines\"]{.crayon-s}[           |
|                                   | ]{.crayon-h}[-Name]{.crayon-cn}[  |
|                                   | ]{.c                              |
|                                   | rayon-h}[\$folderName]{.crayon-v} |
|                                   | :::                               |
|                                   | ::::::::::::::                    |
+-----------------------------------+-----------------------------------+
:::
::::::::::::::::::::

There is something important to note about customisation specs. I have
set my customisation spec to use DHCP and I will be applying a static IP
once the VM has deployed, powered on and been customised. One method
many people have discussed is to clone your main customisation spec into
a new "temporary spec", add the IP address details into this temporary
spec and delete after you have deployed a VM with it. I just didn't get
along with this method. It would be nice if there was a way we could
supply the details within PowerCLI for which you are prompted for when
using the vSphere Client, as far as I know this is not possible.

Finally, we need the command to deploy a VM using the parameters we have
collected above.

:::::::::::::::::::: {#crayon-5a56046b0f141261100978 .crayon-syntax .crayon-theme-classic .crayon-font-monaco .crayon-os-pc .print-yes .notranslate settings=" minimize scroll-mouseover" style=" margin-top: 12px; margin-bottom: 12px; font-size: 12px !important; line-height: 15px !important;"}
:::::::::::::::: {.crayon-toolbar settings=" mouseover overlay hide delay" style="font-size: 12px !important;height: 18px !important; line-height: 18px !important;"}
[]{.crayon-title}

::::::::::::::: {.crayon-tools style="font-size: 12px !important;height: 18px !important; line-height: 18px !important;"}
:::: {.crayon-button .crayon-nums-button title="Toggle Line Numbers"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-plain-button title="Toggle Plain Code"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-wrap-button title="Toggle Line Wrap"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-expand-button title="Expand Code"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-copy-button title="Copy"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-popup-button title="Open Code In New Window"}
::: crayon-button-icon
:::
::::

[PowerShell]{.crayon-language}
:::::::::::::::
::::::::::::::::

::: {.crayon-info style="min-height: 16.8px !important; line-height: 16.8px !important;"}
:::

::: crayon-plain-wrap
:::

::: {.crayon-main style=""}
+-----------------------------------+-----------------------------------+
| ::::::                            | :::::::::::::::: {.crayon-pr      |
| :::::::::: {.crayon-nums-content  | e style="font-size: 12px !importa |
| style="font-size: 12px !important | nt; line-height: 15px !important; |
| ; line-height: 15px !important;"} |  -moz-tab-size:4; -o-tab-size:4;  |
| ::: {.crayon-num line="           | -webkit-tab-size:4; tab-size:4;"} |
| crayon-5a56046b0f141261100978-1"} | ::: {#crayon-5a560                |
| 1                                 | 46b0f141261100978-1 .crayon-line} |
| :::                               | [try]{.crayon-e}[                 |
|                                   | ]{.crayon-h}[{]{.crayon-sy}       |
| ::: {.cray                        | :::                               |
| on-num .crayon-striped-num line=" |                                   |
| crayon-5a56046b0f141261100978-2"} | ::: {#                            |
| 2                                 | crayon-5a56046b0f141261100978-2 . |
| :::                               | crayon-line .crayon-striped-line} |
|                                   | [\$vm]{.crayon-v}[                |
| ::: {.crayon-num line="           | ]{.crayon-h}[=]{.crayon-o}[       |
| crayon-5a56046b0f141261100978-3"} | ]{.crayon-h}[New-VM]{.crayon-r}[  |
| 3                                 | ]{.crayon-h}[-Name]{.crayon-cn}[  |
| :::                               | ]{.crayon-h}[\$name]{.crayon-v}[  |
|                                   | ]{.crayon-h}[\`]{.crayon-sy}      |
| ::: {.cray                        | :::                               |
| on-num .crayon-striped-num line=" |                                   |
| crayon-5a56046b0f141261100978-4"} | ::: {#crayon-5a560                |
| 4                                 | 46b0f141261100978-3 .crayon-line} |
| :::                               | [                                 |
|                                   | ]                                 |
| ::: {.crayon-num line="           | {.crayon-h}[-VMHost]{.crayon-cn}[ |
| crayon-5a56046b0f141261100978-5"} | ]                                 |
| 5                                 | {.crayon-h}[\$vmhost]{.crayon-v}[ |
| :::                               | ]{.crayon-h}[\`]{.crayon-sy}      |
|                                   | :::                               |
| ::: {.cray                        |                                   |
| on-num .crayon-striped-num line=" | ::: {#                            |
| crayon-5a56046b0f141261100978-6"} | crayon-5a56046b0f141261100978-4 . |
| 6                                 | crayon-line .crayon-striped-line} |
| :::                               | [                                 |
|                                   | ]{.                               |
| ::: {.crayon-num line="           | crayon-h}[-Template]{.crayon-cn}[ |
| crayon-5a56046b0f141261100978-7"} | ]{.                               |
| 7                                 | crayon-h}[\$template]{.crayon-v}[ |
| :::                               | ]{.crayon-h}[\`]{.crayon-sy}      |
|                                   | :::                               |
| ::: {.cray                        |                                   |
| on-num .crayon-striped-num line=" | ::: {#crayon-5a560                |
| crayon-5a56046b0f141261100978-8"} | 46b0f141261100978-5 .crayon-line} |
| 8                                 | [                                 |
| :::                               | ]{.crayon-h}[-                    |
|                                   | OSCustomizationSpec]{.crayon-cn}[ |
| ::: {.crayon-num line="           | ]{.                               |
| crayon-5a56046b0f141261100978-9"} | crayon-h}[\$custSpec]{.crayon-v}[ |
| 9                                 | ]{.crayon-h}[\`]{.crayon-sy}      |
| :::                               | :::                               |
|                                   |                                   |
| ::: {.crayo                       | ::: {#                            |
| n-num .crayon-striped-num line="c | crayon-5a56046b0f141261100978-6 . |
| rayon-5a56046b0f141261100978-10"} | crayon-line .crayon-striped-line} |
| 10                                | [                                 |
| :::                               | ]{.c                              |
|                                   | rayon-h}[-Datastore]{.crayon-cn}[ |
| ::: {.crayon-num line="c          | ]{.c                              |
| rayon-5a56046b0f141261100978-11"} | rayon-h}[\$datastore]{.crayon-v}[ |
| 11                                | ]{.crayon-h}[\`]{.crayon-sy}      |
| :::                               | :::                               |
|                                   |                                   |
| ::: {.crayo                       | ::: {#crayon-5a560                |
| n-num .crayon-striped-num line="c | 46b0f141261100978-7 .crayon-line} |
| rayon-5a56046b0f141261100978-12"} | [                                 |
| 12                                | ]{.                               |
| :::                               | crayon-h}[-Location]{.crayon-cn}[ |
|                                   | ]                                 |
| ::: {.crayon-num line="c          | {.crayon-h}[\$folder]{.crayon-v}[ |
| rayon-5a56046b0f141261100978-13"} | ]{.crayon-h}[\`]{.crayon-sy}      |
| 13                                | :::                               |
| :::                               |                                   |
| ::::::::::::::::                  | ::: {#                            |
|                                   | crayon-5a56046b0f141261100978-8 . |
|                                   | crayon-line .crayon-striped-line} |
|                                   | [                                 |
|                                   | ]{.                               |
|                                   | crayon-h}[-ErrorAction]{.crayon-c |
|                                   | n}[:]{.crayon-o}[Stop]{.crayon-i} |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#crayon-5a560                |
|                                   | 46b0f141261100978-9 .crayon-line} |
|                                   | [}]{.crayon-sy}                   |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#c                           |
|                                   | rayon-5a56046b0f141261100978-10 . |
|                                   | crayon-line .crayon-striped-line} |
|                                   | [catch]{.crayon-e}                |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#crayon-5a5604               |
|                                   | 6b0f141261100978-11 .crayon-line} |
|                                   | [{]{.crayon-sy}                   |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#c                           |
|                                   | rayon-5a56046b0f141261100978-12 . |
|                                   | crayon-line .crayon-striped-line} |
|                                   | [\# your catch                    |
|                                   | method\....]{.crayon-c}           |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#crayon-5a5604               |
|                                   | 6b0f141261100978-13 .crayon-line} |
|                                   | [}]{.crayon-sy}                   |
|                                   | :::                               |
|                                   | ::::::::::::::::                  |
+-----------------------------------+-----------------------------------+
:::
::::::::::::::::::::

That's it for the deploy VM script, I told you it was simple!

## The control script

Getting slightly more complicated, the control script is the "brain" of
the process and, you guessed it, controls the deployment of multiple VMs
as well as completing any other post-deployment tasks. Lets just cover
the basics.

First we must gather some information about the VMs we are to deploy.
Some information I like to "hard code" in parameters at the top of the
script such as the template name, DNS servers and gateway address. Other
information I like to collect interactively when at run-time such as VM
names, IP addresses, number of VMs. Here is an example:

:::::::::::::::::::: {#crayon-5a56046b0f148493071767 .crayon-syntax .crayon-theme-classic .crayon-font-monaco .crayon-os-pc .print-yes .notranslate settings=" minimize scroll-mouseover wrap" style=" margin-top: 12px; margin-bottom: 12px; font-size: 12px !important; line-height: 15px !important;"}
:::::::::::::::: {.crayon-toolbar settings=" mouseover overlay hide delay" style="font-size: 12px !important;height: 18px !important; line-height: 18px !important;"}
[]{.crayon-title}

::::::::::::::: {.crayon-tools style="font-size: 12px !important;height: 18px !important; line-height: 18px !important;"}
:::: {.crayon-button .crayon-nums-button title="Toggle Line Numbers"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-plain-button title="Toggle Plain Code"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-wrap-button title="Toggle Line Wrap"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-expand-button title="Expand Code"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-copy-button title="Copy"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-popup-button title="Open Code In New Window"}
::: crayon-button-icon
:::
::::

[PowerShell]{.crayon-language}
:::::::::::::::
::::::::::::::::

::: {.crayon-info style="min-height: 16.8px !important; line-height: 16.8px !important;"}
:::

::: crayon-plain-wrap
:::

::: {.crayon-main style=""}
+-----------------------------------+-----------------------------------+
| ::::::::::::                      | :                                 |
| :::::::::: {.crayon-nums-content  | ::::::::::::::::::::: {.crayon-pr |
| style="font-size: 12px !important | e style="font-size: 12px !importa |
| ; line-height: 15px !important;"} | nt; line-height: 15px !important; |
| ::: {.crayon-num line="           |  -moz-tab-size:4; -o-tab-size:4;  |
| crayon-5a56046b0f148493071767-1"} | -webkit-tab-size:4; tab-size:4;"} |
| 1                                 | ::: {#crayon-5a560                |
| :::                               | 46b0f148493071767-1 .crayon-line} |
|                                   | [\# HARD CODED                    |
| ::: {.cray                        | PARAMETERS]{.crayon-c}            |
| on-num .crayon-striped-num line=" | :::                               |
| crayon-5a56046b0f148493071767-2"} |                                   |
| 2                                 | ::: {#                            |
| :::                               | crayon-5a56046b0f148493071767-2 . |
|                                   | crayon-line .crayon-striped-line} |
| ::: {.crayon-num line="           | [\# vm settings]{.crayon-c}       |
| crayon-5a56046b0f148493071767-3"} | :::                               |
| 3                                 |                                   |
| :::                               | ::: {#crayon-5a560                |
|                                   | 46b0f148493071767-3 .crayon-line} |
| ::: {.cray                        | [\$folder]{.crayon-v}[            |
| on-num .crayon-striped-num line=" | ]{.crayon-h}[=]{.crayon-o}[       |
| crayon-5a56046b0f148493071767-4"} | ]{.crayon-h}[\"Citrix-NG          |
| 4                                 | Servers\"]{.crayon-s}             |
| :::                               | :::                               |
|                                   |                                   |
| ::: {.crayon-num line="           | ::: {#                            |
| crayon-5a56046b0f148493071767-5"} | crayon-5a56046b0f148493071767-4 . |
| 5                                 | crayon-line .crayon-striped-line} |
| :::                               | [\$template]{.crayon-v}[          |
|                                   | ]{.crayon-h}[=]{.crayon-o}[       |
| ::: {.cray                        | ]{.crayon-h}[\'Citrix XenApp 6.5  |
| on-num .crayon-striped-num line=" | Farm Member Template (Citrix      |
| crayon-5a56046b0f148493071767-6"} | Installed)\']{.crayon-s}          |
| 6                                 | :::                               |
| :::                               |                                   |
|                                   | ::: {#crayon-5a560                |
| ::: {.crayon-num line="           | 46b0f148493071767-5 .crayon-line} |
| crayon-5a56046b0f148493071767-7"} |                                   |
| 7                                 | :::                               |
| :::                               |                                   |
|                                   | ::: {#                            |
| ::: {.cray                        | crayon-5a56046b0f148493071767-6 . |
| on-num .crayon-striped-num line=" | crayon-line .crayon-striped-line} |
| crayon-5a56046b0f148493071767-8"} | [\# network variables]{.crayon-c} |
| 8                                 | :::                               |
| :::                               |                                   |
|                                   | ::: {#crayon-5a560                |
| ::: {.crayon-num line="           | 46b0f148493071767-7 .crayon-line} |
| crayon-5a56046b0f148493071767-9"} | [\$networkSubnet]{.crayon-v}[     |
| 9                                 | ]{.crayon-h}[=]{.crayon-o}[       |
| :::                               | ]{.cray                           |
|                                   | on-h}[\"255.255.0.0\"]{.crayon-s} |
| ::: {.crayo                       | :::                               |
| n-num .crayon-striped-num line="c |                                   |
| rayon-5a56046b0f148493071767-10"} | ::: {#                            |
| 10                                | crayon-5a56046b0f148493071767-8 . |
| :::                               | crayon-line .crayon-striped-line} |
|                                   | [\$networkGateway]{.crayon-v}[    |
| ::: {.crayon-num line="c          | ]{.crayon-h}[=]{.crayon-o}[       |
| rayon-5a56046b0f148493071767-11"} | ]{.cra                            |
| 11                                | yon-h}[\"172.16.0.1\"]{.crayon-s} |
| :::                               | :::                               |
|                                   |                                   |
| ::: {.crayo                       | ::: {#crayon-5a560                |
| n-num .crayon-striped-num line="c | 46b0f148493071767-9 .crayon-line} |
| rayon-5a56046b0f148493071767-12"} | [\$networkDns]{.crayon-v}[        |
| 12                                | ]{.crayon-h}[=]{.crayon-o}[       |
| :::                               | ]{.crayon-h}[\"                   |
|                                   | 172.16.0.2\"]{.crayon-s}[,]{.cray |
| ::: {.crayon-num line="c          | on-sy}[\"172.16.0.3\"]{.crayon-s} |
| rayon-5a56046b0f148493071767-13"} | :::                               |
| 13                                |                                   |
| :::                               | ::: {#c                           |
|                                   | rayon-5a56046b0f148493071767-10 . |
| ::: {.crayo                       | crayon-line .crayon-striped-line} |
| n-num .crayon-striped-num line="c |                                   |
| rayon-5a56046b0f148493071767-14"} | :::                               |
| 14                                |                                   |
| :::                               | ::: {#crayon-5a5604               |
|                                   | 6b0f148493071767-11 .crayon-line} |
| ::: {.crayon-num line="c          | [\# GATHER FROM USER              |
| rayon-5a56046b0f148493071767-15"} | PARAMETERS]{.crayon-c}            |
| 15                                | :::                               |
| :::                               |                                   |
|                                   | ::: {#c                           |
| ::: {.crayo                       | rayon-5a56046b0f148493071767-12 . |
| n-num .crayon-striped-num line="c | crayon-line .crayon-striped-line} |
| rayon-5a56046b0f148493071767-16"} | [\# get VM name                   |
| 16                                | prefix]{.crayon-c}                |
| :::                               | :::                               |
|                                   |                                   |
| ::: {.crayon-num line="c          | ::: {#crayon-5a5604               |
| rayon-5a56046b0f148493071767-17"} | 6b0f148493071767-13 .crayon-line} |
| 17                                | [\$vmFirstName]{.crayon-v}[       |
| :::                               | ]{.crayon-h}[=]{.crayon-o}[       |
|                                   | ]{                                |
| ::: {.crayo                       | .crayon-h}[Read-Host]{.crayon-r}[ |
| n-num .crayon-striped-num line="c | ]{.crayon-h}[\"Enter the name of  |
| rayon-5a56046b0f148493071767-18"} | the first VM (e.g.                |
| 18                                | MAILSERVER01)\"]{.crayon-s}       |
| :::                               | :::                               |
|                                   |                                   |
| ::: {.crayon-num line="c          | ::: {#c                           |
| rayon-5a56046b0f148493071767-19"} | rayon-5a56046b0f148493071767-14 . |
| 19                                | crayon-line .crayon-striped-line} |
| :::                               |                                   |
| ::::::::::::::::::::::            | :::                               |
|                                   |                                   |
|                                   | ::: {#crayon-5a5604               |
|                                   | 6b0f148493071767-15 .crayon-line} |
|                                   | [\# get number of VMs to          |
|                                   | build]{.crayon-c}                 |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#c                           |
|                                   | rayon-5a56046b0f148493071767-16 . |
|                                   | crayon-line .crayon-striped-line} |
|                                   | [\[]{.cr                          |
|                                   | ayon-sy}[int]{.crayon-t}[\]]{.cra |
|                                   | yon-sy}[\$vmQuantity]{.crayon-v}[ |
|                                   | ]{.crayon-h}[=]{.crayon-o}[       |
|                                   | ]{                                |
|                                   | .crayon-h}[Read-Host]{.crayon-r}[ |
|                                   | ]{.crayon-h}[\"How many VMs are   |
|                                   | you deploying?\"]{.crayon-s}      |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#crayon-5a5604               |
|                                   | 6b0f148493071767-17 .crayon-line} |
|                                   |                                   |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#c                           |
|                                   | rayon-5a56046b0f148493071767-18 . |
|                                   | crayon-line .crayon-striped-line} |
|                                   | [\# get IP Address ]{.crayon-c}   |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#crayon-5a5604               |
|                                   | 6b0f148493071767-19 .crayon-line} |
|                                   | [\$ipAddress]{.crayon-v}[         |
|                                   | ]{.crayon-h}[=]{.crayon-o}[       |
|                                   | ]{                                |
|                                   | .crayon-h}[Read-Host]{.crayon-r}[ |
|                                   | ]{.crayon-h}[\"You need a block   |
|                                   | of \$vmQuantity adjacent IP       |
|                                   | addresses, enter the              |
|                                   | first\"]{.crayon-s}               |
|                                   | :::                               |
|                                   | ::::::::::::::::::::::            |
+-----------------------------------+-----------------------------------+
:::
::::::::::::::::::::

You might have already noticed, when I deploy multiple VMs they will
have the same name but a different number at the end, for example;
MAILSERVER01, MAILSERVER02 etc. I also deploy VMs with a sequence of
adjacent IP addresses for example; 172.16.0.101, 172.16.0.102 etc. This
makes deployment and this script very simple because all I need to know
is the first VM name, the first IP address, and how many VMs to deploy.
From this information I can create a nice PowerShell array with each VM
name and its IP address. Lets split up the first IP address we have
collected to do this:

:::::::::::::::::::: {#crayon-5a56046b0f150546407735 .crayon-syntax .crayon-theme-classic .crayon-font-monaco .crayon-os-pc .print-yes .notranslate settings=" minimize scroll-mouseover" style=" margin-top: 12px; margin-bottom: 12px; font-size: 12px !important; line-height: 15px !important;"}
:::::::::::::::: {.crayon-toolbar settings=" mouseover overlay hide delay" style="font-size: 12px !important;height: 18px !important; line-height: 18px !important;"}
[]{.crayon-title}

::::::::::::::: {.crayon-tools style="font-size: 12px !important;height: 18px !important; line-height: 18px !important;"}
:::: {.crayon-button .crayon-nums-button title="Toggle Line Numbers"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-plain-button title="Toggle Plain Code"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-wrap-button title="Toggle Line Wrap"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-expand-button title="Expand Code"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-copy-button title="Copy"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-popup-button title="Open Code In New Window"}
::: crayon-button-icon
:::
::::

[PowerShell]{.crayon-language}
:::::::::::::::
::::::::::::::::

::: {.crayon-info style="min-height: 16.8px !important; line-height: 16.8px !important;"}
:::

::: crayon-plain-wrap
:::

::: {.crayon-main style=""}
+-----------------------------------+-----------------------------------+
| ::::::: {.crayon-nums-content     | ::::::: {.crayon-pr               |
| style="font-size: 12px !important | e style="font-size: 12px !importa |
| ; line-height: 15px !important;"} | nt; line-height: 15px !important; |
| ::: {.crayon-num line="           |  -moz-tab-size:4; -o-tab-size:4;  |
| crayon-5a56046b0f150546407735-1"} | -webkit-tab-size:4; tab-size:4;"} |
| 1                                 | ::: {#crayon-5a560                |
| :::                               | 46b0f150546407735-1 .crayon-line} |
|                                   | [\# split up IP                   |
| ::: {.cray                        | address]{.crayon-c}               |
| on-num .crayon-striped-num line=" | :::                               |
| crayon-5a56046b0f150546407735-2"} |                                   |
| 2                                 | ::: {#                            |
| :::                               | crayon-5a56046b0f150546407735-2 . |
|                                   | crayon-line .crayon-striped-line} |
| ::: {.crayon-num line="           | [\$ipSplit]{.crayon-v}[           |
| crayon-5a56046b0f150546407735-3"} | ]{.crayon-h}[=]{.crayon-o}[       |
| 3                                 | ]{.crayon-h}[\$ipA                |
| :::                               | ddress]{.crayon-v}[.]{.crayon-sy} |
|                                   | [Split]{.crayon-e}[(]{.crayon-sy} |
| ::: {.cray                        | [\".\"]{.crayon-s}[)]{.crayon-sy} |
| on-num .crayon-striped-num line=" | :::                               |
| crayon-5a56046b0f150546407735-4"} |                                   |
| 4                                 | ::: {#crayon-5a560                |
| :::                               | 46b0f150546407735-3 .crayon-line} |
| :::::::                           | [\$ipPrefix]{.crayon-v}[          |
|                                   | ]{.crayon-h}[=]{.crayon-o}[       |
|                                   | ]{                                |
|                                   | .crayon-h}[\$ipSplit]{.crayon-v}[ |
|                                   | \[]{.crayon-sy}0[\]]{.crayon-sy}[ |
|                                   | ]{.crayon-h}[+]{.crayon-o}[       |
|                                   | ]{.crayon-h}[\".\"]{.crayon-s}[   |
|                                   | ]{.crayon-h}[+]{.crayon-o}[       |
|                                   | ]{                                |
|                                   | .crayon-h}[\$ipSplit]{.crayon-v}[ |
|                                   | \[]{.crayon-sy}1[\]]{.crayon-sy}[ |
|                                   | ]{.crayon-h}[+]{.crayon-o}[       |
|                                   | ]{.crayon-h}[\".\"]{.crayon-s}[   |
|                                   | ]{.crayon-h}[+]{.crayon-o}[       |
|                                   | ]{                                |
|                                   | .crayon-h}[\$ipSplit]{.crayon-v}[ |
|                                   | \[]{.crayon-sy}2[\]]{.crayon-sy}[ |
|                                   | ]{.crayon-h}[+]{.crayon-o}[       |
|                                   | ]{.crayon-h}[\".\"]{.crayon-s}[   |
|                                   | ]{.crayon-h}                      |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#                            |
|                                   | crayon-5a56046b0f150546407735-4 . |
|                                   | crayon-line .crayon-striped-line} |
|                                   | [\[]{.                            |
|                                   | crayon-sy}[int]{.crayon-t}[\]]{.c |
|                                   | rayon-sy}[\$ipSuffix]{.crayon-v}[ |
|                                   | ]{.crayon-h}[=]{.crayon-o}[       |
|                                   | ]                                 |
|                                   | {.crayon-h}[\$ipSplit]{.crayon-v} |
|                                   | [\[]{.crayon-sy}3[\]]{.crayon-sy} |
|                                   | :::                               |
|                                   | :::::::                           |
+-----------------------------------+-----------------------------------+
:::
::::::::::::::::::::

Now I have the IP address prefix, such as 172.16.0 and the first IP
address suffix, such as 101. If I put together the prefix and the suffix
I get the complete IP address: 172.16.0.101. I can add one to the
suffix, put them together again and have the IP address of the next VM:
172.16.0.102. Simples!

Again, I need to do this with the VM name, so take the last two
characters of the name and you have the first VM number (suffix), take
the other characters and you have the VM name (prefix):

:::::::::::::::::::: {#crayon-5a56046b0f157448374595 .crayon-syntax .crayon-theme-classic .crayon-font-monaco .crayon-os-pc .print-yes .notranslate settings=" minimize scroll-mouseover" style=" margin-top: 12px; margin-bottom: 12px; font-size: 12px !important; line-height: 15px !important;"}
:::::::::::::::: {.crayon-toolbar settings=" mouseover overlay hide delay" style="font-size: 12px !important;height: 18px !important; line-height: 18px !important;"}
[]{.crayon-title}

::::::::::::::: {.crayon-tools style="font-size: 12px !important;height: 18px !important; line-height: 18px !important;"}
:::: {.crayon-button .crayon-nums-button title="Toggle Line Numbers"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-plain-button title="Toggle Plain Code"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-wrap-button title="Toggle Line Wrap"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-expand-button title="Expand Code"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-copy-button title="Copy"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-popup-button title="Open Code In New Window"}
::: crayon-button-icon
:::
::::

[PowerShell]{.crayon-language}
:::::::::::::::
::::::::::::::::

::: {.crayon-info style="min-height: 16.8px !important; line-height: 16.8px !important;"}
:::

::: crayon-plain-wrap
:::

::: {.crayon-main style=""}
+-----------------------------------+-----------------------------------+
| :::::: {.crayon-nums-content      | :::::: {.crayon-pr                |
| style="font-size: 12px !important | e style="font-size: 12px !importa |
| ; line-height: 15px !important;"} | nt; line-height: 15px !important; |
| ::: {.crayon-num line="           |  -moz-tab-size:4; -o-tab-size:4;  |
| crayon-5a56046b0f157448374595-1"} | -webkit-tab-size:4; tab-size:4;"} |
| 1                                 | ::: {#crayon-5a560                |
| :::                               | 46b0f157448374595-1 .crayon-line} |
|                                   | [\# split up VM name]{.crayon-c}  |
| ::: {.cray                        | :::                               |
| on-num .crayon-striped-num line=" |                                   |
| crayon-5a56046b0f157448374595-2"} | ::: {#                            |
| 2                                 | crayon-5a56046b0f157448374595-2 . |
| :::                               | crayon-line .crayon-striped-line} |
|                                   | [\[]{.crayo                       |
| ::: {.crayon-num line="           | n-sy}[int]{.crayon-t}[\]]{.crayon |
| crayon-5a56046b0f157448374595-3"} | -sy}[\$vmFirstNumber]{.crayon-v}[ |
| 3                                 | ]{.crayon-h}[=]{.crayon-o}[       |
| :::                               | ]{.crayon-h}[\$vmFi               |
| ::::::                            | rstName]{.crayon-v}[.]{.crayon-sy |
|                                   | }[Substring]{.crayon-e}[(]{.crayo |
|                                   | n-sy}[\$vmFirstName]{.crayon-v}[. |
|                                   | ]{.crayon-sy}[length]{.crayon-i}[ |
|                                   | ]{.crayon-h}[-]{.crayon-o}[       |
|                                   | ]{.crayon-h                       |
|                                   | }2[,]{.crayon-sy}2[)]{.crayon-sy} |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#crayon-5a560                |
|                                   | 46b0f157448374595-3 .crayon-line} |
|                                   | [\$vmNamePrefix]{.crayon-v}[      |
|                                   | ]{.crayon-h}[=]{.crayon-o}[       |
|                                   | ]{                                |
|                                   | .crayon-h}[\$vmFirstName]{.crayon |
|                                   | -v}[.]{.crayon-sy}[Substring]{.cr |
|                                   | ayon-e}[(]{.crayon-sy}0[,]{.crayo |
|                                   | n-sy}[\$vmFirstName]{.crayon-v}[. |
|                                   | ]{.crayon-sy}[length]{.crayon-i}[ |
|                                   | ]{.crayon-h}[-]{.crayon-o}[       |
|                                   | ]{.crayon-h}2[)]{.crayon-sy}      |
|                                   | :::                               |
|                                   | ::::::                            |
+-----------------------------------+-----------------------------------+
:::
::::::::::::::::::::

### Keeping track of VMs as they go through the build process

Now we have collected all the info we need to deploy multiple VMs,
I like to create an array which will keep track of the VMs, tasks and
their status. So lets create a loop to build the VM name and the IP
address using the prefixes and suffixes we created earlier and add each
one to the array. I also add a few other properties for each VM to keep
track of the build status, such as Clone Initiated, Cloned, Powered On
and Re-IP.

:::::::::::::::::::: {#crayon-5a56046b0f15f963307050 .crayon-syntax .crayon-theme-classic .crayon-font-monaco .crayon-os-pc .print-yes .notranslate settings=" minimize scroll-mouseover wrap" style=" margin-top: 12px; margin-bottom: 12px; font-size: 12px !important; line-height: 15px !important;"}
:::::::::::::::: {.crayon-toolbar settings=" mouseover overlay hide delay" style="font-size: 12px !important;height: 18px !important; line-height: 18px !important;"}
[]{.crayon-title}

::::::::::::::: {.crayon-tools style="font-size: 12px !important;height: 18px !important; line-height: 18px !important;"}
:::: {.crayon-button .crayon-nums-button title="Toggle Line Numbers"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-plain-button title="Toggle Plain Code"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-wrap-button title="Toggle Line Wrap"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-expand-button title="Expand Code"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-copy-button title="Copy"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-popup-button title="Open Code In New Window"}
::: crayon-button-icon
:::
::::

[PowerShell]{.crayon-language}
:::::::::::::::
::::::::::::::::

::: {.crayon-info style="min-height: 16.8px !important; line-height: 16.8px !important;"}
:::

::: crayon-plain-wrap
:::

::: {.crayon-main style=""}
+-----------------------------------+-----------------------------------+
| ::::::::::::::::::::::::::::::::: | ::::::::::::::::::::::            |
| :::::::::: {.crayon-nums-content  | ::::::::::::::::::::: {.crayon-pr |
| style="font-size: 12px !important | e style="font-size: 12px !importa |
| ; line-height: 15px !important;"} | nt; line-height: 15px !important; |
| ::: {.crayon-num line="           |  -moz-tab-size:4; -o-tab-size:4;  |
| crayon-5a56046b0f15f963307050-1"} | -webkit-tab-size:4; tab-size:4;"} |
| 1                                 | ::: {#crayon-5a560                |
| :::                               | 46b0f15f963307050-1 .crayon-line} |
|                                   | [\###################]{.crayon-c} |
| ::: {.cray                        | :::                               |
| on-num .crayon-striped-num line=" |                                   |
| crayon-5a56046b0f15f963307050-2"} | ::: {#                            |
| 2                                 | crayon-5a56046b0f15f963307050-2 . |
| :::                               | crayon-line .crayon-striped-line} |
|                                   | [\# Build \$vmStatus array to     |
| ::: {.crayon-num line="           | keep track of VMs and completed   |
| crayon-5a56046b0f15f963307050-3"} | tasks]{.crayon-c}                 |
| 3                                 | :::                               |
| :::                               |                                   |
|                                   | ::: {#crayon-5a560                |
| ::: {.cray                        | 46b0f15f963307050-3 .crayon-line} |
| on-num .crayon-striped-num line=" | [\###################]{.crayon-c} |
| crayon-5a56046b0f15f963307050-4"} | :::                               |
| 4                                 |                                   |
| :::                               | ::: {#                            |
|                                   | crayon-5a56046b0f15f963307050-4 . |
| ::: {.crayon-num line="           | crayon-line .crayon-striped-line} |
| crayon-5a56046b0f15f963307050-5"} |                                   |
| 5                                 | :::                               |
| :::                               |                                   |
|                                   | ::: {#crayon-5a560                |
| ::: {.cray                        | 46b0f15f963307050-5 .crayon-line} |
| on-num .crayon-striped-num line=" | [\# create an object collection   |
| crayon-5a56046b0f15f963307050-6"} | holding status info for all VMs   |
| 6                                 | being built]{.crayon-c}           |
| :::                               | :::                               |
|                                   |                                   |
| ::: {.crayon-num line="           | ::: {#                            |
| crayon-5a56046b0f15f963307050-7"} | crayon-5a56046b0f15f963307050-6 . |
| 7                                 | crayon-line .crayon-striped-line} |
| :::                               | [\$vmStatus]{.crayon-v}[          |
|                                   | ]{.crayon-h}[=]{.crayon-o}[       |
| ::: {.cray                        | ]{.crayon-h}[@]{.crayon-          |
| on-num .crayon-striped-num line=" | sy}[(]{.crayon-sy}[)]{.crayon-sy} |
| crayon-5a56046b0f15f963307050-8"} | :::                               |
| 8                                 |                                   |
| :::                               | ::: {#crayon-5a560                |
|                                   | 46b0f15f963307050-7 .crayon-line} |
| ::: {.crayon-num line="           |                                   |
| crayon-5a56046b0f15f963307050-9"} | :::                               |
| 9                                 |                                   |
| :::                               | ::: {#                            |
|                                   | crayon-5a56046b0f15f963307050-8 . |
| ::: {.crayo                       | crayon-line .crayon-striped-line} |
| n-num .crayon-striped-num line="c | [\# create counters for VM name   |
| rayon-5a56046b0f15f963307050-10"} | and IP                            |
| 10                                | address/datastore]{.crayon-c}     |
| :::                               | :::                               |
|                                   |                                   |
| ::: {.crayon-num line="c          | ::: {#crayon-5a560                |
| rayon-5a56046b0f15f963307050-11"} | 46b0f15f963307050-9 .crayon-line} |
| 11                                | [\$vmNumber]{.crayon-v}[          |
| :::                               | ]{.crayon-h}[=]{.crayon-o}[       |
|                                   | ]{.cray                           |
| ::: {.crayo                       | on-h}[\$vmFirstNumber]{.crayon-v} |
| n-num .crayon-striped-num line="c | :::                               |
| rayon-5a56046b0f15f963307050-12"} |                                   |
| 12                                | ::: {#c                           |
| :::                               | rayon-5a56046b0f15f963307050-10 . |
|                                   | crayon-line .crayon-striped-line} |
| ::: {.crayon-num line="c          | [\$vmCounter]{.crayon-v}[         |
| rayon-5a56046b0f15f963307050-13"} | ]{.crayon-h}[=]{.crayon-o}[       |
| 13                                | ]{.crayon-h}0                     |
| :::                               | :::                               |
|                                   |                                   |
| ::: {.crayo                       | ::: {#crayon-5a5604               |
| n-num .crayon-striped-num line="c | 6b0f15f963307050-11 .crayon-line} |
| rayon-5a56046b0f15f963307050-14"} |                                   |
| 14                                | :::                               |
| :::                               |                                   |
|                                   | ::: {#c                           |
| ::: {.crayon-num line="c          | rayon-5a56046b0f15f963307050-12 . |
| rayon-5a56046b0f15f963307050-15"} | crayon-line .crayon-striped-line} |
| 15                                | 1[.]{.crayon-sy}[.]{.cra          |
| :::                               | yon-sy}[\$vmQuantity]{.crayon-v}[ |
|                                   | ]{.crayon-h}[\|]{.crayon-o}[      |
| ::: {.crayo                       | ]                                 |
| n-num .crayon-striped-num line="c | {.crayon-h}[foreach]{.crayon-st}[ |
| rayon-5a56046b0f15f963307050-16"} | ]{.crayon-h}[{]{.crayon-sy}       |
| 16                                | :::                               |
| :::                               |                                   |
|                                   | ::: {#crayon-5a5604               |
| ::: {.crayon-num line="c          | 6b0f15f963307050-13 .crayon-line} |
| rayon-5a56046b0f15f963307050-17"} | [ ]{.crayon-h}[\# if VM number is |
| 17                                | less than 10 add a 0 onto the     |
| :::                               | server name so we dont have       |
|                                   | MAILSERVER1 for                   |
| ::: {.crayo                       | example]{.crayon-c}               |
| n-num .crayon-striped-num line="c | :::                               |
| rayon-5a56046b0f15f963307050-18"} |                                   |
| 18                                | ::: {#c                           |
| :::                               | rayon-5a56046b0f15f963307050-14 . |
|                                   | crayon-line .crayon-striped-line} |
| ::: {.crayon-num line="c          | [ ]{.crayon-h}[if]{.crayon-st}[   |
| rayon-5a56046b0f15f963307050-19"} | ]{.crayon-h}[(]{.c                |
| 19                                | rayon-sy}[\$vmNumber]{.crayon-v}[ |
| :::                               | ]{.crayon-h}[-lt]{.crayon-cn}[    |
|                                   | ]{.crayon-h}10[)]{.crayon-sy}[    |
| ::: {.crayo                       | ]{.crayon-h}[{]{.crayon-sy}       |
| n-num .crayon-striped-num line="c | :::                               |
| rayon-5a56046b0f15f963307050-20"} |                                   |
| 20                                | ::: {#crayon-5a5604               |
| :::                               | 6b0f15f963307050-15 .crayon-line} |
|                                   | [                                 |
| ::: {.crayon-num line="c          | ]                                 |
| rayon-5a56046b0f15f963307050-21"} | {.crayon-h}[\$vmName]{.crayon-v}[ |
| 21                                | ]{.crayon-h}[=]{.crayon-o}[       |
| :::                               | ]{.cray                           |
|                                   | on-h}[\$vmNamePrefix]{.crayon-v}[ |
| ::: {.crayo                       | ]{.crayon-h}[+]{.crayon-o}[       |
| n-num .crayon-striped-num line="c | ]{.crayon-h}[\"0\"]{.crayon-s}[   |
| rayon-5a56046b0f15f963307050-22"} | ]{.crayon-h}[+]{.crayon-o}[       |
| 22                                | ]{                                |
| :::                               | .crayon-h}[\$vmNumber]{.crayon-v} |
|                                   | :::                               |
| ::: {.crayon-num line="c          |                                   |
| rayon-5a56046b0f15f963307050-23"} | ::: {#c                           |
| 23                                | rayon-5a56046b0f15f963307050-16 . |
| :::                               | crayon-line .crayon-striped-line} |
|                                   | [ ]{.crayon-h}[}]{.crayon-sy}[    |
| ::: {.crayo                       | ]{.crayon-h}[else]{.crayon-st}[   |
| n-num .crayon-striped-num line="c | ]{.crayon-h}[{]{.crayon-sy}       |
| rayon-5a56046b0f15f963307050-24"} | :::                               |
| 24                                |                                   |
| :::                               | ::: {#crayon-5a5604               |
|                                   | 6b0f15f963307050-17 .crayon-line} |
| ::: {.crayon-num line="c          | [                                 |
| rayon-5a56046b0f15f963307050-25"} | ]                                 |
| 25                                | {.crayon-h}[\$vmName]{.crayon-v}[ |
| :::                               | ]{.crayon-h}[=]{.crayon-o}[       |
|                                   | ]{.cray                           |
| ::: {.crayo                       | on-h}[\$vmNamePrefix]{.crayon-v}[ |
| n-num .crayon-striped-num line="c | ]{.crayon-h}[+]{.crayon-o}[       |
| rayon-5a56046b0f15f963307050-26"} | ]{                                |
| 26                                | .crayon-h}[\$vmNumber]{.crayon-v} |
| :::                               | :::                               |
|                                   |                                   |
| ::: {.crayon-num line="c          | ::: {#c                           |
| rayon-5a56046b0f15f963307050-27"} | rayon-5a56046b0f15f963307050-18 . |
| 27                                | crayon-line .crayon-striped-line} |
| :::                               | [ ]{.crayon-h}[}]{.crayon-sy}     |
|                                   | :::                               |
| ::: {.crayo                       |                                   |
| n-num .crayon-striped-num line="c | ::: {#crayon-5a5604               |
| rayon-5a56046b0f15f963307050-28"} | 6b0f15f963307050-19 .crayon-line} |
| 28                                |                                   |
| :::                               | :::                               |
|                                   |                                   |
| ::: {.crayon-num line="c          | ::: {#c                           |
| rayon-5a56046b0f15f963307050-29"} | rayon-5a56046b0f15f963307050-20 . |
| 29                                | crayon-line .crayon-striped-line} |
| :::                               | [ ]{.crayon-h}[\# compile IP      |
|                                   | address]{.crayon-c}               |
| ::: {.crayo                       | :::                               |
| n-num .crayon-striped-num line="c |                                   |
| rayon-5a56046b0f15f963307050-30"} | ::: {#crayon-5a5604               |
| 30                                | 6b0f15f963307050-21 .crayon-line} |
| :::                               | [        ]{.cra                   |
|                                   | yon-h}[\$vmIpAddress]{.crayon-v}[ |
| ::: {.crayon-num line="c          | ]{.crayon-h}[=]{.crayon-o}[       |
| rayon-5a56046b0f15f963307050-31"} | ]{.                               |
| 31                                | crayon-h}[\$ipPrefix]{.crayon-v}[ |
| :::                               | ]{.crayon-h}[+]{.crayon-o}[       |
|                                   | ]{.crayon-h}[(]{.c                |
| ::: {.crayo                       | rayon-sy}[\$ipSuffix]{.crayon-v}[ |
| n-num .crayon-striped-num line="c | ]{.crayon-h}[+]{.crayon-o}[       |
| rayon-5a56046b0f15f963307050-32"} | ]{.crayon-h}[(]                   |
| 32                                | {.crayon-sy}[\$vmCounter]{.crayon |
| :::                               | -v}[)]{.crayon-sy}[)]{.crayon-sy} |
|                                   | :::                               |
| ::: {.crayon-num line="c          |                                   |
| rayon-5a56046b0f15f963307050-33"} | ::: {#c                           |
| 33                                | rayon-5a56046b0f15f963307050-22 . |
| :::                               | crayon-line .crayon-striped-line} |
|                                   |                                   |
| ::: {.crayo                       | :::                               |
| n-num .crayon-striped-num line="c |                                   |
| rayon-5a56046b0f15f963307050-34"} | ::: {#crayon-5a5604               |
| 34                                | 6b0f15f963307050-23 .crayon-line} |
| :::                               | [ ]{.crayon-h}[\# setup object    |
|                                   | properties]{.crayon-c}            |
| ::: {.crayon-num line="c          | :::                               |
| rayon-5a56046b0f15f963307050-35"} |                                   |
| 35                                | ::: {#c                           |
| :::                               | rayon-5a56046b0f15f963307050-24 . |
|                                   | crayon-line .crayon-striped-line} |
| ::: {.crayo                       | [                                 |
| n-num .crayon-striped-num line="c | ]{.cr                             |
| rayon-5a56046b0f15f963307050-36"} | ayon-h}[\$properties]{.crayon-v}[ |
| 36                                | ]{.crayon-h}[=]{.crayon-o}[       |
| :::                               | ]{.crayon-h}[\[]{.crayon-sy}      |
|                                   | [ordered]{.crayon-e}[\]]{.crayon- |
| ::: {.crayon-num line="c          | sy}[@]{.crayon-sy}[{]{.crayon-sy} |
| rayon-5a56046b0f15f963307050-37"} | :::                               |
| 37                                |                                   |
| :::                               | ::: {#crayon-5a5604               |
|                                   | 6b0f15f963307050-25 .crayon-line} |
| ::: {.crayo                       | [                                 |
| n-num .crayon-striped-num line="c | ]{.c                              |
| rayon-5a56046b0f15f963307050-38"} | rayon-h}[MachineName]{.crayon-i}[ |
| 38                                | ]{.crayon-h}[=]{.crayon-o}[       |
| :::                               | ]{.crayon-h}[\$vmName]{.crayon-v} |
|                                   | :::                               |
| ::: {.crayon-num line="c          |                                   |
| rayon-5a56046b0f15f963307050-39"} | ::: {#c                           |
| 39                                | rayon-5a56046b0f15f963307050-26 . |
| :::                               | crayon-line .crayon-striped-line} |
|                                   | [                                 |
| ::: {.crayo                       | ]{                                |
| n-num .crayon-striped-num line="c | .crayon-h}[IpAddress]{.crayon-i}[ |
| rayon-5a56046b0f15f963307050-40"} | ]{.crayon-h}[=]{.crayon-o}[       |
| 40                                | ]{.cr                             |
| :::                               | ayon-h}[\$vmIpAddress]{.crayon-v} |
| ::::::::::                        | :::                               |
| ::::::::::::::::::::::::::::::::: |                                   |
|                                   | ::: {#crayon-5a5604               |
|                                   | 6b0f15f963307050-27 .crayon-line} |
|                                   | [                                 |
|                                   | ]{.cray                           |
|                                   | on-h}[CloneInitiated]{.crayon-i}[ |
|                                   | ]{.crayon-h}[=]{.crayon-o}[       |
|                                   | ]{.crayon-h}0                     |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#c                           |
|                                   | rayon-5a56046b0f15f963307050-28 . |
|                                   | crayon-line .crayon-striped-line} |
|                                   | [                                 |
|                                   | ]{.crayon-h}[Cloned]{.crayon-i}[  |
|                                   | ]{.crayon-h}[=]{.crayon-o}[       |
|                                   | ]{.crayon-h}0                     |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#crayon-5a5604               |
|                                   | 6b0f15f963307050-29 .crayon-line} |
|                                   | [                                 |
|                                   | ]{                                |
|                                   | .crayon-h}[PoweredOn]{.crayon-i}[ |
|                                   | ]{.crayon-h}[=]{.crayon-o}[       |
|                                   | ]{.crayon-h}0                     |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#c                           |
|                                   | rayon-5a56046b0f15f963307050-30 . |
|                                   | crayon-line .crayon-striped-line} |
|                                   | [ ]{.crayon-h}[ReIP]{.crayon-i}[  |
|                                   | ]{.crayon-h}[=]{.crayon-o}[       |
|                                   | ]{.crayon-h}0                     |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#crayon-5a5604               |
|                                   | 6b0f15f963307050-31 .crayon-line} |
|                                   | [ ]{.crayon-h}[}]{.crayon-sy}     |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#c                           |
|                                   | rayon-5a56046b0f15f963307050-32 . |
|                                   | crayon-line .crayon-striped-line} |
|                                   |                                   |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#crayon-5a5604               |
|                                   | 6b0f15f963307050-33 .crayon-line} |
|                                   | [ ]{.crayon-h}[\# create object   |
|                                   | and add to collection]{.crayon-c} |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#c                           |
|                                   | rayon-5a56046b0f15f963307050-34 . |
|                                   | crayon-line .crayon-striped-line} |
|                                   | [ ]{.crayon-h}[\$obj]{.crayon-v}[ |
|                                   | ]{.crayon-h}[=]{.crayon-o}[       |
|                                   | ]{.                               |
|                                   | crayon-h}[New-Object]{.crayon-r}[ |
|                                   | ]{.                               |
|                                   | crayon-h}[-TypeName]{.crayon-cn}[ |
|                                   | ]                                 |
|                                   | {.crayon-h}[PSObject]{.crayon-t}[ |
|                                   | ]{.                               |
|                                   | crayon-h}[-Property]{.crayon-cn}[ |
|                                   | ]{.c                              |
|                                   | rayon-h}[\$properties]{.crayon-v} |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#crayon-5a5604               |
|                                   | 6b0f15f963307050-35 .crayon-line} |
|                                   | [                                 |
|                                   | ]{.                               |
|                                   | crayon-h}[\$vmStatus]{.crayon-v}[ |
|                                   | ]{.crayon-h}[+=]{.crayon-o}[      |
|                                   | ]{.crayon-h}[\$obj]{.crayon-v}    |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#c                           |
|                                   | rayon-5a56046b0f15f963307050-36 . |
|                                   | crayon-line .crayon-striped-line} |
|                                   |                                   |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#crayon-5a5604               |
|                                   | 6b0f15f963307050-37 .crayon-line} |
|                                   | [ ]{.crayon-h}[\# increase        |
|                                   | counters]{.crayon-c}              |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#c                           |
|                                   | rayon-5a56046b0f15f963307050-38 . |
|                                   | crayon-line .crayon-striped-line} |
|                                   | [                                 |
|                                   | ]{.crayon-h}[\$vm                 |
|                                   | Number]{.crayon-v}[++]{.crayon-o} |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#crayon-5a5604               |
|                                   | 6b0f15f963307050-39 .crayon-line} |
|                                   | [                                 |
|                                   | ]{.crayon-h}[\$vmC                |
|                                   | ounter]{.crayon-v}[++]{.crayon-o} |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#c                           |
|                                   | rayon-5a56046b0f15f963307050-40 . |
|                                   | crayon-line .crayon-striped-line} |
|                                   | [}]{.crayon-sy}                   |
|                                   | :::                               |
|                                   | ::::::::::                        |
|                                   | ::::::::::::::::::::::::::::::::: |
+-----------------------------------+-----------------------------------+
:::
::::::::::::::::::::

This is a good start, we now have a list of VMs, their IP addresses and
the status of deployment. You could modify the above to perhaps import a
list from CSV, rather than ask for the VM details interactively. A good
reason to do this would be if you wanted to deploy multiple VMs without
following my naming convention or adjacent IP addresses.

Prepare the command to deploy a VM using the deploy VM script we created
earlier. I also clear the current job list in case any are still in
memory from a previous time I ran the script without reloading my
PowerShell session:

:::::::::::::::::::: {#crayon-5a56046b0f167097991859 .crayon-syntax .crayon-theme-classic .crayon-font-monaco .crayon-os-pc .print-yes .notranslate settings=" minimize scroll-mouseover wrap" style=" margin-top: 12px; margin-bottom: 12px; font-size: 12px !important; line-height: 15px !important;"}
:::::::::::::::: {.crayon-toolbar settings=" mouseover overlay hide delay" style="font-size: 12px !important;height: 18px !important; line-height: 18px !important;"}
[]{.crayon-title}

::::::::::::::: {.crayon-tools style="font-size: 12px !important;height: 18px !important; line-height: 18px !important;"}
:::: {.crayon-button .crayon-nums-button title="Toggle Line Numbers"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-plain-button title="Toggle Plain Code"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-wrap-button title="Toggle Line Wrap"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-expand-button title="Expand Code"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-copy-button title="Copy"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-popup-button title="Open Code In New Window"}
::: crayon-button-icon
:::
::::

[PowerShell]{.crayon-language}
:::::::::::::::
::::::::::::::::

::: {.crayon-info style="min-height: 16.8px !important; line-height: 16.8px !important;"}
:::

::: crayon-plain-wrap
:::

::: {.crayon-main style=""}
+-----------------------------------+-----------------------------------+
| :::::                             | ::::::::::::::: {.crayon-pr       |
| :::::::::: {.crayon-nums-content  | e style="font-size: 12px !importa |
| style="font-size: 12px !important | nt; line-height: 15px !important; |
| ; line-height: 15px !important;"} |  -moz-tab-size:4; -o-tab-size:4;  |
| ::: {.crayon-num line="           | -webkit-tab-size:4; tab-size:4;"} |
| crayon-5a56046b0f167097991859-1"} | ::: {#crayon-5a560                |
| 1                                 | 46b0f167097991859-1 .crayon-line} |
| :::                               | [\###################]{.crayon-c} |
|                                   | :::                               |
| ::: {.cray                        |                                   |
| on-num .crayon-striped-num line=" | ::: {#                            |
| crayon-5a56046b0f167097991859-2"} | crayon-5a56046b0f167097991859-2 . |
| 2                                 | crayon-line .crayon-striped-line} |
| :::                               | [\# Prepare to start deploying    |
|                                   | VMs]{.crayon-c}                   |
| ::: {.crayon-num line="           | :::                               |
| crayon-5a56046b0f167097991859-3"} |                                   |
| 3                                 | ::: {#crayon-5a560                |
| :::                               | 46b0f167097991859-3 .crayon-line} |
|                                   | [\###################]{.crayon-c} |
| ::: {.cray                        | :::                               |
| on-num .crayon-striped-num line=" |                                   |
| crayon-5a56046b0f167097991859-4"} | ::: {#                            |
| 4                                 | crayon-5a56046b0f167097991859-4 . |
| :::                               | crayon-line .crayon-striped-line} |
|                                   |                                   |
| ::: {.crayon-num line="           | :::                               |
| crayon-5a56046b0f167097991859-5"} |                                   |
| 5                                 | ::: {#crayon-5a560                |
| :::                               | 46b0f167097991859-5 .crayon-line} |
|                                   | [\# prepare VM deploy job         |
| ::: {.cray                        | command]{.crayon-c}               |
| on-num .crayon-striped-num line=" | :::                               |
| crayon-5a56046b0f167097991859-6"} |                                   |
| 6                                 | ::: {#                            |
| :::                               | crayon-5a56046b0f167097991859-6 . |
|                                   | crayon-line .crayon-striped-line} |
| ::: {.crayon-num line="           | [\$job]{.crayon-v}[=]{.crayon-o}[ |
| crayon-5a56046b0f167097991859-7"} | ]{.crayon-h}[{]{.crayon-sy}       |
| 7                                 | :::                               |
| :::                               |                                   |
|                                   | ::: {#crayon-5a560                |
| ::: {.cray                        | 46b0f167097991859-7 .crayon-line} |
| on-num .crayon-striped-num line=" | [                                 |
| crayon-5a56046b0f167097991859-8"} | ]{.cr                             |
| 8                                 | ayon-h}[Set-Location]{.crayon-r}[ |
| :::                               | ]{.crayon-h}[\$args]{.crayon-v}   |
|                                   | [\[]{.crayon-sy}0[\]]{.crayon-sy} |
| ::: {.crayon-num line="           | :::                               |
| crayon-5a56046b0f167097991859-9"} |                                   |
| 9                                 | ::: {#                            |
| :::                               | crayon-5a56046b0f167097991859-8 . |
|                                   | crayon-line .crayon-striped-line} |
| ::: {.crayo                       | [                                 |
| n-num .crayon-striped-num line="c | ]{.                               |
| rayon-5a56046b0f167097991859-10"} | crayon-h}[powershell]{.crayon-i}[ |
| 10                                | ]{                                |
| :::                               | .crayon-h}[-command]{.crayon-cn}[ |
|                                   | ]{.crayon-h}[\".\\DeployVM.ps1    |
| ::: {.crayon-num line="c          | -session \$(\$args\[1\])          |
| rayon-5a56046b0f167097991859-11"} | -vcserver \$(\$args\[2\]) -name   |
| 11                                | \$(\$args\[3\]) -datastore        |
| :::                               | \$(\$args\[4\]) -ipaddr           |
|                                   | \$(\$args\[5\]) -template         |
| ::: {.crayo                       | \`\'\$(\$args\[6\])\`\' -folder   |
| n-num .crayon-striped-num line="c | \`\'                              |
| rayon-5a56046b0f167097991859-12"} | \$(\$args\[7\])\`\'\"]{.crayon-s} |
| 12                                | :::                               |
| :::                               |                                   |
| :::::::::::::::                   | ::: {#crayon-5a560                |
|                                   | 46b0f167097991859-9 .crayon-line} |
|                                   | [}]{.crayon-sy}                   |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#c                           |
|                                   | rayon-5a56046b0f167097991859-10 . |
|                                   | crayon-line .crayon-striped-line} |
|                                   |                                   |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#crayon-5a5604               |
|                                   | 6b0f167097991859-11 .crayon-line} |
|                                   | [\# clear job list]{.crayon-c}    |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#c                           |
|                                   | rayon-5a56046b0f167097991859-12 . |
|                                   | crayon-line .crayon-striped-line} |
|                                   | [Get-Job]{.crayon-r}[             |
|                                   | ]{.crayon-h}[\|]{.crayon-o}[      |
|                                   | ]{                                |
|                                   | .crayon-h}[Remove-Job]{.crayon-r} |
|                                   | :::                               |
|                                   | :::::::::::::::                   |
+-----------------------------------+-----------------------------------+
:::
::::::::::::::::::::

Time to start deploying multiple VMs! The following loop will continue
looping until all tasks have been completed, that is all the properties
for each VM within \$vmStatus have a value of 1. Notice in the next few
examples before each task I select the VMs which have not yet had that
task completed and where previous tasks have been completed using a
where query on \$vmStatus. If the task then completes successfully, I
set that property to 1.

:::::::::::::::::::: {#crayon-5a56046b0f16e449678172 .crayon-syntax .crayon-theme-classic .crayon-font-monaco .crayon-os-pc .print-yes .notranslate settings=" minimize scroll-mouseover" style=" margin-top: 12px; margin-bottom: 12px; font-size: 12px !important; line-height: 15px !important;"}
:::::::::::::::: {.crayon-toolbar settings=" mouseover overlay hide delay" style="font-size: 12px !important;height: 18px !important; line-height: 18px !important;"}
[]{.crayon-title}

::::::::::::::: {.crayon-tools style="font-size: 12px !important;height: 18px !important; line-height: 18px !important;"}
:::: {.crayon-button .crayon-nums-button title="Toggle Line Numbers"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-plain-button title="Toggle Plain Code"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-wrap-button title="Toggle Line Wrap"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-expand-button title="Expand Code"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-copy-button title="Copy"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-popup-button title="Open Code In New Window"}
::: crayon-button-icon
:::
::::

[PowerShell]{.crayon-language}
:::::::::::::::
::::::::::::::::

::: {.crayon-info style="min-height: 16.8px !important; line-height: 16.8px !important;"}
:::

::: crayon-plain-wrap
:::

::: {.crayon-main style=""}
+-----------------------------------+-----------------------------------+
| ::::::::::::                      | :                                 |
| :::::::::: {.crayon-nums-content  | ::::::::::::::::::::: {.crayon-pr |
| style="font-size: 12px !important | e style="font-size: 12px !importa |
| ; line-height: 15px !important;"} | nt; line-height: 15px !important; |
| ::: {.crayon-num line="           |  -moz-tab-size:4; -o-tab-size:4;  |
| crayon-5a56046b0f16e449678172-1"} | -webkit-tab-size:4; tab-size:4;"} |
| 1                                 | ::: {#crayon-5a560                |
| :::                               | 46b0f16e449678172-1 .crayon-line} |
|                                   | [while]{.crayon-st}[              |
| ::: {.cray                        | ]{.crayon-h}[(]{.crayo            |
| on-num .crayon-striped-num line=" | n-sy}[\$vmsCompleted]{.crayon-v}[ |
| crayon-5a56046b0f16e449678172-2"} | .]{.crayon-sy}[Count]{.crayon-i}[ |
| 2                                 | ]{.crayon-h}[-lt]{.crayon-cn}[    |
| :::                               | ]{.crayon-h}[\$vmQua              |
|                                   | ntity]{.crayon-v}[)]{.crayon-sy}[ |
| ::: {.crayon-num line="           | ]{.crayon-h}[{]{.crayon-sy}       |
| crayon-5a56046b0f16e449678172-3"} | :::                               |
| 3                                 |                                   |
| :::                               | ::: {#                            |
|                                   | crayon-5a56046b0f16e449678172-2 . |
| ::: {.cray                        | crayon-line .crayon-striped-line} |
| on-num .crayon-striped-num line=" |                                   |
| crayon-5a56046b0f16e449678172-4"} | :::                               |
| 4                                 |                                   |
| :::                               | ::: {#crayon-5a560                |
|                                   | 46b0f16e449678172-3 .crayon-line} |
| ::: {.crayon-num line="           | [                                 |
| crayon-5a56046b0f16e449678172-5"} | ]{.crayon-h}                      |
| 5                                 | [\###################]{.crayon-c} |
| :::                               | :::                               |
|                                   |                                   |
| ::: {.cray                        | ::: {#                            |
| on-num .crayon-striped-num line=" | crayon-5a56046b0f16e449678172-4 . |
| crayon-5a56046b0f16e449678172-6"} | crayon-line .crayon-striped-line} |
| 6                                 | [ ]{.crayon-h}[\# Initiate VM     |
| :::                               | cloning, using                    |
|                                   | \$maxConcurrentJobs as the        |
| ::: {.crayon-num line="           | limit]{.crayon-c}                 |
| crayon-5a56046b0f16e449678172-7"} | :::                               |
| 7                                 |                                   |
| :::                               | ::: {#crayon-5a560                |
|                                   | 46b0f16e449678172-5 .crayon-line} |
| ::: {.cray                        | [                                 |
| on-num .crayon-striped-num line=" | ]{.crayon-h}                      |
| crayon-5a56046b0f16e449678172-8"} | [\###################]{.crayon-c} |
| 8                                 | :::                               |
| :::                               |                                   |
|                                   | ::: {#                            |
| ::: {.crayon-num line="           | crayon-5a56046b0f16e449678172-6 . |
| crayon-5a56046b0f16e449678172-9"} | crayon-line .crayon-striped-line} |
| 9                                 |                                   |
| :::                               | :::                               |
|                                   |                                   |
| ::: {.crayo                       | ::: {#crayon-5a560                |
| n-num .crayon-striped-num line="c | 46b0f16e449678172-7 .crayon-line} |
| rayon-5a56046b0f16e449678172-10"} | [ ]{.crayon-h}[\# do not start    |
| 10                                | job if \$maxConcurrentJobs are    |
| :::                               | already running, do not start if  |
|                                   | all VMs are now cloned (clone     |
| ::: {.crayon-num line="c          | initiated) - skip this            |
| rayon-5a56046b0f16e449678172-11"} | task]{.crayon-c}                  |
| 11                                | :::                               |
| :::                               |                                   |
|                                   | ::: {#                            |
| ::: {.crayo                       | crayon-5a56046b0f16e449678172-8 . |
| n-num .crayon-striped-num line="c | crayon-line .crayon-striped-line} |
| rayon-5a56046b0f16e449678172-12"} | [ ]{.crayon-h}[if]{.crayon-st}[   |
| 12                                | ]{.crayon                         |
| :::                               | -h}[(]{.crayon-sy}[(]{.crayon-sy} |
|                                   | [\$vmsCloneInitiated]{.crayon-v}[ |
| ::: {.crayon-num line="c          | .]{.crayon-sy}[Count]{.crayon-i}[ |
| rayon-5a56046b0f16e449678172-13"} | ]{.crayon-h}[-lt]{.crayon-cn}[    |
| 13                                | ]{.crayon-h}[\$vmQua              |
| :::                               | ntity]{.crayon-v}[)]{.crayon-sy}[ |
|                                   | ]{.crayon-h}[-and]{.crayon-cn}[   |
| ::: {.crayo                       | ]{.crayon-h}[(]{.crayon-s         |
| n-num .crayon-striped-num line="c | y}[\$runningJobCount]{.crayon-v}[ |
| rayon-5a56046b0f16e449678172-14"} | ]{.crayon-h}[-lt]{.crayon-cn}[    |
| 14                                | ]{.crayon                         |
| :::                               | -h}[\$maxConcurrentJobs]{.crayon- |
|                                   | v}[)]{.crayon-sy}[)]{.crayon-sy}[ |
| ::: {.crayon-num line="c          | ]{.crayon-h}[{]{.crayon-sy}       |
| rayon-5a56046b0f16e449678172-15"} | :::                               |
| 15                                |                                   |
| :::                               | ::: {#crayon-5a560                |
|                                   | 46b0f16e449678172-9 .crayon-line} |
| ::: {.crayo                       |                                   |
| n-num .crayon-striped-num line="c | :::                               |
| rayon-5a56046b0f16e449678172-16"} |                                   |
| 16                                | ::: {#c                           |
| :::                               | rayon-5a56046b0f16e449678172-10 . |
|                                   | crayon-line .crayon-striped-line} |
| ::: {.crayon-num line="c          | [        ]{.crayon-h}[\# select   |
| rayon-5a56046b0f16e449678172-17"} | VMs which have not yet been       |
| 17                                | cloned - and only the first       |
| :::                               | \$maxConcurrentJobs               |
|                                   | VMs]{.crayon-c}                   |
| ::: {.crayo                       | :::                               |
| n-num .crayon-striped-num line="c |                                   |
| rayon-5a56046b0f16e449678172-18"} | ::: {#crayon-5a5604               |
| 18                                | 6b0f16e449678172-11 .crayon-line} |
| :::                               | [        ]{.                      |
|                                   | crayon-h}[\$vmStatus]{.crayon-v}[ |
| ::: {.crayon-num line="c          | ]{.crayon-h}[\|]{.crayon-o}[      |
| rayon-5a56046b0f16e449678172-19"} | ]{.crayon-h}[where]{.crayon-st}[  |
| 19                                | ]{.crayon-h}[{]{.crayon-sy}[      |
| :::                               | (]{.crayon-sy}[\$\_]{.crayon-v}[. |
| ::::::::::::::::::::::            | ]{.crayon-sy}[Cloned]{.crayon-i}[ |
|                                   | ]{.crayon-h}[-eq]{.crayon-cn}[    |
|                                   | ]{.crayon-h                       |
|                                   | }0[)]{.crayon-sy}[}]{.crayon-sy}[ |
|                                   | ]{.crayon-h}[\|]{.crayon-o}[      |
|                                   | ]{.cra                            |
|                                   | yon-h}[Select-Object]{.crayon-r}[ |
|                                   | ]{.crayon-h}[-first]{.crayon-cn}[ |
|                                   | ]{.crayon-h}                      |
|                                   | [\$maxConcurrentJobs]{.crayon-v}[ |
|                                   | ]{.crayon-h}[\|]{.crayon-o}[      |
|                                   | ]                                 |
|                                   | {.crayon-h}[foreach]{.crayon-st}[ |
|                                   | ]{.crayon-h}[{]{.crayon-sy}       |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#c                           |
|                                   | rayon-5a56046b0f16e449678172-12 . |
|                                   | crayon-line .crayon-striped-line} |
|                                   |                                   |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#crayon-5a5604               |
|                                   | 6b0f16e449678172-13 .crayon-line} |
|                                   | [     ]{.crayon-h}[\# create      |
|                                   | job]{.crayon-c}                   |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#c                           |
|                                   | rayon-5a56046b0f16e449678172-14 . |
|                                   | crayon-line .crayon-striped-line} |
|                                   | [                                 |
|                                   |     ]{                            |
|                                   | .crayon-h}[Start-Job]{.crayon-r}[ |
|                                   | ]{.crayon-h}[-Name]{.crayon-cn}[  |
|                                   | ]{.                               |
|                                   | crayon-h}[\$\_]{.crayon-v}[.]{.cr |
|                                   | ayon-sy}[MachineName]{.crayon-i}[ |
|                                   | ]{.cra                            |
|                                   | yon-h}[-ScriptBlock]{.crayon-cn}[ |
|                                   | ]{.crayon-h}[\$job]{.crayon-v}[   |
|                                   | ]{.cray                           |
|                                   | on-h}[-ArgumentList]{.crayon-cn}[ |
|                                   | ]{.crayon-h}[\$curren             |
|                                   | tPath]{.crayon-v}[,]{.crayon-sy}[ |
|                                   | ]{.crayon-h}[\$se                 |
|                                   | ssion]{.crayon-v}[,]{.crayon-sy}[ |
|                                   | ]{.crayon-h}[\$vcs                |
|                                   | erver]{.crayon-v}[,]{.crayon-sy}[ |
|                                   | ]{.crayon-h}[\$\_]                |
|                                   | {.crayon-v}[.]{.crayon-sy}[Machin |
|                                   | eName]{.crayon-i}[,]{.crayon-sy}[ |
|                                   | ]{.crayon-h}[\$\                  |
|                                   | _]{.crayon-v}[.]{.crayon-sy}[Data |
|                                   | store]{.crayon-i}[,]{.crayon-sy}[ |
|                                   | ]{.crayon-h}[\$\                  |
|                                   | _]{.crayon-v}[.]{.crayon-sy}[IpAd |
|                                   | dress]{.crayon-i}[,]{.crayon-sy}[ |
|                                   | ]{.crayon-h}[\$tem                |
|                                   | plate]{.crayon-v}[,]{.crayon-sy}[ |
|                                   | ]                                 |
|                                   | {.crayon-h}[\$folder]{.crayon-v}[ |
|                                   | ]{.crayon-h}                      |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#crayon-5a5604               |
|                                   | 6b0f16e449678172-15 .crayon-line} |
|                                   |                                   |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#c                           |
|                                   | rayon-5a56046b0f16e449678172-16 . |
|                                   | crayon-line .crayon-striped-line} |
|                                   | [     ]{.crayon-h}[\# update      |
|                                   | vmStatus object                   |
|                                   | collection]{.crayon-c}            |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#crayon-5a5604               |
|                                   | 6b0f16e449678172-17 .crayon-line} |
|                                   | [                                 |
|                                   |     ]{.cra                        |
|                                   | yon-h}[\$\_]{.crayon-v}[.]{.crayo |
|                                   | n-sy}[CloneInitiated]{.crayon-i}[ |
|                                   | ]{.crayon-h}[=]{.crayon-o}[       |
|                                   | ]{.crayon-h}1                     |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#c                           |
|                                   | rayon-5a56046b0f16e449678172-18 . |
|                                   | crayon-line .crayon-striped-line} |
|                                   | [                                 |
|                                   |       ]{.crayon-h}[}]{.crayon-sy} |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#crayon-5a5604               |
|                                   | 6b0f16e449678172-19 .crayon-line} |
|                                   | [ ]{.crayon-h}[}]{.crayon-sy}     |
|                                   | :::                               |
|                                   | ::::::::::::::::::::::            |
+-----------------------------------+-----------------------------------+
:::
::::::::::::::::::::

There are a few variables in the above example I have not yet mentioned,
such as \$vmsComplete.Count, \$vmsCloneInitiated.Count and
\$runningJobCount. Lets update these

:::::::::::::::::::: {#crayon-5a56046b0f175674389632 .crayon-syntax .crayon-theme-classic .crayon-font-monaco .crayon-os-pc .print-yes .notranslate settings=" minimize scroll-mouseover" style=" margin-top: 12px; margin-bottom: 12px; font-size: 12px !important; line-height: 15px !important;"}
:::::::::::::::: {.crayon-toolbar settings=" mouseover overlay hide delay" style="font-size: 12px !important;height: 18px !important; line-height: 18px !important;"}
[]{.crayon-title}

::::::::::::::: {.crayon-tools style="font-size: 12px !important;height: 18px !important; line-height: 18px !important;"}
:::: {.crayon-button .crayon-nums-button title="Toggle Line Numbers"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-plain-button title="Toggle Plain Code"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-wrap-button title="Toggle Line Wrap"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-expand-button title="Expand Code"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-copy-button title="Copy"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-popup-button title="Open Code In New Window"}
::: crayon-button-icon
:::
::::

[PowerShell]{.crayon-language}
:::::::::::::::
::::::::::::::::

::: {.crayon-info style="min-height: 16.8px !important; line-height: 16.8px !important;"}
:::

::: crayon-plain-wrap
:::

::: {.crayon-main style=""}
+-----------------------------------+-----------------------------------+
| ::::::::::::                      | :                                 |
| :::::::::: {.crayon-nums-content  | ::::::::::::::::::::: {.crayon-pr |
| style="font-size: 12px !important | e style="font-size: 12px !importa |
| ; line-height: 15px !important;"} | nt; line-height: 15px !important; |
| ::: {.crayon-num line="           |  -moz-tab-size:4; -o-tab-size:4;  |
| crayon-5a56046b0f175674389632-1"} | -webkit-tab-size:4; tab-size:4;"} |
| 1                                 | ::: {#crayon-5a560                |
| :::                               | 46b0f175674389632-1 .crayon-line} |
|                                   | [                                 |
| ::: {.cray                        | ]{.crayon-h}                      |
| on-num .crayon-striped-num line=" | [\###################]{.crayon-c} |
| crayon-5a56046b0f175674389632-2"} | :::                               |
| 2                                 |                                   |
| :::                               | ::: {#                            |
|                                   | crayon-5a56046b0f175674389632-2 . |
| ::: {.crayon-num line="           | crayon-line .crayon-striped-line} |
| crayon-5a56046b0f175674389632-3"} | [ ]{.crayon-h}[\# Check running   |
| 3                                 | job count and look for VMs which  |
| :::                               | have finished cloning]{.crayon-c} |
|                                   | :::                               |
| ::: {.cray                        |                                   |
| on-num .crayon-striped-num line=" | ::: {#crayon-5a560                |
| crayon-5a56046b0f175674389632-4"} | 46b0f175674389632-3 .crayon-line} |
| 4                                 | [                                 |
| :::                               | ]{.crayon-h}                      |
|                                   | [\###################]{.crayon-c} |
| ::: {.crayon-num line="           | :::                               |
| crayon-5a56046b0f175674389632-5"} |                                   |
| 5                                 | ::: {#                            |
| :::                               | crayon-5a56046b0f175674389632-4 . |
|                                   | crayon-line .crayon-striped-line} |
| ::: {.cray                        |                                   |
| on-num .crayon-striped-num line=" | :::                               |
| crayon-5a56046b0f175674389632-6"} |                                   |
| 6                                 | ::: {#crayon-5a560                |
| :::                               | 46b0f175674389632-5 .crayon-line} |
|                                   | [ ]{.crayon-h}[\# count current   |
| ::: {.crayon-num line="           | running jobs]{.crayon-c}          |
| crayon-5a56046b0f175674389632-7"} | :::                               |
| 7                                 |                                   |
| :::                               | ::: {#                            |
|                                   | crayon-5a56046b0f175674389632-6 . |
| ::: {.cray                        | crayon-line .crayon-striped-line} |
| on-num .crayon-striped-num line=" | [                                 |
| crayon-5a56046b0f175674389632-8"} | ]{.crayon-h}[\$jobs]{.crayon-v}[  |
| 8                                 | ]{.crayon-h}[=]{.crayon-o}[       |
| :::                               | ]{.crayon-h}[Get-Job]{.crayon-r}[ |
|                                   | ]{.crayon-h}                      |
| ::: {.crayon-num line="           | :::                               |
| crayon-5a56046b0f175674389632-9"} |                                   |
| 9                                 | ::: {#crayon-5a560                |
| :::                               | 46b0f175674389632-7 .crayon-line} |
|                                   | [                                 |
| ::: {.crayo                       | ]{.cra                            |
| n-num .crayon-striped-num line="c | yon-h}[\$runningJobs]{.crayon-v}[ |
| rayon-5a56046b0f175674389632-10"} | ]{.crayon-h}[=]{.crayon-o}[       |
| 10                                | ]{.crayon-h}[\$jobs]{.crayon-v}[  |
| :::                               | ]{.crayon-h}[\|]{.crayon-o}[      |
|                                   | ]{.crayon-h}[?]{.crayon-sy}[      |
| ::: {.crayon-num line="c          | ]{.crayon-h}[{]{.crayon-sy}[      |
| rayon-5a56046b0f175674389632-11"} | ]{.crayon-h}[\$\_]{.crayon-v}[    |
| 11                                | .]{.crayon-sy}[state]{.crayon-i}[ |
| :::                               | ]{.crayon-h}[-eq]{.crayon-cn}[    |
|                                   | ]{.c                              |
| ::: {.crayo                       | rayon-h}[\"Running\"]{.crayon-s}[ |
| n-num .crayon-striped-num line="c | ]{.crayon-h}[}]{.crayon-sy}       |
| rayon-5a56046b0f175674389632-12"} | :::                               |
| 12                                |                                   |
| :::                               | ::: {#                            |
|                                   | crayon-5a56046b0f175674389632-8 . |
| ::: {.crayon-num line="c          | crayon-line .crayon-striped-line} |
| rayon-5a56046b0f175674389632-13"} | [                                 |
| 13                                | ]{.crayon-                        |
| :::                               | h}[\$runningJobCount]{.crayon-v}[ |
|                                   | ]{.crayon-h}[=]{.crayon-o}[       |
| ::: {.crayo                       | ]{.cr                             |
| n-num .crayon-striped-num line="c | ayon-h}[\$runningJobs]{.crayon-v} |
| rayon-5a56046b0f175674389632-14"} | [.]{.crayon-sy}[count]{.crayon-i} |
| 14                                | :::                               |
| :::                               |                                   |
|                                   | ::: {#crayon-5a560                |
| ::: {.crayon-num line="c          | 46b0f175674389632-9 .crayon-line} |
| rayon-5a56046b0f175674389632-15"} |                                   |
| 15                                | :::                               |
| :::                               |                                   |
|                                   | ::: {#c                           |
| ::: {.crayo                       | rayon-5a56046b0f175674389632-10 . |
| n-num .crayon-striped-num line="c | crayon-line .crayon-striped-line} |
| rayon-5a56046b0f175674389632-16"} | [ ]{.crayon-h}[\# check for vms   |
| 16                                | which have finished               |
| :::                               | cloning]{.crayon-c}               |
|                                   | :::                               |
| ::: {.crayon-num line="c          |                                   |
| rayon-5a56046b0f175674389632-17"} | ::: {#crayon-5a5604               |
| 17                                | 6b0f175674389632-11 .crayon-line} |
| :::                               | [                                 |
|                                   | ]{.crayo                          |
| ::: {.crayo                       | n-h}[\$completedJobs]{.crayon-v}[ |
| n-num .crayon-striped-num line="c | ]{.crayon-h}[=]{.crayon-o}[       |
| rayon-5a56046b0f175674389632-18"} | ]{.crayon-h}[\$jobs]{.crayon-v}[  |
| 18                                | ]{.crayon-h}[\|]{.crayon-o}[      |
| :::                               | ]{.crayon-h}[?]{.crayon-sy}[      |
|                                   | ]{.crayon-h}[{]{.crayon-sy}[      |
| ::: {.crayon-num line="c          | ]{.crayon-h}[\$\_]{.crayon-v}[    |
| rayon-5a56046b0f175674389632-19"} | .]{.crayon-sy}[state]{.crayon-i}[ |
| 19                                | ]{.crayon-h}[-eq]{.crayon-cn}[    |
| :::                               | ]{.cra                            |
| ::::::::::::::::::::::            | yon-h}[\"Completed\"]{.crayon-s}[ |
|                                   | ]{.crayon-h}[}]{.crayon-sy}[      |
|                                   | ]{.crayon-h}                      |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#c                           |
|                                   | rayon-5a56046b0f175674389632-12 . |
|                                   | crayon-line .crayon-striped-line} |
|                                   |                                   |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#crayon-5a5604               |
|                                   | 6b0f175674389632-13 .crayon-line} |
|                                   | [ ]{.crayon-h}[\# update          |
|                                   | \$vmStatus]{.crayon-c}            |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#c                           |
|                                   | rayon-5a56046b0f175674389632-14 . |
|                                   | crayon-line .crayon-striped-line} |
|                                   | [                                 |
|                                   | ]                                 |
|                                   | {.crayon-h}[foreach]{.crayon-st}[ |
|                                   | ]{.crayon-h}[(]{.crayo            |
|                                   | n-sy}[\$completedJob]{.crayon-v}[ |
|                                   | ]{.crayon-h}[in]{.crayon-st}[     |
|                                   | ]{.crayon-h}[\$complete           |
|                                   | dJobs]{.crayon-v}[)]{.crayon-sy}[ |
|                                   | ]{.crayon-h}[{]{.crayon-sy}       |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#crayon-5a5604               |
|                                   | 6b0f175674389632-15 .crayon-line} |
|                                   |                                   |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#c                           |
|                                   | rayon-5a56046b0f175674389632-16 . |
|                                   | crayon-line .crayon-striped-line} |
|                                   | [ ]{.crayon-h}[\# get index of    |
|                                   | object and set cloned to          |
|                                   | 1]{.crayon-c}                     |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#crayon-5a5604               |
|                                   | 6b0f175674389632-17 .crayon-line} |
|                                   | [                                 |
|                                   | ]{.crayo                          |
|                                   | n-h}[\$vmStatusIndex]{.crayon-v}[ |
|                                   | ]{.crayon-h}[=]{.crayon-o}[       |
|                                   | ]{.crayon-h}0[.]                  |
|                                   | {.crayon-sy}[.]{.crayon-sy}[(]{.c |
|                                   | rayon-sy}[\$vmStatus]{.crayon-v}[ |
|                                   | .]{.crayon-sy}[Count]{.crayon-i}[ |
|                                   | ]{.crayon-h                       |
|                                   | }[-1]{.crayon-cn}[)]{.crayon-sy}[ |
|                                   | ]{.crayon-h}[\|]{.crayon-o}[      |
|                                   | ]{.crayon-h}[where]{.crayon-st}[  |
|                                   | ]{.cray                           |
|                                   | on-h}[{]{.crayon-sy}[\$vmStatus]{ |
|                                   | .crayon-v}[\[]{.crayon-sy}[\$\_]{ |
|                                   | .crayon-v}[\]]{.crayon-sy}[.]{.cr |
|                                   | ayon-sy}[MachineName]{.crayon-i}[ |
|                                   | ]{.crayon-h}[-eq]{.crayon-cn}[    |
|                                   | ]{.crayon-h}[\$compl              |
|                                   | etedJob]{.crayon-v}[.]{.crayon-sy |
|                                   | }[Name]{.crayon-i}[}]{.crayon-sy} |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#c                           |
|                                   | rayon-5a56046b0f175674389632-18 . |
|                                   | crayon-line .crayon-striped-line} |
|                                   | [                                 |
|                                   | ]{.crayon-h}[\$vmStatus]{.crayo   |
|                                   | n-v}[\[]{.crayon-sy}[\$vmStatusIn |
|                                   | dex]{.crayon-v}[\]]{.crayon-sy}[. |
|                                   | ]{.crayon-sy}[Cloned]{.crayon-i}[ |
|                                   | ]{.crayon-h}[=]{.crayon-o}[       |
|                                   | ]{.crayon-h}1                     |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#crayon-5a5604               |
|                                   | 6b0f175674389632-19 .crayon-line} |
|                                   | [ ]{.crayon-h}[}]{.crayon-sy}     |
|                                   | :::                               |
|                                   | ::::::::::::::::::::::            |
+-----------------------------------+-----------------------------------+
:::
::::::::::::::::::::

Finally, the rest of the loop to complete the remaining tasks. I have
stripped out much of the code within each task to keep this looking
simple for the purpose of the article.

:::::::::::::::::::: {#crayon-5a56046b0f17e671646460 .crayon-syntax .crayon-theme-classic .crayon-font-monaco .crayon-os-pc .print-yes .notranslate settings=" minimize scroll-mouseover" style=" margin-top: 12px; margin-bottom: 12px; font-size: 12px !important; line-height: 15px !important;"}
:::::::::::::::: {.crayon-toolbar settings=" mouseover overlay hide delay" style="font-size: 12px !important;height: 18px !important; line-height: 18px !important;"}
[]{.crayon-title}

::::::::::::::: {.crayon-tools style="font-size: 12px !important;height: 18px !important; line-height: 18px !important;"}
:::: {.crayon-button .crayon-nums-button title="Toggle Line Numbers"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-plain-button title="Toggle Plain Code"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-wrap-button title="Toggle Line Wrap"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-expand-button title="Expand Code"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-copy-button title="Copy"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-popup-button title="Open Code In New Window"}
::: crayon-button-icon
:::
::::

[PowerShell]{.crayon-language}
:::::::::::::::
::::::::::::::::

::: {.crayon-info style="min-height: 16.8px !important; line-height: 16.8px !important;"}
:::

::: crayon-plain-wrap
:::

::: {.crayon-main style=""}
+-----------------------------------+-----------------------------------+
| ::::::::::::::::::::::::::::::    | :::::::::::::::::::               |
| :::::::::: {.crayon-nums-content  | ::::::::::::::::::::: {.crayon-pr |
| style="font-size: 12px !important | e style="font-size: 12px !importa |
| ; line-height: 15px !important;"} | nt; line-height: 15px !important; |
| ::: {.crayon-num line="           |  -moz-tab-size:4; -o-tab-size:4;  |
| crayon-5a56046b0f17e671646460-1"} | -webkit-tab-size:4; tab-size:4;"} |
| 1                                 | ::: {#crayon-5a560                |
| :::                               | 46b0f17e671646460-1 .crayon-line} |
|                                   | [                                 |
| ::: {.cray                        | ]{.crayon-h}                      |
| on-num .crayon-striped-num line=" | [\###################]{.crayon-c} |
| crayon-5a56046b0f17e671646460-2"} | :::                               |
| 2                                 |                                   |
| :::                               | ::: {#                            |
|                                   | crayon-5a56046b0f17e671646460-2 . |
| ::: {.crayon-num line="           | crayon-line .crayon-striped-line} |
| crayon-5a56046b0f17e671646460-3"} | [ ]{.crayon-h}[\# Power on VMs    |
| 3                                 | which have finished               |
| :::                               | cloning]{.crayon-c}               |
|                                   | :::                               |
| ::: {.cray                        |                                   |
| on-num .crayon-striped-num line=" | ::: {#crayon-5a560                |
| crayon-5a56046b0f17e671646460-4"} | 46b0f17e671646460-3 .crayon-line} |
| 4                                 | [                                 |
| :::                               | ]{.crayon-h}                      |
|                                   | [\###################]{.crayon-c} |
| ::: {.crayon-num line="           | :::                               |
| crayon-5a56046b0f17e671646460-5"} |                                   |
| 5                                 | ::: {#                            |
| :::                               | crayon-5a56046b0f17e671646460-4 . |
|                                   | crayon-line .crayon-striped-line} |
| ::: {.cray                        |                                   |
| on-num .crayon-striped-num line=" | :::                               |
| crayon-5a56046b0f17e671646460-6"} |                                   |
| 6                                 | ::: {#crayon-5a560                |
| :::                               | 46b0f17e671646460-5 .crayon-line} |
|                                   | [                                 |
| ::: {.crayon-num line="           | ]{.                               |
| crayon-5a56046b0f17e671646460-7"} | crayon-h}[\$vmStatus]{.crayon-v}[ |
| 7                                 | ]{.crayon-h}[\|]{.crayon-o}[      |
| :::                               | ]{.crayon-h}[where]{.crayon-st}[  |
|                                   | ]{.crayon-h}[{]{.crayon-sy}[      |
| ::: {.cray                        | (]{.crayon-sy}[\$\_]{.crayon-v}[. |
| on-num .crayon-striped-num line=" | ]{.crayon-sy}[Cloned]{.crayon-i}[ |
| crayon-5a56046b0f17e671646460-8"} | ]{.crayon-h}[-ne]{.crayon-cn}[    |
| 8                                 | ]{.crayon-h}0[)]{.crayon-sy}[     |
| :::                               | ]{.crayon-h}[-and]{.crayon-cn}[   |
|                                   | ]{.crayon-h}[(]{                  |
| ::: {.crayon-num line="           | .crayon-sy}[\$\_]{.crayon-v}[.]{. |
| crayon-5a56046b0f17e671646460-9"} | crayon-sy}[PoweredOn]{.crayon-i}[ |
| 9                                 | ]{.crayon-h}[-eq]{.crayon-cn}[    |
| :::                               | ]{.crayon-h                       |
|                                   | }0[)]{.crayon-sy}[}]{.crayon-sy}[ |
| ::: {.crayo                       | ]{.crayon-h}[\|]{.crayon-o}[      |
| n-num .crayon-striped-num line="c | ]                                 |
| rayon-5a56046b0f17e671646460-10"} | {.crayon-h}[foreach]{.crayon-st}[ |
| 10                                | ]{.crayon-h}[{]{.crayon-sy}       |
| :::                               | :::                               |
|                                   |                                   |
| ::: {.crayon-num line="c          | ::: {#                            |
| rayon-5a56046b0f17e671646460-11"} | crayon-5a56046b0f17e671646460-6 . |
| 11                                | crayon-line .crayon-striped-line} |
| :::                               |                                   |
|                                   | :::                               |
| ::: {.crayo                       |                                   |
| n-num .crayon-striped-num line="c | ::: {#crayon-5a560                |
| rayon-5a56046b0f17e671646460-12"} | 46b0f17e671646460-7 .crayon-line} |
| 12                                | [ ]{.crayon-h}[\# start the       |
| :::                               | VM]{.crayon-c}                    |
|                                   | :::                               |
| ::: {.crayon-num line="c          |                                   |
| rayon-5a56046b0f17e671646460-13"} | ::: {#                            |
| 13                                | crayon-5a56046b0f17e671646460-8 . |
| :::                               | crayon-line .crayon-striped-line} |
|                                   | [                                 |
| ::: {.crayo                       | ]                                 |
| n-num .crayon-striped-num line="c | {.crayon-h}[Start-VM]{.crayon-r}[ |
| rayon-5a56046b0f17e671646460-14"} | ]{                                |
| 14                                | .crayon-h}[\$\_]{.crayon-v}[.]{.c |
| :::                               | rayon-sy}[MachineName]{.crayon-i} |
|                                   | :::                               |
| ::: {.crayon-num line="c          |                                   |
| rayon-5a56046b0f17e671646460-15"} | ::: {#crayon-5a560                |
| 15                                | 46b0f17e671646460-9 .crayon-line} |
| :::                               |                                   |
|                                   | :::                               |
| ::: {.crayo                       |                                   |
| n-num .crayon-striped-num line="c | ::: {#c                           |
| rayon-5a56046b0f17e671646460-16"} | rayon-5a56046b0f17e671646460-10 . |
| 16                                | crayon-line .crayon-striped-line} |
| :::                               | [ ]{.crayon-h}[\# update          |
|                                   | \$vmStatus]{.crayon-c}            |
| ::: {.crayon-num line="c          | :::                               |
| rayon-5a56046b0f17e671646460-17"} |                                   |
| 17                                | ::: {#crayon-5a5604               |
| :::                               | 6b0f17e671646460-11 .crayon-line} |
|                                   | [                                 |
| ::: {.crayo                       | ]                                 |
| n-num .crayon-striped-num line="c | {.crayon-h}[\$\_]{.crayon-v}[.]{. |
| rayon-5a56046b0f17e671646460-18"} | crayon-sy}[PoweredOn]{.crayon-i}[ |
| 18                                | ]{.crayon-h}[=]{.crayon-o}[       |
| :::                               | ]{.crayon-h}1                     |
|                                   | :::                               |
| ::: {.crayon-num line="c          |                                   |
| rayon-5a56046b0f17e671646460-19"} | ::: {#c                           |
| 19                                | rayon-5a56046b0f17e671646460-12 . |
| :::                               | crayon-line .crayon-striped-line} |
|                                   | [ ]{.crayon-h}[}]{.crayon-sy}[    |
| ::: {.crayo                       | ]{.crayon-h}                      |
| n-num .crayon-striped-num line="c | :::                               |
| rayon-5a56046b0f17e671646460-20"} |                                   |
| 20                                | ::: {#crayon-5a5604               |
| :::                               | 6b0f17e671646460-13 .crayon-line} |
|                                   |                                   |
| ::: {.crayon-num line="c          | :::                               |
| rayon-5a56046b0f17e671646460-21"} |                                   |
| 21                                | ::: {#c                           |
| :::                               | rayon-5a56046b0f17e671646460-14 . |
|                                   | crayon-line .crayon-striped-line} |
| ::: {.crayo                       | [                                 |
| n-num .crayon-striped-num line="c | ]{.crayon-h}                      |
| rayon-5a56046b0f17e671646460-22"} | [\###################]{.crayon-c} |
| 22                                | :::                               |
| :::                               |                                   |
|                                   | ::: {#crayon-5a5604               |
| ::: {.crayon-num line="c          | 6b0f17e671646460-15 .crayon-line} |
| rayon-5a56046b0f17e671646460-23"} | [ ]{.crayon-h}[\# Set correct IP  |
| 23                                | address for VMs and start Citrix  |
| :::                               | service]{.crayon-c}               |
|                                   | :::                               |
| ::: {.crayo                       |                                   |
| n-num .crayon-striped-num line="c | ::: {#c                           |
| rayon-5a56046b0f17e671646460-24"} | rayon-5a56046b0f17e671646460-16 . |
| 24                                | crayon-line .crayon-striped-line} |
| :::                               | [                                 |
|                                   | ]{.crayon-h}                      |
| ::: {.crayon-num line="c          | [\###################]{.crayon-c} |
| rayon-5a56046b0f17e671646460-25"} | :::                               |
| 25                                |                                   |
| :::                               | ::: {#crayon-5a5604               |
|                                   | 6b0f17e671646460-17 .crayon-line} |
| ::: {.crayo                       |                                   |
| n-num .crayon-striped-num line="c | :::                               |
| rayon-5a56046b0f17e671646460-26"} |                                   |
| 26                                | ::: {#c                           |
| :::                               | rayon-5a56046b0f17e671646460-18 . |
|                                   | crayon-line .crayon-striped-line} |
| ::: {.crayon-num line="c          | [                                 |
| rayon-5a56046b0f17e671646460-27"} | ]{.                               |
| 27                                | crayon-h}[\$vmStatus]{.crayon-v}[ |
| :::                               | ]{.crayon-h}[\|]{.crayon-o}[      |
|                                   | ]{.crayon-h}[where]{.crayon-st}[  |
| ::: {.crayo                       | ]{.crayon-h}[{]{.crayon-sy}[      |
| n-num .crayon-striped-num line="c | (]{.crayon-sy}[\$\_]{.crayon-v}[. |
| rayon-5a56046b0f17e671646460-28"} | ]{.crayon-sy}[Cloned]{.crayon-i}[ |
| 28                                | ]{.crayon-h}[-ne]{.crayon-cn}[    |
| :::                               | ]{.crayon-h}0[)]{.crayon-sy}[     |
|                                   | ]{.crayon-h}[-and]{.crayon-cn}[   |
| ::: {.crayon-num line="c          | ]{.crayon-h}[(]{                  |
| rayon-5a56046b0f17e671646460-29"} | .crayon-sy}[\$\_]{.crayon-v}[.]{. |
| 29                                | crayon-sy}[PoweredOn]{.crayon-i}[ |
| :::                               | ]{.crayon-h}[-ne]{.crayon-cn}[    |
|                                   | ]{.crayon-h}0[)]{.crayon-sy}[     |
| ::: {.crayo                       | ]{.crayon-h}[-and]{.crayon-cn}[   |
| n-num .crayon-striped-num line="c | ]{.crayon-h                       |
| rayon-5a56046b0f17e671646460-30"} | }[(]{.crayon-sy}[\$\_]{.crayon-v} |
| 30                                | [.]{.crayon-sy}[ReIP]{.crayon-i}[ |
| :::                               | ]{.crayon-h}[-eq]{.crayon-cn}[    |
|                                   | ]{.crayon-h                       |
| ::: {.crayon-num line="c          | }0[)]{.crayon-sy}[}]{.crayon-sy}[ |
| rayon-5a56046b0f17e671646460-31"} | ]{.crayon-h}[\|]{.crayon-o}[      |
| 31                                | ]                                 |
| :::                               | {.crayon-h}[foreach]{.crayon-st}[ |
|                                   | ]{.crayon-h}[{]{.crayon-sy}       |
| ::: {.crayo                       | :::                               |
| n-num .crayon-striped-num line="c |                                   |
| rayon-5a56046b0f17e671646460-32"} | ::: {#crayon-5a5604               |
| 32                                | 6b0f17e671646460-19 .crayon-line} |
| :::                               |                                   |
|                                   | :::                               |
| ::: {.crayon-num line="c          |                                   |
| rayon-5a56046b0f17e671646460-33"} | ::: {#c                           |
| 33                                | rayon-5a56046b0f17e671646460-20 . |
| :::                               | crayon-line .crayon-striped-line} |
|                                   | [ ]{.crayon-h}[\# check machine   |
| ::: {.crayo                       | is online]{.crayon-c}             |
| n-num .crayon-striped-num line="c | :::                               |
| rayon-5a56046b0f17e671646460-34"} |                                   |
| 34                                | ::: {#crayon-5a5604               |
| :::                               | 6b0f17e671646460-21 .crayon-line} |
|                                   | [ ]{.crayon-h}[if]{.crayon-st}[   |
| ::: {.crayon-num line="c          | ]{.crayon-h}[(]{.crayon           |
| rayon-5a56046b0f17e671646460-35"} | -sy}[Test-Connection]{.crayon-r}[ |
| 35                                | ]{.                               |
| :::                               | crayon-h}[\$\_]{.crayon-v}[.]{.cr |
|                                   | ayon-sy}[MachineName]{.crayon-i}[ |
| ::: {.crayo                       | ]{.crayon-h}[-Count]{.crayon-cn}[ |
| n-num .crayon-striped-num line="c | ]{.crayon-h}1[                    |
| rayon-5a56046b0f17e671646460-36"} | ]{.cra                            |
| 36                                | yon-h}[-ErrorAction]{.crayon-cn}[ |
| :::                               | ]{.crayon-h}[SilentlyCon          |
|                                   | tinue]{.crayon-i}[)]{.crayon-sy}[ |
| ::: {.crayon-num line="c          | ]{.crayon-h}[{]{.crayon-sy}       |
| rayon-5a56046b0f17e671646460-37"} | :::                               |
| 37                                |                                   |
| :::                               | ::: {#c                           |
| :::::::                           | rayon-5a56046b0f17e671646460-22 . |
| ::::::::::::::::::::::::::::::::: | crayon-line .crayon-striped-line} |
|                                   |                                   |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#crayon-5a5604               |
|                                   | 6b0f17e671646460-23 .crayon-line} |
|                                   | [            ]{.crayon-h}[\# code |
|                                   | to set IP address - mentioned     |
|                                   | later in blog post]{.crayon-c}    |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#c                           |
|                                   | rayon-5a56046b0f17e671646460-24 . |
|                                   | crayon-line .crayon-striped-line} |
|                                   | [ ]{.crayon-h}[}]{.crayon-sy}     |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#crayon-5a5604               |
|                                   | 6b0f17e671646460-25 .crayon-line} |
|                                   | [ ]{.crayon-h}[}]{.crayon-sy}     |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#c                           |
|                                   | rayon-5a56046b0f17e671646460-26 . |
|                                   | crayon-line .crayon-striped-line} |
|                                   |                                   |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#crayon-5a5604               |
|                                   | 6b0f17e671646460-27 .crayon-line} |
|                                   | [                                 |
|                                   | ]{.crayon-h}                      |
|                                   | [\###################]{.crayon-c} |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#c                           |
|                                   | rayon-5a56046b0f17e671646460-28 . |
|                                   | crayon-line .crayon-striped-line} |
|                                   | [ ]{.crayon-h}[\# Update counters |
|                                   | and sleep]{.crayon-c}             |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#crayon-5a5604               |
|                                   | 6b0f17e671646460-29 .crayon-line} |
|                                   | [                                 |
|                                   | ]{.crayon-h}                      |
|                                   | [\###################]{.crayon-c} |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#c                           |
|                                   | rayon-5a56046b0f17e671646460-30 . |
|                                   | crayon-line .crayon-striped-line} |
|                                   |                                   |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#crayon-5a5604               |
|                                   | 6b0f17e671646460-31 .crayon-line} |
|                                   | [ ]{.crayon-h}[\# update          |
|                                   | counters]{.crayon-c}              |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#c                           |
|                                   | rayon-5a56046b0f17e671646460-32 . |
|                                   | crayon-line .crayon-striped-line} |
|                                   | [                                 |
|                                   | ]{.crayon-h}[\[]{.crayon-sy}[     |
|                                   | ARRAY]{.crayon-t}[\]]{.crayon-sy} |
|                                   | [\$vmsCloneInitiated]{.crayon-v}[ |
|                                   | ]{.crayon-h}[=]{.crayon-o}[       |
|                                   | ]{.                               |
|                                   | crayon-h}[\$vmStatus]{.crayon-v}[ |
|                                   | ]{.crayon-h}[\|]{.crayon-o}[      |
|                                   | ]{.crayon-h}[?]{.crayon-sy}[      |
|                                   | ]{.cray                           |
|                                   | on-h}[CloneInitiated]{.crayon-i}[ |
|                                   | ]{.crayon-h}[-eq]{.crayon-cn}[    |
|                                   | ]{.crayon-h}1                     |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#crayon-5a5604               |
|                                   | 6b0f17e671646460-33 .crayon-line} |
|                                   | [                                 |
|                                   | ]{.crayon-h}[\[]{.crayon          |
|                                   | -sy}[ARRAY]{.crayon-t}[\]]{.crayo |
|                                   | n-sy}[\$vmsCompleted]{.crayon-v}[ |
|                                   | ]{.crayon-h}[=]{.crayon-o}[       |
|                                   | ]{.                               |
|                                   | crayon-h}[\$vmStatus]{.crayon-v}[ |
|                                   | ]{.crayon-h}[\|]{.crayon-o}[      |
|                                   | ]{.crayon-h}[?]{.crayon-sy}[      |
|                                   | ]{.crayon-h}[{]{.crayon-sy}[      |
|                                   | (]{.crayon-sy}[\$\_]{.crayon-v}[. |
|                                   | ]{.crayon-sy}[Cloned]{.crayon-i}[ |
|                                   | ]{.crayon-h}[-ne]{.crayon-cn}[    |
|                                   | ]{.crayon-h}0[)]{.crayon-sy}[     |
|                                   | ]{.crayon-h}[-and]{.crayon-cn}[   |
|                                   | ]{.crayon-h}[(]{                  |
|                                   | .crayon-sy}[\$\_]{.crayon-v}[.]{. |
|                                   | crayon-sy}[PoweredOn]{.crayon-i}[ |
|                                   | ]{.crayon-h}[-ne]{.crayon-cn}[    |
|                                   | ]{.crayon-h}0[)]{.crayon-sy}[     |
|                                   | ]{.crayon-h}[-and]{.crayon-cn}[   |
|                                   | ]{.crayon-h                       |
|                                   | }[(]{.crayon-sy}[\$\_]{.crayon-v} |
|                                   | [.]{.crayon-sy}[ReIP]{.crayon-i}[ |
|                                   | ]{.crayon-h}[-ne]{.crayon-cn}[    |
|                                   | ]{.crayon-                        |
|                                   | h}0[)]{.crayon-sy}[}]{.crayon-sy} |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#c                           |
|                                   | rayon-5a56046b0f17e671646460-34 . |
|                                   | crayon-line .crayon-striped-line} |
|                                   |                                   |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#crayon-5a5604               |
|                                   | 6b0f17e671646460-35 .crayon-line} |
|                                   | [ ]{.crayon-h}[\# sleep for       |
|                                   | \$cycleTime seconds]{.crayon-c}   |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#c                           |
|                                   | rayon-5a56046b0f17e671646460-36 . |
|                                   | crayon-line .crayon-striped-line} |
|                                   | [                                 |
|                                   | ]{.c                              |
|                                   | rayon-h}[Start-Sleep]{.crayon-r}[ |
|                                   | ]{.                               |
|                                   | crayon-h}[\$cycleTime]{.crayon-v} |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#crayon-5a5604               |
|                                   | 6b0f17e671646460-37 .crayon-line} |
|                                   | [}]{.crayon-sy}                   |
|                                   | :::                               |
|                                   | :::::::                           |
|                                   | ::::::::::::::::::::::::::::::::: |
+-----------------------------------+-----------------------------------+
:::
::::::::::::::::::::

## Changing the IP address of a VM once deployed

I mentioned earlier I didn't get along with the guest customisation
method of setting the VM's IP address. In the example above you can see
where I set the IP address -- one of the last things I do. I removed the
code snippet for clarity in the article but here it is. Note, I also
flush DNS and wait 15 seconds before continuing:

:::::::::::::::::::: {#crayon-5a56046b0f186313510163 .crayon-syntax .crayon-theme-classic .crayon-font-monaco .crayon-os-pc .print-yes .notranslate settings=" minimize scroll-mouseover" style=" margin-top: 12px; margin-bottom: 12px; font-size: 12px !important; line-height: 15px !important;"}
:::::::::::::::: {.crayon-toolbar settings=" mouseover overlay hide delay" style="font-size: 12px !important;height: 18px !important; line-height: 18px !important;"}
[]{.crayon-title}

::::::::::::::: {.crayon-tools style="font-size: 12px !important;height: 18px !important; line-height: 18px !important;"}
:::: {.crayon-button .crayon-nums-button title="Toggle Line Numbers"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-plain-button title="Toggle Plain Code"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-wrap-button title="Toggle Line Wrap"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-expand-button title="Expand Code"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-copy-button title="Copy"}
::: crayon-button-icon
:::
::::

:::: {.crayon-button .crayon-popup-button title="Open Code In New Window"}
::: crayon-button-icon
:::
::::

[PowerShell]{.crayon-language}
:::::::::::::::
::::::::::::::::

::: {.crayon-info style="min-height: 16.8px !important; line-height: 16.8px !important;"}
:::

::: crayon-plain-wrap
:::

::: {.crayon-main style=""}
+-----------------------------------+-----------------------------------+
| :::::::::::::::::::::::::::::     | ::::::::::::::::::                |
| :::::::::: {.crayon-nums-content  | ::::::::::::::::::::: {.crayon-pr |
| style="font-size: 12px !important | e style="font-size: 12px !importa |
| ; line-height: 15px !important;"} | nt; line-height: 15px !important; |
| ::: {.crayon-num line="           |  -moz-tab-size:4; -o-tab-size:4;  |
| crayon-5a56046b0f186313510163-1"} | -webkit-tab-size:4; tab-size:4;"} |
| 1                                 | ::: {#crayon-5a560                |
| :::                               | 46b0f186313510163-1 .crayon-line} |
|                                   | [\$vmStatus]{.crayon-v}[          |
| ::: {.cray                        | ]{.crayon-h}[\|]{.crayon-o}[      |
| on-num .crayon-striped-num line=" | ]{.crayon-h}[where]{.crayon-st}[  |
| crayon-5a56046b0f186313510163-2"} | ]{.crayon-h}[{]{.crayon-sy}[      |
| 2                                 | (]{.crayon-sy}[\$\_]{.crayon-v}[. |
| :::                               | ]{.crayon-sy}[Cloned]{.crayon-i}[ |
|                                   | ]{.crayon-h}[-ne]{.crayon-cn}[    |
| ::: {.crayon-num line="           | ]{.crayon-h}0[)]{.crayon-sy}[     |
| crayon-5a56046b0f186313510163-3"} | ]{.crayon-h}[-and]{.crayon-cn}[   |
| 3                                 | ]{.crayon-h}[(]{                  |
| :::                               | .crayon-sy}[\$\_]{.crayon-v}[.]{. |
|                                   | crayon-sy}[PoweredOn]{.crayon-i}[ |
| ::: {.cray                        | ]{.crayon-h}[-ne]{.crayon-cn}[    |
| on-num .crayon-striped-num line=" | ]{.crayon-h}0[)]{.crayon-sy}[     |
| crayon-5a56046b0f186313510163-4"} | ]{.crayon-h}[-and]{.crayon-cn}[   |
| 4                                 | ]{.crayon-h                       |
| :::                               | }[(]{.crayon-sy}[\$\_]{.crayon-v} |
|                                   | [.]{.crayon-sy}[InAD]{.crayon-i}[ |
| ::: {.crayon-num line="           | ]{.crayon-h}[-ne]{.crayon-cn}[    |
| crayon-5a56046b0f186313510163-5"} | ]{.crayon-h}0[)]{.crayon-sy}[     |
| 5                                 | ]{.crayon-h}[-and]{.crayon-cn}[   |
| :::                               | ]{.crayon-h}[(]{.c                |
|                                   | rayon-sy}[\$\_]{.crayon-v}[.]{.cr |
| ::: {.cray                        | ayon-sy}[ActivateWin]{.crayon-i}[ |
| on-num .crayon-striped-num line=" | ]{.crayon-h}[-ne]{.crayon-cn}[    |
| crayon-5a56046b0f186313510163-6"} | ]{.crayon-h}0[)]{.crayon-sy}[     |
| 6                                 | ]{.crayon-h}[-and]{.crayon-cn}[   |
| :::                               | ]{.crayon-h}[(]{.cray             |
|                                   | on-sy}[\$\_]{.crayon-v}[.]{.crayo |
| ::: {.crayon-num line="           | n-sy}[ActivateOffice]{.crayon-i}[ |
| crayon-5a56046b0f186313510163-7"} | ]{.crayon-h}[-ne]{.crayon-cn}[    |
| 7                                 | ]{.crayon-h}0[)]{.crayon-sy}[     |
| :::                               | ]{.crayon-h}[-and]{.crayon-cn}[   |
|                                   | ]{.crayon-h                       |
| ::: {.cray                        | }[(]{.crayon-sy}[\$\_]{.crayon-v} |
| on-num .crayon-striped-num line=" | [.]{.crayon-sy}[ReIP]{.crayon-i}[ |
| crayon-5a56046b0f186313510163-8"} | ]{.crayon-h}[-eq]{.crayon-cn}[    |
| 8                                 | ]{.crayon-h                       |
| :::                               | }0[)]{.crayon-sy}[}]{.crayon-sy}[ |
|                                   | ]{.crayon-h}[\|]{.crayon-o}[      |
| ::: {.crayon-num line="           | ]                                 |
| crayon-5a56046b0f186313510163-9"} | {.crayon-h}[foreach]{.crayon-st}[ |
| 9                                 | ]{.crayon-h}[{]{.crayon-sy}       |
| :::                               | :::                               |
|                                   |                                   |
| ::: {.crayo                       | ::: {#                            |
| n-num .crayon-striped-num line="c | crayon-5a56046b0f186313510163-2 . |
| rayon-5a56046b0f186313510163-10"} | crayon-line .crayon-striped-line} |
| 10                                |                                   |
| :::                               | :::                               |
|                                   |                                   |
| ::: {.crayon-num line="c          | ::: {#crayon-5a560                |
| rayon-5a56046b0f186313510163-11"} | 46b0f186313510163-3 .crayon-line} |
| 11                                | [ ]{.crayon-h}[\# check machine   |
| :::                               | is online]{.crayon-c}             |
|                                   | :::                               |
| ::: {.crayo                       |                                   |
| n-num .crayon-striped-num line="c | ::: {#                            |
| rayon-5a56046b0f186313510163-12"} | crayon-5a56046b0f186313510163-4 . |
| 12                                | crayon-line .crayon-striped-line} |
| :::                               | [ ]{.crayon-h}[if]{.crayon-st}[   |
|                                   | ]{.crayon-h}[(]{.crayon           |
| ::: {.crayon-num line="c          | -sy}[Test-Connection]{.crayon-r}[ |
| rayon-5a56046b0f186313510163-13"} | ]{.                               |
| 13                                | crayon-h}[\$\_]{.crayon-v}[.]{.cr |
| :::                               | ayon-sy}[MachineName]{.crayon-i}[ |
|                                   | ]{.crayon-h}[-Count]{.crayon-cn}[ |
| ::: {.crayo                       | ]{.crayon-h}1[                    |
| n-num .crayon-striped-num line="c | ]{.cra                            |
| rayon-5a56046b0f186313510163-14"} | yon-h}[-ErrorAction]{.crayon-cn}[ |
| 14                                | ]{.crayon-h}[SilentlyCon          |
| :::                               | tinue]{.crayon-i}[)]{.crayon-sy}[ |
|                                   | ]{.crayon-h}[{]{.crayon-sy}       |
| ::: {.crayon-num line="c          | :::                               |
| rayon-5a56046b0f186313510163-15"} |                                   |
| 15                                | ::: {#crayon-5a560                |
| :::                               | 46b0f186313510163-5 .crayon-line} |
|                                   |                                   |
| ::: {.crayo                       | :::                               |
| n-num .crayon-striped-num line="c |                                   |
| rayon-5a56046b0f186313510163-16"} | ::: {#                            |
| 16                                | crayon-5a56046b0f186313510163-6 . |
| :::                               | crayon-line .crayon-striped-line} |
|                                   | [                                 |
| ::: {.crayon-num line="c          | ]{.                               |
| rayon-5a56046b0f186313510163-17"} | crayon-h}[Write-Host]{.crayon-r}[ |
| 17                                | ]{.                               |
| :::                               | crayon-h}[\"\$(\$\_.MachineName): |
|                                   | Online, changing IP               |
| ::: {.crayo                       | address\"]{.crayon-s}             |
| n-num .crayon-striped-num line="c | :::                               |
| rayon-5a56046b0f186313510163-18"} |                                   |
| 18                                | ::: {#crayon-5a560                |
| :::                               | 46b0f186313510163-7 .crayon-line} |
|                                   |                                   |
| ::: {.crayon-num line="c          | :::                               |
| rayon-5a56046b0f186313510163-19"} |                                   |
| 19                                | ::: {#                            |
| :::                               | crayon-5a56046b0f186313510163-8 . |
|                                   | crayon-line .crayon-striped-line} |
| ::: {.crayo                       | [ ]{.crayon-h}[\# change IP       |
| n-num .crayon-striped-num line="c | settings]{.crayon-c}              |
| rayon-5a56046b0f186313510163-20"} | :::                               |
| 20                                |                                   |
| :::                               | ::: {#crayon-5a560                |
|                                   | 46b0f186313510163-9 .crayon-line} |
| ::: {.crayon-num line="c          | [ ]{.crayon-h}[try]{.crayon-e}[   |
| rayon-5a56046b0f186313510163-21"} | ]{.crayon-h}[{]{.crayon-sy}       |
| 21                                | :::                               |
| :::                               |                                   |
|                                   | ::: {#c                           |
| ::: {.crayo                       | rayon-5a56046b0f186313510163-10 . |
| n-num .crayon-striped-num line="c | crayon-line .crayon-striped-line} |
| rayon-5a56046b0f186313510163-22"} | [                                 |
| 22                                | ]{.crayon-h}[Get-VM]{.crayon-r}[  |
| :::                               | ]{.crayon-h}[-Name]{.crayon-cn}[  |
|                                   | ]{.                               |
| ::: {.crayon-num line="c          | crayon-h}[\$\_]{.crayon-v}[.]{.cr |
| rayon-5a56046b0f186313510163-23"} | ayon-sy}[MachineName]{.crayon-i}[ |
| 23                                | ]{.crayon-h}[\|]{.crayon-o}[      |
| :::                               | ]{.crayon-h}[Get-VMG              |
|                                   | uestNetworkInterface]{.crayon-r}[ |
| ::: {.crayo                       | ]{.crayon-h}[\`]{.crayon-sy}      |
| n-num .crayon-striped-num line="c | :::                               |
| rayon-5a56046b0f186313510163-24"} |                                   |
| 24                                | ::: {#crayon-5a5604               |
| :::                               | 6b0f186313510163-11 .crayon-line} |
|                                   | [                                 |
| ::: {.crayon-num line="c          | ]{.c                              |
| rayon-5a56046b0f186313510163-25"} | rayon-h}[-Guestuser]{.crayon-cn}[ |
| 25                                | ]{.crayon                         |
| :::                               | -h}[\$localAdminUser]{.crayon-v}[ |
|                                   | ]{.crayon-h}[\`]{.crayon-sy}      |
| ::: {.crayo                       | :::                               |
| n-num .crayon-striped-num line="c |                                   |
| rayon-5a56046b0f186313510163-26"} | ::: {#c                           |
| 26                                | rayon-5a56046b0f186313510163-12 . |
| :::                               | crayon-line .crayon-striped-line} |
|                                   | [                                 |
| ::: {.crayon-num line="c          | ]{.crayo                          |
| rayon-5a56046b0f186313510163-27"} | n-h}[-GuestPassword]{.crayon-cn}[ |
| 27                                | ]{.crayon-h}[                     |
| :::                               | \$localAdminPassword]{.crayon-v}[ |
|                                   | ]{.crayon-h}[\|]{.crayon-o}[      |
| ::: {.crayo                       | ]{.crayon-h}[\`]{.crayon-sy}      |
| n-num .crayon-striped-num line="c | :::                               |
| rayon-5a56046b0f186313510163-28"} |                                   |
| 28                                | ::: {#crayon-5a5604               |
| :::                               | 6b0f186313510163-13 .crayon-line} |
|                                   | [ ]{.crayon-h}[?]{.crayon-sy}[    |
| ::: {.crayon-num line="c          | ]{.crayon-h}[{]{.crayon-sy}[      |
| rayon-5a56046b0f186313510163-29"} | ]{.crayon-h}[\$\_]{.crayon-v}     |
| 29                                | [.]{.crayon-sy}[name]{.crayon-i}[ |
| :::                               | ]{.crayon-h}[-like]{.crayon-cn}[  |
|                                   | ]{.crayon-h}                      |
| ::: {.crayo                       | [\$localNICNameQuery]{.crayon-v}[ |
| n-num .crayon-striped-num line="c | ]{.crayon-h}[}]{.crayon-sy}[      |
| rayon-5a56046b0f186313510163-30"} | ]{.crayon-h}[\|]{.crayon-o}[      |
| 30                                | ]{.crayon-h}[\`]{.crayon-sy}      |
| :::                               | :::                               |
|                                   |                                   |
| ::: {.crayon-num line="c          | ::: {#c                           |
| rayon-5a56046b0f186313510163-31"} | rayon-5a56046b0f186313510163-14 . |
| 31                                | crayon-line .crayon-striped-line} |
| :::                               | [                                 |
|                                   | ]                                 |
| ::: {.crayo                       | {.crayon-h}[Set]{.crayon-r}[-VMGu |
| n-num .crayon-striped-num line="c | estNetworkInterface]{.crayon-cn}[ |
| rayon-5a56046b0f186313510163-32"} | ]{.crayon-h}[\`]{.crayon-sy}      |
| 32                                | :::                               |
| :::                               |                                   |
|                                   | ::: {#crayon-5a5604               |
| ::: {.crayon-num line="c          | 6b0f186313510163-15 .crayon-line} |
| rayon-5a56046b0f186313510163-33"} | [                                 |
| 33                                | ]{.c                              |
| :::                               | rayon-h}[-Guestuser]{.crayon-cn}[ |
|                                   | ]{.crayon                         |
| ::: {.crayo                       | -h}[\$localAdminUser]{.crayon-v}[ |
| n-num .crayon-striped-num line="c | ]{.crayon-h}[\`]{.crayon-sy}      |
| rayon-5a56046b0f186313510163-34"} | :::                               |
| 34                                |                                   |
| :::                               | ::: {#c                           |
|                                   | rayon-5a56046b0f186313510163-16 . |
| ::: {.crayon-num line="c          | crayon-line .crayon-striped-line} |
| rayon-5a56046b0f186313510163-35"} | [                                 |
| 35                                | ]{.crayo                          |
| :::                               | n-h}[-GuestPassword]{.crayon-cn}[ |
|                                   | ]{.crayon-h}[                     |
| ::: {.crayo                       | \$localAdminPassword]{.crayon-v}[ |
| n-num .crayon-striped-num line="c | ]{.crayon-h}[\`]{.crayon-sy}      |
| rayon-5a56046b0f186313510163-36"} | :::                               |
| 36                                |                                   |
| :::                               | ::: {#crayon-5a5604               |
| ::::::                            | 6b0f186313510163-17 .crayon-line} |
| ::::::::::::::::::::::::::::::::: | [                                 |
|                                   | ]{.                               |
|                                   | crayon-h}[-IPPolicy]{.crayon-cn}[ |
|                                   | ]{.crayon-h}[static]{.crayon-i}[  |
|                                   | ]{.crayon-h}[\`]{.crayon-sy}      |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#c                           |
|                                   | rayon-5a56046b0f186313510163-18 . |
|                                   | crayon-line .crayon-striped-line} |
|                                   | [ ]{.crayon-h}[-IP]{.crayon-cn}[  |
|                                   | ]                                 |
|                                   | {.crayon-h}[\$\_]{.crayon-v}[.]{. |
|                                   | crayon-sy}[IpAddress]{.crayon-i}[ |
|                                   | ]{.crayon-h}[\`]{.crayon-sy}      |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#crayon-5a5604               |
|                                   | 6b0f186313510163-19 .crayon-line} |
|                                   | [                                 |
|                                   | ]{                                |
|                                   | .crayon-h}[-Netmask]{.crayon-cn}[ |
|                                   | ]{.cr                             |
|                                   | ayon-h}[\$networkSubnet]{.crayon- |
|                                   | v}[  ]{.crayon-h}[\`]{.crayon-sy} |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#c                           |
|                                   | rayon-5a56046b0f186313510163-20 . |
|                                   | crayon-line .crayon-striped-line} |
|                                   | [                                 |
|                                   | ]{                                |
|                                   | .crayon-h}[-Gateway]{.crayon-cn}[ |
|                                   | ]{.crayon                         |
|                                   | -h}[\$networkGateway]{.crayon-v}[ |
|                                   | ]{.crayon-h}[\`]{.crayon-sy}      |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#crayon-5a5604               |
|                                   | 6b0f186313510163-21 .crayon-line} |
|                                   | [ ]{.crayon-h}[-DNS]{.crayon-cn}[ |
|                                   | ]{.c                              |
|                                   | rayon-h}[\$networkDns]{.crayon-v} |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#c                           |
|                                   | rayon-5a56046b0f186313510163-22 . |
|                                   | crayon-line .crayon-striped-line} |
|                                   |                                   |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#crayon-5a5604               |
|                                   | 6b0f186313510163-23 .crayon-line} |
|                                   | [                                 |
|                                   | ]{.                               |
|                                   | crayon-h}[Write-Host]{.crayon-r}[ |
|                                   | ]{.                               |
|                                   | crayon-h}[\"\$(\$\_.MachineName): |
|                                   | IP Address changed\"]{.crayon-s}  |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#c                           |
|                                   | rayon-5a56046b0f186313510163-24 . |
|                                   | crayon-line .crayon-striped-line} |
|                                   |                                   |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#crayon-5a5604               |
|                                   | 6b0f186313510163-25 .crayon-line} |
|                                   | [ ]{.crayon-h}[\# flush           |
|                                   | dns]{.crayon-c}                   |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#c                           |
|                                   | rayon-5a56046b0f186313510163-26 . |
|                                   | crayon-line .crayon-striped-line} |
|                                   | [                                 |
|                                   | ]                                 |
|                                   | {.crayon-h}[ipconfig]{.crayon-i}[ |
|                                   | ]{.crayon-h}[/                    |
|                                   | ]{.crayon-o}[flushdns]{.crayon-e} |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#crayon-5a5604               |
|                                   | 6b0f186313510163-27 .crayon-line} |
|                                   | [                ]{.c             |
|                                   | rayon-e}[Start-Sleep]{.crayon-r}[ |
|                                   | ]{.crayon-h}15                    |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#c                           |
|                                   | rayon-5a56046b0f186313510163-28 . |
|                                   | crayon-line .crayon-striped-line} |
|                                   |                                   |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#crayon-5a5604               |
|                                   | 6b0f186313510163-29 .crayon-line} |
|                                   | [         ]{.crayon-h}[\# update  |
|                                   | \$vmStatus ]{.crayon-c}           |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#c                           |
|                                   | rayon-5a56046b0f186313510163-30 . |
|                                   | crayon-line .crayon-striped-line} |
|                                   | [                                 |
|                                   | ]{.crayon-h}[\$\_]{.crayon-v}     |
|                                   | [.]{.crayon-sy}[ReIP]{.crayon-i}[ |
|                                   | ]{.crayon-h}[=]{.crayon-o}[       |
|                                   | ]{.crayon-h}1                     |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#crayon-5a5604               |
|                                   | 6b0f186313510163-31 .crayon-line} |
|                                   | [ ]{.crayon-h}[}]{.crayon-sy}     |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#c                           |
|                                   | rayon-5a56046b0f186313510163-32 . |
|                                   | crayon-line .crayon-striped-line} |
|                                   | [ ]{.crayon-h}[catch]{.crayon-e}[ |
|                                   | ]{.crayon-h}[{]{.crayon-sy}       |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#crayon-5a5604               |
|                                   | 6b0f186313510163-33 .crayon-line} |
|                                   | [                                 |
|                                   |   ]{.crayon-h}[\$null]{.crayon-v} |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#c                           |
|                                   | rayon-5a56046b0f186313510163-34 . |
|                                   | crayon-line .crayon-striped-line} |
|                                   | [ ]{.crayon-h}[}]{.crayon-sy}     |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#crayon-5a5604               |
|                                   | 6b0f186313510163-35 .crayon-line} |
|                                   | [ ]{.crayon-h}[}]{.crayon-sy}     |
|                                   | :::                               |
|                                   |                                   |
|                                   | ::: {#c                           |
|                                   | rayon-5a56046b0f186313510163-36 . |
|                                   | crayon-line .crayon-striped-line} |
|                                   | [ ]{.crayon-h}[}]{.crayon-sy}     |
|                                   | :::                               |
|                                   | ::::::                            |
|                                   | ::::::::::::::::::::::::::::::::: |
+-----------------------------------+-----------------------------------+
:::
::::::::::::::::::::

## Conclusion

That's all folks! As I mentioned previously I have published the entire
script for download but hopefully the above will give you a better
understanding of how this works than reading the script alone. There are
a few extra features in the full script which are not explained above,
such as selecting an appropriate datastore for each VM auto-magically,
activating Windows and Office and starting a Windows service. You can
disable these features using the \$enableFeature parameters at the top
of script.

Did you find this useful? I'd love to hear your feedback, or if you are
having any issues running the script or a variant of your own leave a
commend and I will do my best to help!

:::::::::: e-mailit_bottom_toolbox
::::::::: {.e-mailit_toolbox .square .size32 emailit-url="http://www.harryjohn.org/deploy-multiple-vms-from-template/" emailit-title="Deploy multiple VMs from template with PowerCLI"}
::: e-mailit_btn_Facebook
:::

::: e-mailit_btn_Twitter
:::

::: e-mailit_btn_Send_via_Email
:::

::: e-mailit_btn_Pinterest
:::

::: e-mailit_btn_LinkedIn
:::

::: e-mailit_btn_EMAILiT
:::
:::::::::
::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

This entry was posted in
[PowerCLI](http://www.harryjohn.org/category/powercli/){rel="tag"},
[VMware](http://www.harryjohn.org/category/vmware/){rel="tag"} and
tagged
[deployment](http://www.harryjohn.org/tag/deployment/){rel="tag"},
[PowerCLI](http://www.harryjohn.org/tag/powercli/){rel="tag"},
[scripts](http://www.harryjohn.org/tag/scripts/){rel="tag"},
[VMware](http://www.harryjohn.org/tag/vmware/){rel="tag"}. Bookmark the
[permalink](http://www.harryjohn.org/deploy-multiple-vms-from-template/ "Permalink to Deploy multiple VMs from template with PowerCLI"){rel="bookmark"}.
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

::: {.clearfix .shadowfix}
:::
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

# Post navigation {#post-navigation .assistive-text .section-heading}

::: nav-previous
[[←
Previous]{.meta-nav}](http://www.harryjohn.org/equallogic-vmware-iscsi-setup-part-3-powerconnect-configuration/){rel="prev"}
:::

::: nav-next
[[Next
→]{.meta-nav}](http://www.harryjohn.org/office-2016-install-we-need-to-remove-some-older-apps/){rel="next"}
:::

:::: {#comments}
## One thought on "Deploy multiple VMs from template with PowerCLI" {#comments-title}

1.  ::::::: {#li-comment-799}
    ::: {.comment-author .vcard}
    ![](http://1.gravatar.com/avatar/786ed2f89de6a867d6df15412883270c?s=40&d=mm&r=g){.avatar
    .avatar-40 .photo
    srcset="http://1.gravatar.com/avatar/786ed2f89de6a867d6df15412883270c?s=80&d=mm&r=g 2x"
    height="40" width="40"} bvi1998
    :::

    ::: {.comment-meta .commentmetadata}
    [January 21, 2014 at 9:00
    pm](http://www.harryjohn.org/deploy-multiple-vms-from-template/#comment-799)
    :::

    ::: comment-content
    Hi,\
    Thanks for sharing the script! I am having trouble with the job
    calling the deployvm script, it does not seem to have the
    machinename or datastore at that point. Can you explain the
    following?\
    -- when you set the job up, where are you getting all of those
    arguments?\
    -- when the job is started, more parameters are sent -- haven't they
    already been set up when you set up the job?\
    -- the deployvm script seems to be waiting for different named
    parameters

    Sorry it's just that I am not experienced enough in PS to understand
    all of this, though I have tried. Does the script run for you?

    Thanks!
    :::

    ::: reply
    [Reply](http://www.harryjohn.org/deploy-multiple-vms-from-template/?replytocom=799#respond){.comment-reply-link
    rel="nofollow"
    onclick="return addComment.moveForm( \"comment-799\", \"799\", \"respond\", \"204\" )"
    aria-label="Reply to bvi1998"}
    :::
    :::::::

::: {#respond .comment-respond}
### Leave a Reply [[Cancel reply](/deploy-multiple-vms-from-template/#respond){#cancel-comment-reply-link rel="nofollow" style="display:none;"}]{.small} {#reply-title .comment-reply-title}

[Your email address will not be published.]{#email-notes} Required
fields are marked [\*]{.required}

Comment

Name [\*]{.required}

Email [\*]{.required}

Website
:::
::::
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

::::: {#sidebar .widget-area .col300 role="complementary"}
::: {#social-media .clearfix}
:::

<div>

Search for:

</div>

## Recent Posts {#recent-posts .widget-title}

-   [Office 2016 Install -- "We need to remove some older
    apps"](http://www.harryjohn.org/office-2016-install-we-need-to-remove-some-older-apps/)
-   [Deploy multiple VMs from template with
    PowerCLI](http://www.harryjohn.org/deploy-multiple-vms-from-template/)
-   [EqualLogic VMware iSCSI Setup -- Part 3: PowerConnect
    Configuration](http://www.harryjohn.org/equallogic-vmware-iscsi-setup-part-3-powerconnect-configuration/)
-   [EqualLogic VMware iSCSI Setup -- Part 2: Network
    Configuration](http://www.harryjohn.org/equallogic-vmware-iscsi-setup-part-2-network/)
-   [Veeam Backup job failed. Cannot create a shadow copy of the volumes
    containing writer's
    data.](http://www.harryjohn.org/veeam-backup-job-failed-cannot-create-a-shadow-copy-of-the-volumes-containing-writers-data/)

## Recent Comments {#recent-comments .widget-title}

-   [[Rabin](http://www.universaltech.com.au){.url
    rel="external nofollow"}]{.comment-author-link} on [Office 2016
    Install -- "We need to remove some older
    apps"](http://www.harryjohn.org/office-2016-install-we-need-to-remove-some-older-apps/#comment-36046)
-   [Phil]{.comment-author-link} on [Office 2016 Install -- "We need to
    remove some older
    apps"](http://www.harryjohn.org/office-2016-install-we-need-to-remove-some-older-apps/#comment-35654)
-   [MG]{.comment-author-link} on [Office 2016 Install -- "We need to
    remove some older
    apps"](http://www.harryjohn.org/office-2016-install-we-need-to-remove-some-older-apps/#comment-35340)
-   [Vince F]{.comment-author-link} on [Office 2016 Install -- "We need
    to remove some older
    apps"](http://www.harryjohn.org/office-2016-install-we-need-to-remove-some-older-apps/#comment-35159)
-   [Steve]{.comment-author-link} on [Office 2016 Install -- "We need to
    remove some older
    apps"](http://www.harryjohn.org/office-2016-install-we-need-to-remove-some-older-apps/#comment-35066)

## Archives {#archives .widget-title}

-   [September 2015](http://www.harryjohn.org/2015/09/)
-   [January 2014](http://www.harryjohn.org/2014/01/)
-   [December 2013](http://www.harryjohn.org/2013/12/)
-   [February 2013](http://www.harryjohn.org/2013/02/)

## Categories {#categories .widget-title}

-   [EqualLogic VMware iSCSI
    Setup](http://www.harryjohn.org/category/equallogic-vmware-iscsi-setup/)
-   [PowerCLI](http://www.harryjohn.org/category/powercli/)
-   [Uncategorized](http://www.harryjohn.org/category/uncategorized/)
-   [VMware](http://www.harryjohn.org/category/vmware/)

## Meta {#meta .widget-title}

-   [Log in](http://www.harryjohn.org/wp-login.php){rel="nofollow"}
-   [Entries RSS](http://www.harryjohn.org/feed/)
-   [Comments RSS](http://www.harryjohn.org/comments/feed/)
-   [WordPress.org](https://wordpress.org/ "Powered by WordPress, state-of-the-art semantic personal publishing platform.")
:::::
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

::: {#site-generator}
© Harry John
:::
