---
title: Configure GDM
---

# How

All of the gdm display manager configurations are managed by the gdm user. Usually, options are controlled by obscure config files, and I have always found it to be an annoyance trying to find a way to change settings in gdm (example: try figuring out how to properly change the keyboard layout selection in gdm).

But since all the settings (which are regular gnome settings) are managed by gdm you can change them by gaining access to the gdm account and launching settings applications (gnome-control-center, gnome-tweaks, dconf-editor, dconf, gsettings, etc...). For some settings, you don't even need desktop access (changing the keyboard layout selection), but for others you do (changing display resolution), so I'll go over two methods.

# No desktop access

We can (after providing the gdm user authorization rights) simply execute applications as the gdm user and change settings.

First, we need to provide authorization rights:

~~~bash
xhost +SI:localuser:gdm
~~~

Then we launch an application:

~~~bash
sudo -u gdm dbus-launch <application>
# for example:
# launching the settings app
sudo -u gdm dbus-launch gnome-control-center
# launching a shell as the gdm user
sudo -u gdm dbus-launch $SHELL
# you can launch any application as the gdm user from that shell
~~~

Now revoke authorization rights again:

~~~bash
xhost -SI:localuser:gdm
~~~

# With desktop access

You can gain access to an entire gnome desktop as the user gdm. From there you can launch and configure how you want and will gain access to some settings like display resolution

To gain desktop access the gdm user needs to have a password

Setting a password for gdm:

~~~bash
sudo passwd gdm
~~~

![password example](https://raw.githubusercontent.com/Surferlul/configuring-gdm/gh-pages/pictures/gdm_password.png)

Now you can log out of your account and into gdm. You must press the "Not listed?" option in the user overview to log into gdm.

![Not listed?](https://raw.githubusercontent.com/Surferlul/configuring-gdm/gh-pages/pictures/login_page.png)

Now enter the username "gdm" and your chosen password to access the desktop.

![Username](https://raw.githubusercontent.com/Surferlul/configuring-gdm/gh-pages/pictures/uname.png)

![Password](https://raw.githubusercontent.com/Surferlul/configuring-gdm/gh-pages/pictures/passwd.png)

![Desktop](https://raw.githubusercontent.com/Surferlul/configuring-gdm/gh-pages/pictures/desktop.png)

Voilà! No gdm configuration pain anymore.
