# Typewriter Caps lock with evdev and libinput

Modern Linux keyboard layouts exhibit several behaviors that may
surprise those who come from the Windows world and expect Caps lock to
behave like a manual typewriter, i.e. to turn off Caps lock you have to
press Shift:

aaaa \<Caps\> AAAA \<Caps\> AAAA \<Shift\> aaaa

Interestingly, there exists a solution for the venerable loadkeys (from
the KEYMAPS man page):

    The following entry sets the Shift and Caps Lock keys to behave more
    nicely, like in older typewriters. That is, pressing Caps Lock key once
    or more sets the keyboard in CapsLock state and pressing either of the
    Shift keys releases it.

    keycode 42 = Uncaps_Shift
    keycode 54 = Uncaps_Shift
    keycode 58 = Caps_On

However, while XKB\'s *shift:breaks\_caps* option has the same effect as
loadkeys *Uncaps\_Shift*, pressing *KEY\_CAPSLOCK* still toggles Caps
lock.

I searched far and wide for a solution to this problem, but to no avail;
therefore, I decided to take matters in my own hands and, after some
research, found out that I could prevent *KEY\_CAPSLOCK* from turning off
Caps lock by modifying a file called **evdev.c**, that is shipped in one of
two packages:

1.  X11 based distributions – evdev input driver:\
    **xserver-xorg-input-evdev** package

2.  Wayland based distributions – input device management and event
    handling library:\
    **libinput10** package

With the provided patch files this works out as follows:

1.  X11 based distributions – evdev input driver:

    1.  sudo su root
    2.  apt-get build-dep xserver-xorg-input-evdev
    3.  apt-get source xserver-xorg-input-evdev
    4.  patch -p0 \< xserver-xorg-input-evdev\_2.10.5.patch
    5.  ./configure \--prefix=/usr
    6.  make
    7.  cp -a src/.libs/evdev\_drv.so
        /usr/lib/xorg/modules/input/evdev\_drv.so

2.  Wayland based distributions – input device management and event
    handling library:

    1.  sudo su root
    2.  sudo apt-get install -y meson cmake
    2.  apt-get build-dep -y libinput10
    3.  apt-get install -y libgtk-3-dev libunwind-dev libsystemd-dev
        libudev-dev libmtdev-dev libevdev-dev libwacom-dev check doxygen
    4.  apt-get install -y python3-pydot python3-sphinx python3-recommonmark
        python3-sphinx-rtd-theme
    5.  apt-get source libinput10
    6.  patch -p0 \< libinput-1.14.1.patch
    7.  cd libinput-1.14.1
    8.  meson \--prefix=/usr build/
    9.  ninja -C build/
    10.  ninja -C build/ install
    11.  udevadm hwdb \--update

Log out, log in, and – et voilà – Caps lock should now behave like the Gods
intended ;-)

N.b.: for the above to work, it is necessary to enable XKB's shift(break_caps)
option!

Also, there is a small difference between using the accompanying .patch files
and the pre-patched evdev.c – the latter contains a BACKSLASH_ESC_CANCEL_CAPS
option, that (if enabled) results in the '#' key, that some keyboards offer,
to also disable Caps Lock … this is intended for Vim users, who may already
have mapped this key to 'ESC'; if the BACKSLASH_ESC_CANCEL_CAPS option is
then enabled on top of that, the '#' key does double duty: besides acting as
an Escape, it also turns of Caps Lock.
