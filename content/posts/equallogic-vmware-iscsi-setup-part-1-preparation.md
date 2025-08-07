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

:::::::::::::::::::::::::::::::::: {#container}
::::::::::::::::::::::::::::::::: {#content .clearfix}
::::::::::::::::::::::::::::: {#main .col620 .clearfix role="main"}
:::: post-meta-style
::: post-date-style
[Feb 24,
2013](http://www.harryjohn.org/equallogic-vmware-iscsi-setup-part-1-preparation/){rel="bookmark"}
:::

by [Harry
John](http://www.harryjohn.org/author/harry/ "View all posts by Harry John"){.url
.fn .n rel="author"}

in [EqualLogic VMware iSCSI
Setup](http://www.harryjohn.org/category/equallogic-vmware-iscsi-setup/){rel="tag"}

[[Leave a]{.white}
comment](http://www.harryjohn.org/equallogic-vmware-iscsi-setup-part-1-preparation/#respond)
::::

::::::::::::::::::::::: content-wrap
::::::::::::::::::::: post-wrap
::: entry-header
# EqualLogic VMware iSCSI Setup -- Part 1: Preparation {#equallogic-vmware-iscsi-setup-part-1-preparation .entry-title}
:::

::::::::::::::::::: {.entry-content .post_content}
:::::::::: e-mailit_top_toolbox
::::::::: {.e-mailit_toolbox .square .size32 emailit-url="http://www.harryjohn.org/equallogic-vmware-iscsi-setup-part-1-preparation/" emailit-title="EqualLogic VMware iSCSI Setup – Part 1: Preparation"}
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

[![Dell
VMware](http://www.harryjohn.org/wp-content/uploads/2013/02/dell-vmware1.png){.aligncenter
.wp-image-57 width="286" height="144"
srcset="http://www.harryjohn.org/wp-content/uploads/2013/02/dell-vmware1.png 409w, http://www.harryjohn.org/wp-content/uploads/2013/02/dell-vmware1-300x151.png 300w"
sizes="(max-width: 286px) 100vw, 286px"}](http://www.harryjohn.org/wp-content/uploads/2013/02/dell-vmware1.png)EqualLogic
VMware iSCSI setup best practices have gone through many changes
recently, fortunately Dell and VMware seem to have partnered well and
made some good improvements towards the integration of their products.
Development may well settle down now along with the release of ESXi
5.0/5.1 and Dell's own Multipathing Extension Module (MEM) available to
those with Enterprise vSphere licencing.

There is a lot of documentation -- whitepapers, manuals and blog posts
on the topic of iSCSI setup which can become overwhelming quite
quickly.The setup is actually really simple with the correct up to date
documentation to hand. The entire process of the SAN network
installation can be broken down into the following steps:

1.  SAN fabric design
2.  Switch configuration
3.  Array configuration
4.  Host configuration
5.  Monitoring

[]{#more-25}

## EqualLogic VMware iSCSI Documentation

Firstly I do recommend you visit the [Rapid EqualLogic Configuration
Portal](http://en.community.dell.com/techcenter/storage/w/wiki/3615.rapid-equallogic-configuration-portal-by-sis.aspx "Rapid EqualLogic Configuration Portal by SIS - Dell Storage Community"){onclick="__gaTracker('send', 'event', 'outbound-article', 'http://en.community.dell.com/techcenter/storage/w/wiki/3615.rapid-equallogic-configuration-portal-by-sis.aspx', 'Rapid EqualLogic Configuration Portal');"
target="_blank"} which is also broken down into 5 simple stages similar
to how I have arranged this article. Download each document appropriate
to your hardware and have a read through. EqualLogic have included
sections where you can document your IP address settings, I re-created a
lot of this into an Excel spreadsheet for my documentation and have
published a copy for you fine people to use 🙂 I recommend you download
the document and open it in Excel to make full use of the functions and
formulas.

[SAN Network
Configuration.xlsx](https://docs.google.com/file/d/0B8aXU-ZIYTeuOGs4NTVJUGUwODg/edit?usp=sharing "SAN Network Configuration.xlsx"){onclick="__gaTracker('send', 'event', 'outbound-article', 'https://docs.google.com/file/d/0B8aXU-ZIYTeuOGs4NTVJUGUwODg/edit?usp=sharing', 'SAN Network Configuration.xlsx');"
target="_blank"}

## iSCSI Network Settings

It is good practice to document to as much as possible before you start.
This helps you during the planning process to know that you have
everything clearly set out, whilst implementing you know you are
entering the correct settings and if anything goes wrong you have a
reference of what should be in place. In short the settings you need to
prepare are as follows:

-   Host iSCSI vmkernel IP addresses
-   Host vmkernel NIC names
-   EqualLogic group IP address
-   EqualLogic interface IP addresses

In the next step of our EqualLogic VMware iSCSI setup we will look at
the hardware and network fabric keeping a few important design
considerations in mind.

EqualLogic VMware iSCSI Setup -- Part 1: Preparation\
[EqualLogic VMware iSCSI Setup -- Part 2: Network
Configuration](http://www.harryjohn.org/equallogic-vmware-iscsi-setup-part-2-network/ "EqualLogic VMware iSCSI Setup – Part 2: Network Configuration"){onclick="__gaTracker('send', 'event', 'outbound-article', 'http://www.harryjohn.org/equallogic-vmware-iscsi-setup-part-2-network/', 'EqualLogic VMware iSCSI Setup – Part 2: Network Configuration');"}

:::::::::: e-mailit_bottom_toolbox
::::::::: {.e-mailit_toolbox .square .size32 emailit-url="http://www.harryjohn.org/equallogic-vmware-iscsi-setup-part-1-preparation/" emailit-title="EqualLogic VMware iSCSI Setup – Part 1: Preparation"}
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
:::::::::::::::::::

This entry was posted in [EqualLogic VMware iSCSI
Setup](http://www.harryjohn.org/category/equallogic-vmware-iscsi-setup/){rel="tag"}
and tagged
[EqualLogic](http://www.harryjohn.org/tag/equallogic/){rel="tag"},
[iSCSI](http://www.harryjohn.org/tag/iscsi/){rel="tag"},
[VMware](http://www.harryjohn.org/tag/vmware/){rel="tag"}. Bookmark the
[permalink](http://www.harryjohn.org/equallogic-vmware-iscsi-setup-part-1-preparation/ "Permalink to EqualLogic VMware iSCSI Setup – Part 1: Preparation"){rel="bookmark"}.
:::::::::::::::::::::

::: {.clearfix .shadowfix}
:::
:::::::::::::::::::::::

# Post navigation {#post-navigation .assistive-text .section-heading}

::: nav-next
[[Next
→]{.meta-nav}](http://www.harryjohn.org/veeam-backup-job-failed-cannot-create-a-shadow-copy-of-the-volumes-containing-writers-data/){rel="next"}
:::

:::: {#comments}
::: {#respond .comment-respond}
### Leave a Reply [[Cancel reply](/equallogic-vmware-iscsi-setup-part-1-preparation/#respond){#cancel-comment-reply-link rel="nofollow" style="display:none;"}]{.small} {#reply-title .comment-reply-title}

[Your email address will not be published.]{#email-notes} Required
fields are marked [\*]{.required}

Comment

Name [\*]{.required}

Email [\*]{.required}

Website
:::
::::
:::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::

::: {#site-generator}
© Harry John
:::
