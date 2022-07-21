---
title: Native behavior for input devices in VMs
---

# Why not use default tools

The problem with the tools provided by virtualization software to pass inputs into VMs is that they only provide basic functionality, and you can't get native behavior for any device. The only way to do so is to pass a device directly into the VM, for example with USB Passthrough. 

But why not use the passthrough? First of all, you can't pass all devices as usb devices into the VM. For example a touchpad, touchscreen or keyboard integrated into a Laptop. Second, you cannot use the device on your host machine anymore.

But doesn't qemu allow for evdev passthrough? Well yes, but from my experience it's complete trash. It works for simple cases like a usb mouse or a keyboard. But anything else doesn't work properly.

This post shows a way to pass any input device (available in evdev) to a guest system. The devices will behave the same as they behave on the host machine. This method also allows switching between guest and host machine control with a simple keyboard shortcut.

# Setting up the VM

First of all, the OS used in the VM has to implement evdev. This would be the case for any GNU/Linux distribution.

Then you have to make sure that the VM uses server-side mouse cursor rendering. You can try client-side mouse rendering in your VM software, but from my experience with Spice it doesn't work (The input devices are usable, but the displayed cursor doesn't move). To enable server-side mouse cursor rendering with virsh using the Spice display protocol do:

~~~bash
virsh edit <VM name>
~~~

insert

~~~xml
<mouse mode='server'>
~~~

into the 

~~~xml
<graphics type='spice'>
~~~

tag, located in

~~~xml
<devices>
~~~

This will allow the mouse cursor to move from inside of the VM. This is also needed for any other type of passthrough, for example USB passthrough. But the displayed cursor lags all of the time, only updating its position if it mouses over a animated element. A way to get around this in GNOME and GDM is by enabling the Zoom feature in the Accessibility menu. Set the magnification factor to 1.0 and enable it. Now the mouse cursor is smooth, and the content is still displayed properly. Configure this for your GNOME desktop and GDM. For configuring the magnification factor check out [Configure GDM](/configure-gdm). To change the magnification factor from the terminal, execute

~~~bash
dconf write /org/gnome/desktop/a11y/magnifier/mag-factor 1.0
~~~

# Setting up the script

Now install [remote-evdev-python](https://github.com/Surferlul/remote-evdev-python) on your host and guest machine. For this you need a minimum python version of 3.10 with pip installed.

~~~bash
git clone https://github.com/Surferlul/remote-evdev-python
cd remote-evdev-python
./setup <machine type>
~~~

replace \<machine type\> with guest, host, or nothing (for both) respectively. Check for Permission errors. On the guest you need permission to /dev/uinput. On the hosts you need permissions to /dev/input/event\*. Permissions for these devices are often handled through the "input" group. Consider adding yourself to the group, or modifying the permissions for the input devices.

~~~bash
# Add yourself to the group
sudo usermod -aG input $USER
# Modify permissions for uinput
sudo chown root:input /dev/uinput
# Modify permissions for event devices
sudo chown root:input /dev/input/event*
~~~

You might need to allow traffic for the firewall on the guest machine.

~~~bash
sudo firewall-cmd --add-port=64654/tcp
~~~

Now you can modify the "host" and "guest" script respectively. The guest script can probably stay as it is. The host script will need the ip address of your guest machine and the evdev devices you want to pass to it. The devices are specified with "--device" have the options

~~~bash
# Path to the device
--path # filename in /dev/input/by-path
--id # filename in /dev/input/by-id
--event # filename in /dev/input
--full-path # full path to file

# Type of the device
--type # either pointer or keyboard. keyboards are used for switching between host and guest
~~~

To find a event for a device you can cat the event files in /dev/input, /dev/input/by-path and /dev/input/by-id. Try not to use the files in /dev/input, because those can change in contrary to the files in by-path and by-id. If you need to you can also switch the server and client roles around ("--server" and "--client"). But you'll also need to modify the ip addresses (probably 192.168.122.1 for the guest client and 0.0.0.0 for the host server).

Now you can start the server and then the client. You can switch between host and guest inputs by holding left and right control keys simultaneously. To quit press left control, right control and backspace simultaneously.