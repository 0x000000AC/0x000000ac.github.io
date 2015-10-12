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


    #!/bin/sh

    # adblocker.sh - by Todd Stein (toddbstein@gmail.com), Saturday, October 25, 2014
    # for use on routers running OpenWRT firmware
    # updated Monday, September 7, 2015

    # Periodically download lists of known ad and malware servers, and prevents traffic from being sent to them.
    # This is a complete rewrite of a script originally written by teffalump (https://gist.github.com/teffalump/7227752).


    HOST_LISTS="
	    http://adaway.org/hosts.txt
	    http://www.malwaredomainlist.com/hostslist/hosts.txt
	    http://www.mvps.org/winhelp2002/hosts.txt
	    http://pgl.yoyo.org/adservers/serverlist.php?hostformat=hosts&showintro=0&startdate%5Bday%5D=&startdate%5Bmonth%5D=&star
    "

    BLOCKLIST=/tmp/adblocker_hostlist
    BLACKLIST=/etc/adblocker_blacklist
    WHITELIST=/etc/adblocker_whitelist
    LOCKFILE=/tmp/adblocker.lock


    # ensure this is the only instance running
    if ! ln -s $$ "$LOCKFILE" 2>/dev/null; then
	    # if the old instance is still running, exit
	    former_pid=$(readlink "$LOCKFILE")
	    if [ -e "/proc/$former_pid" ]; then
		    exit
	    else
		    # otherwise, update the symlink
		    ln -sf $$ "$LOCKFILE"
	    fi
    fi


    # get script's absolute path and quote spaces for safety
    cd "${0%/*}"
    SCRIPT_NAME="$PWD/${0##*/}"
    SCRIPT_NAME="${SCRIPT_NAME// /' '}"
    cd "$OLDPWD"


    # await internet connectivity before proceeding (in case rc.local executes this script before connectivity is achieved)
    until ping -c1 -w3 google.com || ping -c1 -w3 yahoo.com; do
	    sleep 5
    done &>/dev/null


    # grab list of bad domains from the internet
    IP_REGEX='([0-9]{1,3}\.){3}[0-9]{1,3}'
    hosts=$(wget -qO- $HOST_LISTS | awk "/^$IP_REGEX\W/"'{ print "0.0.0.0",$2 }' | sort -uk2)


    # if the download succeeded, recreate the blocklist
    if [ -n "$hosts" ]; then
	    # add downloaded domains to a fresh block list
	    printf "%s\n" "$hosts" >"$BLOCKLIST"
    fi


    # add blacklisted domains if any have been specified, ensuring no duplicates are added
    if [ -s "$BLACKLIST" ]; then
	    # create a pipe-delimited list of all non-commented words in blacklist and remove them from the block list
	    black_listed_regex='\W('"$(grep -o '^[^#]\+' "$BLACKLIST" | xargs | tr ' ' '|')"')$'
	    sed -ri "/${black_listed_regex//./\.}/d" "$BLOCKLIST"

	    # add blacklisted domains to block list	
	    awk '/^[^#]/ { print "0.0.0.0",$1 }' "$BLACKLIST" >>"$BLOCKLIST"
    fi


    # remove any private net IP addresses (just in case)
    # this variable contains a regex which will be used to prevent the blocking of hosts on 192.168.0.0 and 10.0.0.0 networks
    PROTECTED_RANGES='\W(192\.168(\.[0-9]{1,3}){2}|10(\.[0-9]{1,3}){3})$'
    sed -ri "/$PROTECTED_RANGES/d" "$BLOCKLIST"


    # remove any whitelisted domains from the block list
    if [ -s "$WHITELIST" ]; then
	    # create a pipe-delimited list of all non-commented words in whitelist and remove them from the block list
	    white_listed_regex='\W('"$(grep -Eo '^[^#]+' "$WHITELIST" | xargs | tr ' ' '|')"')$'
	    sed -ri "/${white_listed_regex//./\.}/d" "$BLOCKLIST"
    fi


    # add IPv6 blocking
    sed -ri 's/([^ ]+)$/\1\n::      \1/' "$BLOCKLIST"


    # add block list to dnsmasq config if it's not already there
    if ! uci -q get dhcp.@dnsmasq[0].addnhosts | grep -q "$BLOCKLIST"; then
	    uci add_list dhcp.@dnsmasq[0].addnhosts="$BLOCKLIST" && uci commit
    fi


    # restart dnsmasq service
    /etc/init.d/dnsmasq restart


    # carefully add script to /etc/rc.local if it's not already there
    if ! grep -Fq "$SCRIPT_NAME" /etc/rc.local; then
	    # using awk and cat ensures that no symlinks (if any exist) are clobbered by BusyBox's feature-poor sed.
	    awk -v command="$SCRIPT_NAME" '
		    ! /^exit( 0)?$/ {
			    print $0
		    }
		    /^exit( 0)?$/ {
			    print command "\n" $0
			    entry_added=1
		    }
		    END {
			    if (entry_added != 1) {
				    print command
			    }
		    }' /etc/rc.local >/tmp/rc.local.new
	    cat /tmp/rc.local.new >/etc/rc.local
	    rm -f /tmp/rc.local.new
    fi


     # add script to root's crontab if it's not already there
    if ! grep -Fq "$SCRIPT_NAME" /etc/crontabs/root 2>/dev/null; then
	    # adds 30 minutes of jitter to prevent undue load on the webservers hosting the lists we pull each week
	    # unfortunately, there's no $RANDOM in this shell, so:
	    DELAY=$(head /dev/urandom | wc -c | /usr/bin/awk "{ print \$0 % 30 }")
	    cat >>/etc/crontabs/root <<-:EOF:
		    # Download updated ad and malware server lists every Tuesday at 3:$(printf "%02d" "$DELAY") AM
		    $DELAY 3 * * 2 $SCRIPT_NAME
	    :EOF:
    fi


    # clean up
    rm -f "$LOCKFILE"


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