Fn-Lock switcher for ThinkPad Compact Bluetooth Keyboard with TrackPoint
========================================================================

The Lenovo Thinkpad compact bluetooth keyboard is a repackaging of a Thinkpad
laptop keyboard into a bluetooth keyboard case. Unfortunately Lenovo had
already started their bizzare process of throwing away keys at this point, and
the Fn-keys have been replaced with some random functions Lenovo thought more
useful. The Fn-Lock switching is handled by the windows drivers, and
unfortunately is off by default.

There is also a physically-identical USB version of this keyboard. I'm guessing
that a lot of this would apply here too, but I do not have one to test.

Kernel module
-------------

I have written a HID driver to cope with the keyboard's quirks. It enables
fn-lock by default and allows you to toggle via. sysfs.

For more information, see the module directory.

tp-compact-keyboard
-------------------

If you don't want to install the kernel module, you can use the
tp-compact-keyboard script to at least enable fn-lock.

### Requirements

This was developed under a Debian unstable kernel, 3.12-1-amd64, and has been
reported to work for at least 3.11. However older kernels (3.6, for example)
don't send the report. Not sure why currently.

### Using

To enable/disable Fn-Lock, do "./tp-compact-keyboard --fn-lock-disable" or
"./tp-compact-keyboard --fn-lock-enable". This will find any bluetooth
keyboards connected and enable/disable. It has a few other functions too, but
they're not very useful on their own. Have a look at the source.

I'm assuming what you really want to do is force Fn-Lock on and forget about it
though. To do this, do the following as root:

    mkdir /etc/udev/scripts
    cp tp-compact-keyboard /etc/udev/scripts/tp-compact-keyboard
    cp tp-compact-keyboard.rules /etc/udev/rules.d/tp-compact-keyboard.rules

Then udev will enable Fn-lock every time the keyboard is connected.

### Other options

There are a few other options, however they are useless without a custom kernel
module handling the keyboard:

    --native-fn-enable
    	By default, F7/F9/F11 generate a string of keypresses that might be
    	useful under windows. This stops this, and instead only generates
    	custom events.
    --native-mouse-enable
        The middle button generates 2 events by default, but Linux only
        understands one of them. This disables the event that Linux does
        understand, leaving you with no middle button.
    --native-mouse-disable
        Restore middle button to a working state.

Pair/unpair isn't enough to reset the keyboard, you need to power down the
keyboard to get it back to it's original state.

How I did it
------------

I worked out the custom commands by sniffing what the windows driver did.
Firstly I set up an XP vm in KVM/QEMU and installed the drivers. Then I
disconnected the keyboard from my computer, and ensured that my user running
QEMU could write to the ``/dev/bus/usb`` node associated with my bluetooth
dongle.

Wireshark was readied for capture::

    # modprobe usbmon
    # wireshark

Then, I let windows use the bluetooth dongle::

    (qemu) usb_add host:0a5c:217f

If you capture the whole association process then you should be able to view
HID packets to/from the keyboard (filter for ``bthid``). The particuarly
interesting ones here are the ones going to the keyboard.
