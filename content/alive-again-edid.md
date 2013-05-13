Title: Fixing half-dead monitor with EDID writes
Date: 2013-05-10 17:30
Tags: OSX, EDID, edid-rw, viewsonic, VP2130b
Category: monitor, debugging, fix
Slug: alive-again-edid
Author: Matthew Brett
Summary: My monitor went blank with OSX, I rewrote the EDID and it's OK now.

I have dual monitors attached via DVI to my elderly Mac Pro 1,1 destop.

Both monitors are ViewSonic VP2130b monitors that I found lying around
somewhere. It turned out to be very lucky for me that the monitors were the
same make and model, as you'll see later.

The right monitor of the two is also attached (via a KVM) to an old laptop I
use for hosting our [buildbots](http://nipy.bic.berkeley.edu/builders).  The
laptop runs Ubuntu 12.04.

They were working fine up until a few days ago when I noticed that the screen
went blank when I tried to switch the KVM to the laptop.  I thought nothing
much of it, and connected the laptop to another screen.

Now the laptop had its own screen and the ViewSonic was attached directly to
the Mac.

I noticed two new signs.  First, I saw messages like this on the screen I had
just plugged into the laptop:

    May 10 16:01:39 maqroll kernel: [ 9762.942248] [drm:drm_edid_block_valid] *ERROR* EDID checksum is invalid, remainder is 7

I was in a rush, googled the error in a half-hearted way, and lost interest.

The second thing I noticed was that the monitor that had been attached to the
laptop via the KVM was behaving strangely.

* It was always blank when attached to OSX
* OSX now seemed to believe it was a VGA monitor
* No display settings gave any signal in OSX
* The monitor worked fine when attached to the Ubuntu laptop

I tried all sorts of things, such as doing a factory reset to the monitor. I had to repeat [these
instructions](http://www.flickr.com/photos/14723666@N03/5554138009/in/set-72157626336780968)
a few times before the "FACTORY" OSD came up.  It had no effect, I just mention it for completeness.

It was only at this stage that I began to consider that I had a problem with
the monitor's [EDID](http://en.wikipedia.org/wiki/EDID).  I know that's dumb,
you saw the error I had got before.  I suppose I was tired.  What, you've never
done anything dumb like that?

Then I tried [this
Vodoo](http://www.tablix.org/~avian/blog/archives/2010/06/the_curious_case_of_viewsonic_s_edid)
to reset the EDID, connecting the VGA and DVI in a magic order.  No effect.

I read some pages, very superficially:

* <http://blog.komeil.com/2008/06/fixing-edid-dvi-monitors-no-signal.html>
* <http://www.playtool.com/pages/dvitrouble/dvitrouble.html>
* <http://www.insanelymac.com/forum/topic/208410-fixing-scrambledstretched-or-wrong-resolution-laptop-display-problems/>

I played with [read-edid](http://www.polypux.org/projects/read-edid/), but this
didn't seem to detect a difference in the EDID of my good and my bad monitor.
This later turned out to be because they were only reading the EDID of the LCD
screen of the laptop and there is no way to ask the current tools in
`read-edid` to get the EDID of the attached monitor.

Finally, I stumbled on [edid-rw](https://github.com/bulletmark/edid-rw).

I was now working on a friend's Dell laptop running Ubuntu 12.10 - [thanks
Fernando](http://fperez.org).  I have either the good or bad ViewSonic monitor
plugged into the HDMI out.

I installed the dependencies for `edid-rw`.

By trial and error I found that this:

    sudo ./edid-rw 2

found one display (that turned out to be my laptop LCD) and this:

    sudo ./edid-rw 6

found the ViewSonic (because the output text included `VP2130  SERIES`).  Other
display numbers gave:

    IOError: [Errno 6] No such device or address

I then attached the good ViewSonic, and checked that I got the `VP2130  SERIES`
text from ``sudo ./edid-rw 6`` again.  I then read out the EDID:

    sudo ./edid-rw 6 > good-edid.bin

I unplugged the good monitor, plugged in the bad one, checked it was still
attached to display 6 as above and:

    sudo ./edid-rw -w 6 < good-edid.bin

I read out the EDID again to check it was the same as ``good-edid.bin``; it
was.

Hey presto, plugging into OSX - the monitor is good as new.

Here are the contents of [good-edid.bin](|filename|/downloads/good-edid.bin).
Obviously you would not want to write this to your own monitor as it will
likely do **terrible damage** - but just for reference.

<!--- vim:ft=markdown -->
