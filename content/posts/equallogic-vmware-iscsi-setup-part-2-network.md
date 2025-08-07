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

:::::::::::::::::::::::::::::::::::: {#container}
::::::::::::::::::::::::::::::::::: {#content .clearfix}
::::::::::::::::::::::::::::::: {#main .col620 .clearfix role="main"}
:::: post-meta-style
::: post-date-style
[Feb 26,
2013](http://www.harryjohn.org/equallogic-vmware-iscsi-setup-part-2-network/){rel="bookmark"}
:::

by [Harry
John](http://www.harryjohn.org/author/harry/ "View all posts by Harry John"){.url
.fn .n rel="author"}

in [EqualLogic VMware iSCSI
Setup](http://www.harryjohn.org/category/equallogic-vmware-iscsi-setup/){rel="tag"}

[[Comments]{.white}
(3)](http://www.harryjohn.org/equallogic-vmware-iscsi-setup-part-2-network/#comments)
::::

:::::::::::::::::::::::: content-wrap
:::::::::::::::::::::: post-wrap
::: entry-header
# EqualLogic VMware iSCSI Setup -- Part 2: Network Configuration {#equallogic-vmware-iscsi-setup-part-2-network-configuration .entry-title}
:::

:::::::::::::::::::: {.entry-content .post_content}
:::::::::: e-mailit_top_toolbox
::::::::: {.e-mailit_toolbox .square .size32 emailit-url="http://www.harryjohn.org/equallogic-vmware-iscsi-setup-part-2-network/" emailit-title="EqualLogic VMware iSCSI Setup – Part 2: Network Configuration"}
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

[![SAN LAN
Network](http://www.harryjohn.org/wp-content/uploads/2013/02/SAN-LAN-Network-e1361849167569.png){.size-full
.wp-image-87 .alignright width="220"
height="130"}](http://www.harryjohn.org/wp-content/uploads/2013/02/SAN-LAN-Network-e1361849167569.png)

The EqualLogic iSCSI network consists of the switches, EqualLogic arrays
and ESX hosts, all of which need configuring properly to make the most
of each others load balancing, performance and availability features. In
terms of the network fabric, there are a few things we need to consider:

1.  High availability -- multiple links and switches to provide
    redundancy
2.  Performance -- elimination of bottlenecks
3.  Management -- monitoring and management network

This guide can apply to a range of hardware as long as we are using
EqualLogic arrays and VMware ESX/ESXi hosts. The switches can be Dell or
Cisco for example, as long as they are on the Dell Approved Hardware
list. The hardware I used when making this guide is as follows:

-   [2 x PowerConnect 7048]{style="line-height: 13px;"}
-   6 x PowerEdge R710
-   2 x EqualLogic PS6000

Here is some brief detail about my hardware specifications, and how I
put the key design considerations mentioned earlier into practice when
configuring each component.[]{#more-29}

## EqualLogic PS6000 array members

*4 GbE ports/ controller*\
*2 controllers/member (active passive)*

I will mention management first as the PS6000 does not have a dedicated
management port. I *could* dedicate one of the four Ethernet ports to
management only but this would leave me with three links for iSCSI --
two to switch A but **only one link to the other switch B**. In the case
of a switch failure, we could have **33% available iSCSI bandwidth
compared to normal conditions**.

For this reason I opted for all four ethernet ports to serve both iSCSI
and management for the members, I will configure a VM with SAN HQ on the
iSCSI network for monitoring and email reporting.

The following diagram shows the connections as advised by Dell between
host, switches and array, click the image to enlarge. Note: this diagram
is for a PS6100.

::: {#attachment_83 .wp-caption .aligncenter style="width: 288px"}
[![iSCSI Connection
Diagram](http://www.harryjohn.org/wp-content/uploads/2013/02/iSCSI-Connection-Diagram-278x300.png){.size-medium
.wp-image-83 width="278" height="300"
srcset="http://www.harryjohn.org/wp-content/uploads/2013/02/iSCSI-Connection-Diagram-278x300.png 278w, http://www.harryjohn.org/wp-content/uploads/2013/02/iSCSI-Connection-Diagram-600x646.png 600w, http://www.harryjohn.org/wp-content/uploads/2013/02/iSCSI-Connection-Diagram.png 697w"
sizes="(max-width: 278px) 100vw, 278px"}](http://www.harryjohn.org/wp-content/uploads/2013/02/iSCSI-Connection-Diagram.png)

iSCSI Connection Diagram
:::

## PowerConnect 7048 iSCSI switches

*Stacking modules*

Two switches in a stack will keep the network alive in the case of a
switch PSU or complete failure -- you could also add redundant PSU to
the switches for extra protection. With the stacking modules we have a
fast Inter-Switch Link (ISL) -- whether you LAG or stack you need to
make sure this link is fast as iSCSI traffic can and will traverse the
ISL. I will mention the pros and cons of LAG vs stack in the switch
configuration part of this guide.

The 7048 switches have a high port buffer memory and are approved
EqualLogic iSCSI switches, flow control and all the other iSCSI goodies
are available. These switches also have an Out-of-Band management port
each and these will be connected to our LAN network to give us
management access to the switches.

## VMware ESXi 5.1 hosts

*Dell PowerEdge R710*\
*2 GbE NICs for iSCSI data*

[Two NICs per host is a **requirement** for multipathing to work
properly and allow the hosts and EqualLogics to load balance. You can
add more NICs in pairs if your network utilisation is
high.]{style="font-size: 1em;"}

Now we have our EqualLogic VMware iSCSI network designed to best
practice we can move on to look at the PowerConnect switch
configuration.

[EqualLogic VMware iSCSI Setup -- Part 1:
Preparation](http://www.harryjohn.org/equallogic-vmware-iscsi-setup-part-1-preparation/ "EqualLogic VMware iSCSI Setup – Part 1: Preparation"){onclick="__gaTracker('send', 'event', 'outbound-article', 'http://www.harryjohn.org/equallogic-vmware-iscsi-setup-part-1-preparation/', 'EqualLogic VMware iSCSI Setup – Part 1: Preparation');"}\
EqualLogic VMware iSCSI Setup -- Part 2: Network Configuration

:::::::::: e-mailit_bottom_toolbox
::::::::: {.e-mailit_toolbox .square .size32 emailit-url="http://www.harryjohn.org/equallogic-vmware-iscsi-setup-part-2-network/" emailit-title="EqualLogic VMware iSCSI Setup – Part 2: Network Configuration"}
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
::::::::::::::::::::

This entry was posted in [EqualLogic VMware iSCSI
Setup](http://www.harryjohn.org/category/equallogic-vmware-iscsi-setup/){rel="tag"}.
Bookmark the
[permalink](http://www.harryjohn.org/equallogic-vmware-iscsi-setup-part-2-network/ "Permalink to EqualLogic VMware iSCSI Setup – Part 2: Network Configuration"){rel="bookmark"}.
::::::::::::::::::::::

::: {.clearfix .shadowfix}
:::
::::::::::::::::::::::::

# Post navigation {#post-navigation .assistive-text .section-heading}

::: nav-previous
[[←
Previous]{.meta-nav}](http://www.harryjohn.org/veeam-backup-job-failed-cannot-create-a-shadow-copy-of-the-volumes-containing-writers-data/){rel="prev"}
:::

::: nav-next
[[Next
→]{.meta-nav}](http://www.harryjohn.org/equallogic-vmware-iscsi-setup-part-3-powerconnect-configuration/){rel="next"}
:::

:::: {#comments}
## 3 thoughts on "EqualLogic VMware iSCSI Setup -- Part 2: Network Configuration" {#comments-title}

1.  ::::::: {#li-comment-163}
    ::: {.comment-author .vcard}
    ![](http://1.gravatar.com/avatar/4a0137e8ee91903e2b40626b8521390a?s=40&d=mm&r=g){.avatar
    .avatar-40 .photo
    srcset="http://1.gravatar.com/avatar/4a0137e8ee91903e2b40626b8521390a?s=80&d=mm&r=g 2x"
    height="40" width="40"} Douglas Arnett
    :::

    ::: {.comment-meta .commentmetadata}
    [July 3, 2013 at 8:15
    pm](http://www.harryjohn.org/equallogic-vmware-iscsi-setup-part-2-network/#comment-163)
    :::

    ::: comment-content
    The first two parts are great! Where is Part 3???
    :::

    ::: reply
    [Reply](http://www.harryjohn.org/equallogic-vmware-iscsi-setup-part-2-network/?replytocom=163#respond){.comment-reply-link
    rel="nofollow"
    onclick="return addComment.moveForm( \"comment-163\", \"163\", \"respond\", \"29\" )"
    aria-label="Reply to Douglas Arnett"}
    :::
    :::::::

2.  ::::::: {#li-comment-166}
    ::: {.comment-author .vcard}
    ![](http://1.gravatar.com/avatar/a2d9998871d034b1ebea0c971d391ec3?s=40&d=mm&r=g){.avatar
    .avatar-40 .photo
    srcset="http://1.gravatar.com/avatar/a2d9998871d034b1ebea0c971d391ec3?s=80&d=mm&r=g 2x"
    height="40" width="40"} Zack
    :::

    ::: {.comment-meta .commentmetadata}
    [July 8, 2013 at 2:22
    am](http://www.harryjohn.org/equallogic-vmware-iscsi-setup-part-2-network/#comment-166)
    :::

    ::: comment-content
    Any update on this? I'm curious about the pros/cons of LAG vs Stack.
    :::

    ::: reply
    [Reply](http://www.harryjohn.org/equallogic-vmware-iscsi-setup-part-2-network/?replytocom=166#respond){.comment-reply-link
    rel="nofollow"
    onclick="return addComment.moveForm( \"comment-166\", \"166\", \"respond\", \"29\" )"
    aria-label="Reply to Zack"}
    :::

    -   ::::::: {#li-comment-266}
        ::: {.comment-author .vcard}
        ![](http://1.gravatar.com/avatar/71034cc71c8497e2a220996150f5ff1d?s=40&d=mm&r=g){.avatar
        .avatar-40 .photo
        srcset="http://1.gravatar.com/avatar/71034cc71c8497e2a220996150f5ff1d?s=80&d=mm&r=g 2x"
        height="40" width="40"} Harry John
        :::

        ::: {.comment-meta .commentmetadata}
        [September 4, 2013 at 12:05
        am](http://www.harryjohn.org/equallogic-vmware-iscsi-setup-part-2-network/#comment-266)
        :::

        ::: comment-content
        Hi Zack & Douglas,

        Apologies for not updating this article, at the time of writing
        parts 1 and 2 we were using PowerConnect 5548 switches -- the
        reason for the change to a set of PowerConnect 7048 switches is
        due to a recommendation from Dell that the 5548 switches are no
        longer approved for iSCSI use with EqualLogic Arrays.

        I wanted to install these new switches and ensure my
        configuration is sound in a live environment before writing
        part 3. I can now confirm that this upgrade has proved highly
        beneficial in that we are seeing no longer seeing any iSCSI
        disconnects and the TCP retransmit rate has improved
        significantly (now almost non-existent).

        I will publish part 3 shortly and will include the pros/cons of
        LAG vs stack as promised!

        Harry
        :::

        ::: reply
        [Reply](http://www.harryjohn.org/equallogic-vmware-iscsi-setup-part-2-network/?replytocom=266#respond){.comment-reply-link
        rel="nofollow"
        onclick="return addComment.moveForm( \"comment-266\", \"266\", \"respond\", \"29\" )"
        aria-label="Reply to Harry John"}
        :::
        :::::::
    :::::::

::: {#respond .comment-respond}
### Leave a Reply [[Cancel reply](/equallogic-vmware-iscsi-setup-part-2-network/#respond){#cancel-comment-reply-link rel="nofollow" style="display:none;"}]{.small} {#reply-title .comment-reply-title}

[Your email address will not be published.]{#email-notes} Required
fields are marked [\*]{.required}

Comment

Name [\*]{.required}

Email [\*]{.required}

Website
:::
::::
:::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::

::: {#site-generator}
© Harry John
:::
