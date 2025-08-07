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

::::::::::::::::::::::::::::::::::::: {#container}
:::::::::::::::::::::::::::::::::::: {#content .clearfix}
:::::::::::::::::::::::::::::::: {#main .col620 .clearfix role="main"}
:::: post-meta-style
::: post-date-style
[Feb 26,
2013](http://www.harryjohn.org/veeam-backup-job-failed-cannot-create-a-shadow-copy-of-the-volumes-containing-writers-data/){rel="bookmark"}
:::

by [Harry
John](http://www.harryjohn.org/author/harry/ "View all posts by Harry John"){.url
.fn .n rel="author"}

in
[Uncategorized](http://www.harryjohn.org/category/uncategorized/){rel="tag"}

[[Comments]{.white}
(2)](http://www.harryjohn.org/veeam-backup-job-failed-cannot-create-a-shadow-copy-of-the-volumes-containing-writers-data/#comments)
::::

::::::::::::::::::::::::: content-wrap
::::::::::::::::::::::: post-wrap
::: entry-header
# Veeam Backup job failed. Cannot create a shadow copy of the volumes containing writer's data. {#veeam-backup-job-failed.-cannot-create-a-shadow-copy-of-the-volumes-containing-writers-data. .entry-title}
:::

::::::::::::::::::::: {.entry-content .post_content}
:::::::::: e-mailit_top_toolbox
::::::::: {.e-mailit_toolbox .square .size32 emailit-url="http://www.harryjohn.org/veeam-backup-job-failed-cannot-create-a-shadow-copy-of-the-volumes-containing-writers-data/" emailit-title="Veeam Backup job failed. Cannot create a shadow copy of the volumes containing writer’s data."}
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

::: {#attachment_123 .wp-caption .alignright style="width: 166px"}
[![Backup job
failed](http://www.harryjohn.org/wp-content/uploads/2013/02/Backup-job-failed.png){.size-full
.wp-image-123 width="156"
height="86"}](http://www.harryjohn.org/wp-content/uploads/2013/02/Backup-job-failed.png)

Backup job failed
:::

Occasionally one of our Veeam backup jobs will fail and most of the time
it is due to shadow copy or VSS writer issue which a simple reboot will
resolve, this solution has become our default answer to a failed backup
with an error mentioning the words "shadow copy" or "VSS". One of our
SQL servers had a failed backup last night and gave us the error "Cannot
create a shadow copy of the volumes containing writer's data", here is
the full error message:

> Failed to prepare guest for hot backup. Error: VSSControl: -2147467259
> Backup job failed. Cannot create a shadow copy of the volumes
> containing writer's data. VSS asynchronous operation is not completed.
> Operation: \[Shadow copies commit\]. Code: \[0x8004231f\].\
> Error: VSSControl: -2147467259 Backup job failed. Cannot create a
> shadow copy of the volumes containing writer's data. VSS asynchronous
> operation is not completed. Operation: \[Shadow copies commit\]. Code:
> \[0x8004231f\].

[]{#more-115}This time a reboot did not fix the problem so checking the
event viewer reveals a clear message that I am running out of space on
C:

> When preparing a new volume shadow copy for volume C:, the shadow copy
> storage on volume C: did not have sufficiently large contiguous
> blocks. Consider deleting unnecessary files on the shadow copy storage
> volume or use a different shadow copy storage volume.

Why did my monitoring not pick this up you may be asking! Well the C
drive is a 12GB volume with 2.5GB free -- 20.8%, just above our alert
threshold by 0.8%, apparently 2.5GB free was not enough to run the
shadow copies on this server!

Further investigation showed the default user profile's temporary
internet files had inflated to 400MB! After resolving this issue by
following this guide here on how [to get rid of the huge default user
temporary internet
files](http://www.paessler.com/blog/2009/04/17/prtg-7/how-to-get-rid-of-huge-default-userlocal-settingstemporary-internet-filescontentie5-folders "how to get rid of the huge default user temporary internet files"){onclick="__gaTracker('send', 'event', 'outbound-article', 'http://www.paessler.com/blog/2009/04/17/prtg-7/how-to-get-rid-of-huge-default-userlocal-settingstemporary-internet-filescontentie5-folders', 'to get rid of the huge default user temporary internet files');"
target="_blank"}.

Backup still failed! Time to check the status of the VSS writers by
using the command:

`vssadmin list writers`

::: {#attachment_120 .wp-caption .alignnone style="width: 677px"}
[![List VSSAdmin
Writers](http://www.harryjohn.org/wp-content/uploads/2013/02/List-VSSAdmin-Writers.png){.size-full
.wp-image-120 width="667" height="328"
srcset="http://www.harryjohn.org/wp-content/uploads/2013/02/List-VSSAdmin-Writers.png 667w, http://www.harryjohn.org/wp-content/uploads/2013/02/List-VSSAdmin-Writers-300x147.png 300w, http://www.harryjohn.org/wp-content/uploads/2013/02/List-VSSAdmin-Writers-600x295.png 600w"
sizes="(max-width: 667px) 100vw, 667px"}](http://www.harryjohn.org/wp-content/uploads/2013/02/List-VSSAdmin-Writers.png)

List VSSAdmin Writers
:::

For each failed writer you need to restart the appropriate service. The
following is a list of the failed writers I had and the services I
restarted to bring them back to "stable"

-   SqlServerWriter -- SQL Server VSS Writer
-   IIS Metabase Writer -- IIS Admin Service
-   WMI Writer -- Windows Management Instrumentation

As soon as I had cleared up some free space on C, restarted these
services and all VSS writers showed as stable the backup succeeded. Two
things to learn here; we should also have a threshold on 2.5GB as well
as 20% free space for shadow copies and that "Cannot create a shadow
copy of the volumes containing writer's data" means we need to look
beyond VSS/backups and more towards the storage that shadow copies are
enabled for (check event logs).

:::::::::: e-mailit_bottom_toolbox
::::::::: {.e-mailit_toolbox .square .size32 emailit-url="http://www.harryjohn.org/veeam-backup-job-failed-cannot-create-a-shadow-copy-of-the-volumes-containing-writers-data/" emailit-title="Veeam Backup job failed. Cannot create a shadow copy of the volumes containing writer’s data."}
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
:::::::::::::::::::::

This entry was posted in
[Uncategorized](http://www.harryjohn.org/category/uncategorized/){rel="tag"}
and tagged [Veeam](http://www.harryjohn.org/tag/veeam/){rel="tag"},
[VMware](http://www.harryjohn.org/tag/vmware/){rel="tag"}. Bookmark the
[permalink](http://www.harryjohn.org/veeam-backup-job-failed-cannot-create-a-shadow-copy-of-the-volumes-containing-writers-data/ "Permalink to Veeam Backup job failed. Cannot create a shadow copy of the volumes containing writer’s data."){rel="bookmark"}.
:::::::::::::::::::::::

::: {.clearfix .shadowfix}
:::
:::::::::::::::::::::::::

# Post navigation {#post-navigation .assistive-text .section-heading}

::: nav-previous
[[←
Previous]{.meta-nav}](http://www.harryjohn.org/equallogic-vmware-iscsi-setup-part-1-preparation/){rel="prev"}
:::

::: nav-next
[[Next
→]{.meta-nav}](http://www.harryjohn.org/equallogic-vmware-iscsi-setup-part-2-network/){rel="next"}
:::

:::: {#comments}
## 2 thoughts on "Veeam Backup job failed. Cannot create a shadow copy of the volumes containing writer's data." {#comments-title}

1.  ::::::: {#li-comment-154}
    ::: {.comment-author .vcard}
    ![](http://0.gravatar.com/avatar/0fd04cf6056520604994c6c7093f94ad?s=40&d=mm&r=g){.avatar
    .avatar-40 .photo
    srcset="http://0.gravatar.com/avatar/0fd04cf6056520604994c6c7093f94ad?s=80&d=mm&r=g 2x"
    height="40" width="40"} Michael
    :::

    ::: {.comment-meta .commentmetadata}
    [June 20, 2013 at 1:36
    pm](http://www.harryjohn.org/veeam-backup-job-failed-cannot-create-a-shadow-copy-of-the-volumes-containing-writers-data/#comment-154)
    :::

    ::: comment-content
    Thank you for this Artikel.\
    Has helped.\
    Michael
    :::

    ::: reply
    [Reply](http://www.harryjohn.org/veeam-backup-job-failed-cannot-create-a-shadow-copy-of-the-volumes-containing-writers-data/?replytocom=154#respond){.comment-reply-link
    rel="nofollow"
    onclick="return addComment.moveForm( \"comment-154\", \"154\", \"respond\", \"115\" )"
    aria-label="Reply to Michael"}
    :::
    :::::::

2.  ::::::: {#li-comment-267}
    ::: {.comment-author .vcard}
    ![](http://0.gravatar.com/avatar/639ddcd3fc002729aaca88ab559eb383?s=40&d=mm&r=g){.avatar
    .avatar-40 .photo
    srcset="http://0.gravatar.com/avatar/639ddcd3fc002729aaca88ab559eb383?s=80&d=mm&r=g 2x"
    height="40" width="40"} Susan
    :::

    ::: {.comment-meta .commentmetadata}
    [September 4, 2013 at 8:31
    pm](http://www.harryjohn.org/veeam-backup-job-failed-cannot-create-a-shadow-copy-of-the-volumes-containing-writers-data/#comment-267)
    :::

    ::: comment-content
    Thx, this helped me look to the correct area as well. In my case, I
    deleted old Symantec virus defnitions to free up space on the C:
    drive, then was able to back up the failing VM successfully.
    :::

    ::: reply
    [Reply](http://www.harryjohn.org/veeam-backup-job-failed-cannot-create-a-shadow-copy-of-the-volumes-containing-writers-data/?replytocom=267#respond){.comment-reply-link
    rel="nofollow"
    onclick="return addComment.moveForm( \"comment-267\", \"267\", \"respond\", \"115\" )"
    aria-label="Reply to Susan"}
    :::
    :::::::

::: {#respond .comment-respond}
### Leave a Reply [[Cancel reply](/veeam-backup-job-failed-cannot-create-a-shadow-copy-of-the-volumes-containing-writers-data/#respond){#cancel-comment-reply-link rel="nofollow" style="display:none;"}]{.small} {#reply-title .comment-reply-title}

[Your email address will not be published.]{#email-notes} Required
fields are marked [\*]{.required}

Comment

Name [\*]{.required}

Email [\*]{.required}

Website
:::
::::
::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::

::: {#site-generator}
© Harry John
:::
