
https://forums.macrumors.com/threads/mp3-1-others-sse-4-2-emulation-to-enable-amd-metal-driver.2206682/

              MouSSE (Limited SSE4.2 Emulator) Public Release
                  Current version: 0.38 (19 October 2019)

This software has been tested on various hardware and software configurations,
and it seems to be stable enough to release from beta.  Please provide feedback,
positive or negative, success or failure, via the MacRumors forums.

*** IMPORTANT: USE AT YOUR OWN RISK!

This software is provided as-is, without any warranty whatsoever.  There is no
reason it should cause any problems, but if things go wrong, they're your
responsibility.  It has been tested and benchmarked on various systems and
various versions of MacOS, but there are no guarantees that it will work on any
given system or version of MacOS.  The author is not responsible for any losses
of time, money, hair, memory, peace of mind, car keys, or television remote
controls related in any way to the use of this software.
(And now, after that ringing endorsement...)

*** What is it?

Newer AMD Mac video drivers use some SSE 4.2 instructions.  Older CPUs (Penryn,
Harpertown, and earlier) don't support those instructions.  Older Mac Pro
systems (such as the Mac Pro 3,1) use those older CPUs - therefore, the new AMD
drivers won't work in those systems.  MouSSE is a limited SSE4.2 emulator that
allows those old CPUs to use the newer AMD drivers.  While its primary focus has
always been making the AMD drivers work, it also appears to allow World of
Warcraft to run on a Mac Pro 3,1 regardless of whether or not AMD drivers are
in use.

In theory, depending upon what you're trying to accomplish, this emulator could
prove useful on any Mac system with a Penryn/Harpertown/Wolfdale CPU.  These
include:
       Mac Pro 3,1 (Early 2008)
       Xserve 2,1 (Early 2008)
       MacBook Pro 4,1 (Early 2008)
       MacBook 4,1 (Early 2008)
       MacBook 7,1 (Mid-2010)
       MacBook Air 2,1 (October 2008)
       MacBook Air 3,1 (October 2010) (11")
       MacBook Air 3,2 (October 2010) (13")
       iMac 8,1 (April 2008)
       iMac 9,1 (March 2009)
       iMac 10,1 (October 2009) (with Core2 Duo, not i5)
       Mac Mini 3,1 (March 2009, October 2009)
       Mac Mini 4,1 (Mid 2010) (aka Mac Mini Server)
Not all of these have been tested;  your mileage may vary.  In addition, CPUs
even older than Penryn could utilize this emulator;  however, since they lack
SSE4.1, adding only limited SSE4.2 would probably be of little or no benefit.

I personally have tested this on MacOS 10.13-10.15 (High Sierra, Mojave, and the
release version of Catalina).  Others report that it loads on 10.11-10.12 (El
Capitan and Sierra), although the AMD drivers on those do not seem to require
emulation.

*** What does it do?

MouSSE traps illegal instructions in both privileged (kernel) mode and
unprivileged (user) mode, and emulates the POPCNT and PCMPGTQ instructions.  As
of this writing, those two instructions are the only problematic ones used in
the AMD drivers.  (And, apparently, the only two currently used in World of
Warcraft.)

MouSSE is fully reentrant, and automatically runs on all CPUs/cores/threads.

*** What does it NOT do?

At this time, MouSSE does not implement any SSE4.2 instructions other than
POPCNT and PCMPGTQ.  If there is sufficient demand, a future version may include
support for additional SSE4.2 (or other) instructions.  If MouSSE encounters an
unsupported instruction, it passes control to MacOS, and you will see the same
error handling you'd see if MouSSE was not installed (usually, an "Illegal
instruction" message and program termination).
   * As of version 0.32, MouSSE no longer behaves transparently when an
     unsupported instruction is encountered.  In order to assist
     troubleshooting, MouSSE loads the 16 bytes at the illegal opcode into
     XMM15, and the first 8 bytes of that into R15.  Both registers are used
     because while 16 bytes provides all the context necessary for analysis,
     some crash dumps do not include the XMM registers, so R15 is also loaded
     (8 bytes is better than nothing).  This violates the transparency
     principle, but given that the "illegal opcode" error is generally fatal,
     the effect is negligible.  If you know of a situation where this is
     problematic, please let me know so I can devise an alternative.
   * As of version 0.38, a "magic" illegal opcode is handled: the normally-
     illegal instruction 3f 55 44 ("?UD") returns "MouSSE42" in RAX, as a way
     to test that MouSSE is loaded and active.  Without MouSSE running, that
     instruction will throw a #UD exception on any system, new or old.

Aside from whatever IOKit uses at startup, MouSSE uses no dynamically allocated
memory, so there's no chance of a memory leak.  MouSSE does not create any
processes;  it's simply an extension of the kernel.

MouSSE doesn't read or write any files, or access any networks, or touch any
other devices.  All it does is watch for the instructions it knows about, and
emulate those when they turn up.  At present, no statistics are kept (it's all
about speed), and logging only occurs at load time.

MouSSE also does not check to see if your CPU already supports SSE4.2.  If you
install MouSSE on a newer system, it will load, but it won't ever do anything
besides take up a bit of memory, because newer CPUs can handle the SSE4.2
instructions themselves, and MouSSE will silently bear the loneliness of a
program that's never asked to actually do anything.
   * As of version 0.38, that's not entirely true - MouSSE does check for
     native SSE 4.2, and explicitly disables itself if the CPU natively
     supports it.  At present, there's no simple way to make MouSSE completely
     unload itself in this situation, so it still takes up kernel memory space,
     but never does anything else.

*** Why do I see "AAA.LoadEarly.MouSSE" in my kextstat output?
*** Why is the kext named "AAAMouSSE.kext"?

MacOS has a complex startup procedure.  Since the main point of MouSSE is to
allow AMD video drivers to work, MouSSE must load and initialize before the AMD
drivers do.  Part of the system initialization creates a dependency tree, and
MouSSE tries to put itself as close to the top of that tree as possible.
Outside of that tree, things are handled alphabetically - so, sticking "AAA" at
the beginning of the kext name puts it at the top of the alphabetical list (much
like finding "AAA-BestPlumbers" at the beginning of the Yellow Pages (does
anyone even use the Yellow Pages any more?)).  Ordinarily, the full name of the
kext would include a right-to-left domain name, such as "com.apple.xyzzy" - but
here again, by using a top-level domain named "AAA", MouSSE can push itself to
the top of the alphabetical list.  It's all just an effort to have MouSSE load
and initialize before anything that might use an SSE4.2 instruction.

*** Anything else?

    If you're running newer AMD drivers under a newer OS on an old Mac, you're
probably already running with SIP disabled.  If not, sorry - this software
requires SIP to be disabled.

    To simplify the IOKit interface, MouSSE has a small C++ wrapper, but all of
the important code is written in assembly language (for speed).

    Just to repeat myself, USE THIS SOFTWARE AT YOUR OWN RISK.  I've been using
it daily on my Mac Pro 3,1 with an AMD Radeon RX 570 for several months without
incident, but your mileage may vary.  If you lose time, money, or your sanity
because of flaws in this software, consider yourself forewarned.

    The semi-whimsical name came from elsewhere, but you can think of it as
"Macs oughta understand SSE" (MouSSE) if you like.  Or, if you're not fond of
your sibling, "My outrageously ugly sister sickens everybody."  Or, if you're a
megalomaniac, "My objective ultimately should supersede everything."  Whatever
makes you happy.

*** How do I install it?

I haven't written a proper installer.  For the time being, you need to do the
installation manually.  Because this forum doesn't allow uploads of .tgz files,
the .tgz is zipped - meaning you'll have to uncompress/expand it twice.
I suggest the following:
   1) expand the ZIP file, then the enclosed .TGZ file
   2) open a terminal
   3) cd {path to the .TGZ file}/MouSSE_RELEASE
   4) sudo tar cf - ./AAAMouSSE.kext | (cd /System/Library/Extensions;tar xf -)
   5) sudo kextcache -i /
(If you're installing to an alternate partition, prepend the root of that
partition to both paths, e.g. /Volumes/OtherDisk/System/Library/Extensions and
kextcache -i /Volumes/OtherDisk/)

*** How to I uninstall it?

Since MouSSE isn't the sort of thing you'll probably install and uninstall
regularly, I also haven't bothered creating an uninstall script.  To uninstall
MouSSE on the current running system, just do the following:
   1) open a terminal
   2) type "cd /System/Library/Extensions" (without the quotes), press ENTER
   3) type "sudo rm -rf AAAMouSSE.kext" (without the quotes), press ENTER
   4) type "kextcache -i /" (without the quotes), press ENTER
   5) reboot
(To uninstall MouSSE from an alternative disk/partition, change the path in
steps 2 and 4 to include the leading volume information, e.g.
"cd /Volumes/MyOtherMacOSinstallation/System/Library/Extensions" and
"kextcache -i /Volumes/MyOtherMacOSinstallation/")

Just to be clear, if you load the kext manually (via kextload or kextutil),
you can safely unload it and it will clean up after itself.

*** OK, I installed MouSSE, but now my new AMD video card is misbehaving...

    The most common problems occur if you've previously installed patches to
make legacy video cards work.  If those patches are installed, MacOS can get
confused about which display to use, which framebuffer to use for which display,
etc.  If you're experiencing problems with a supported modern AMD video card and
you still have those patches installed, you'll need to remove them (which
unfortunately, in most cases, involves reinstalling the OS).  Using a legacy
video card (to see the MacOS boot screen) and a modern AMD video card
simultaneously, without the "legacy video" patches, is a hit-and-miss
proposition - some combinations work some of the time, some don't work at all.
While it would be nice to have the "best of both worlds" (visible boot screen
and modern GPU acceleration), your best bet is to keep the legacy card on a
nearby shelf and only use it when necessary.

If you're having problems but don't have any legacy video patches installed,
please try contacting the author on the MacRumors forum.

VERSION HISTORY:
v0.20 - 15 Aug 2019 - First version released into the wild.
v0.30 - 12 Sep 2019 - Tweaks to allow installation from El Capitan onward
v0.32 - 20 Sep 2019 - Capture of illegal opcode data in XMM15/R15
v0.35 - 07 Oct 2019 - Added more initialization diagnostics
v0.38 - 17 Oct 2019 - Added more logging, "magic" opcode detection
