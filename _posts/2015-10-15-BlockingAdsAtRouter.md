---
layout: post
title: Blocking Ads Without Browser Plugins
---

I love Mad Men.   In my mind, there is no better illustration of the sterotypical
American man burdened by ambition than Don Draper.   For the uninitiated, Don works
at an advertising agency in New York at a bygone time of all-formal-all-the-time
attire and social upheaval.  It's a great show on many levels.
 
In any event, I'm not here to talk about the show as much as I am about the current state
of advertisement on the Internet. Sadly, desperately sadly, ads here aren't as easy to
ignore as their physical counterparts.   They track, they redirect,
the scape and sell, and they annoy me to no end.
 
You can certainly help the situation by using Adblock/Adblock Plus.  However, some sites
cleverly counter this add-in and disable content until you disable it.   I get that.
Some sites sole source of information is through ads.   I just don't particularly like them.
I go out of my way to avoid ads.   I see the characteristic yellow square on an Google and
I purposefully go out of my way to scroll and click subsequent pages to get past the ads
for the content I want.

So I set out on a quest.   A quest for ad-free browsing.   Boy, I certainly wasn't 
disappointed.

I came across a bash script written by a guy named Todd Stein which periodically reaches
out to update update a blacklist.   

...


This got me thinking.  I've been needing a new wireless
router for a while now.   When I moved here, for some reason I decided to go the the ISP
all-in-one modem/router.   Unfortunately, the radio on it has been flaky and prompted me
in some parts of the house to use some [Ethernet over powerline adapters](http://www.amazon.com/TP-LINK-TL-PA4010-Powerline-Adapter-500Mbps/dp/B00CUD1M66)
on devices that I really don't want to deal with spotty Internet.   I wanted to go cheap
and dirty and also wanted a device that I could flash with [dd-wrt](http://www.dd-wrt.com/site/index),
[openwrt](https://openwrt.org/), or [tomato](http://www.polarcloud.com/tomato).  I've used
the classic Linksys/dd-wrt combo in the past with success, but wanted to look at some other
options.   I found a super inexpensive [TP-Link TL-WR841N](http://www.amazon.com/TP-LINK-TL-WR841N-Wireless-Router-300Mbps/dp/B001FWYGJS/ref=sr_1_1?s=pc&ie=UTF8&qid=1444622938&sr=1-1&keywords=tp-link+841n)
that seemed to be a good candidate -- meaning I wouldn't mind if I bricked it.

I got the router in a couple of days and decided to go with OpenWrt.  This was the first 
time I had flashed with OpenWrt, so I wasn't sure if I was going to have to build a serial
cable, tftp, or something like JTAG.

It was wildly more simple than that.   Details follow.

1.Logging into the router to check the firmware version so that I can ensure I select
the correct OpenWrt version.
 
![TL-WR841N](/images/1_Version9.jpg)
 
2.Version 9 of the stock firmware works with 15.05

![OpenWrt Version](/images/2_firwareVersion.jpg)

3.Navigating to firmware upgrade section of the 841.

![Firmware update](/images/4_FirmwareUpdate.jpg)

4.It successfully took!

![Upgrade in progress](/images/5_upgrading.jpg)

5.Logging in to OpenWrt after reboot.

![Logging into Wrt](/images/6_loginWRT.jpg)

6.Verifying the OpenWrt version.

![New firmware](/images/7_SystemAdministration.jpg)

7.Now that the new firmware is running, it's time to SSH in to the device so that
so that we can pull down the script and get it running.

![ssh in](/images/9_sshingIn.jpg)

8.Pulling the script

![getting script](/images/10_getScript.jpg)

9.MD5 the script against the site MD5 to verify authenticity.

![MD5 the script](/images/11_md5.jpg)

10.Setting the script to be an executable and running it.

![running the script](/images/12_execute.jpg)

11.Confirming the script is running.

![confirming the script is running](/images/13_confirm.jpg)

12.If needed, you can always uninstall the script as well

![uninstalling script](/images/14_toUninstall.jpg)
