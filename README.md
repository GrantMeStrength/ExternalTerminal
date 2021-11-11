# ExternalTerminal
 
How to connect an external terminal to a Mac and PC.
 
## Connecting an external terminal to a Mac
 
![Heathkit Terminal](terminal.jpg "A serial terminal connected to a Mac's terminal")
 
It is possible to connect an external physical terminal to a Mac, and use it to display a remote terminal session. For example, most vintage terminals use RS232 serial ports to connect, and these can be connected to a Mac via a USB/Serial Dongle.
 
Before going any further, it's worth checking the serial link is working. The simplest way to check everything is good is to connect the Mac via the USB/Serial Dongle to the terminal, and start a terminal emulator program such as [Serial](https://apps.apple.com/us/app/serial/id877615577?mt=12) on the Mac. Serial will list all the available ports, select the USB dongle and connect. Set both systems to the same baud rate and anything you type on the Mac should appear on the terminal, and vice versa.
 
Note: Not working? Check that you have the right serial cable, and if necessary use a NULL MODEM adaptor to correct the wiring. Still not working? Are you sure the external terminal is working? Cross over the RX and TX pins on the terminal's serial port to see if you get an echo (i.e. with local echo turned off, every keypress should still be displayed on the screen). Confirm baud rates, parity and bit settings are the same. You can't go wrong starting with 9600N8 and then speed up from there.
 
### Running the shell on the external terminal
 
This is where it gets cool: it's possible to get the Mac terminal displayed on the remote terminal. It will be text only, but you can still use git and all your other command line tools.
 
1. Find the name of the serial port you are using. e.g. the USB dongle.
 
You should be able to find it by typing **ls /dev/cu* on the Mac's terminal.
 
2. Start a screen session
 
Enter:
 
**screen /dev/NAME_OF_YOUR_SERIAL_PORT SPEED**
 
e.g.
 
**screen /dev/cu.usbserial-AC00I4ST 9600**
 
3. Start the shell:
 
When you launched the screen command, the display should have cleared.
 
3.1 Press **CTRL A** *(nothing appears to happen)*
 
3.2 Press **:** *(a colon appears at the bottom of the screen)*
 
3.3 Enter **exec ::: /usr/libexec/getty std.9600** (yes, three colons. You should see a login prompt when you press enter)
 
Now you can go to the external terminal, and log in with your regular Mac login/password.
 
When you are finished, enter **CTRL A** and **:** and then enter **Quit** or you'll leave the serial port open.
 
This has worked for me on a Catalina and later system, using a USB to serial dongle, and also with a serial to BlueTooth adaptor.
 
### Cleaning up the display
 
The Mac Terminal will try to send interesting characters to your remote terminal, and some of these may appear as nonsense depending on how smart the terminal is. For really old terminals, I found I had to revert to BASH from ZSH, and then set PROMPT_COMMAND to nothing.
 
* **chsh -s /bin/bash**
* **PROMPT_COMMAND=""**
 
It is also possible to set up a getty interface at boot, although this has been really hit-or-miss for me. You can read about the process [here](http://www.club.cc.cmu.edu/~mdille3/doc/mac_osx_serial_console.html).
 
 
## Connecting an external terminal to a PC under WSL2

![Wyse Terminal](wyse.png "A serial terminal connected to a PC WSL2 session")
 
WSL is the "Windows Subsystem for Linux", a Microsoft project that allows Linux distributions to be run on Windows. The latest version is WSL2, which until recently didn't support USB devices. However, a newly published [Tabs vs. Spaces](https://youtu.be/I2jOuLU4o8E) video explains how a third party tool brings USB support to WSL2.
 
Although USB support is most useful for developers wishing to access USB hardware, such as webcams or Arduinos or other microcontrollers, it is also ideal for connecting a USB-to-Serial Port adaptor. This makes it possible to connect an external Terminal (for example, some vintage hardware that has a RS232 port) and then log into the Windows computer's WSL shell. I make no judgement on how useful this is in practical terms.
 
Here is how I got this working on a Windows 11 laptop.
 
Note: Some older USB-to-Serial dongles use a chipset that is not supported by Windows 11 - I hit this issue with the first dongle I tried.
 
1. Follow the instructions at [Connecting USB devices to WSL](https://devblogs.microsoft.com/commandline/connecting-usb-devices-to-wsl/) to get **usbipd-win** installed. I used winget to install the package. Install the Linux tools on the WSL side, as directed in the article.
 
2. Plug in the USB-to-Serial dongle. Use the **usbipd wsl list** command to find the ID of your dongle, and then *attach* the USB device: for example **uspipd wsl attach --busid 1-1**. The USB device will disappear from the Windows side, and become a Linux device instead.
 
3. On the WSL side, check the dongle is now present by looking in the /dev/ directory with an **ls /dev/tty***  It will probably be called ttyUSB0.
 
4. On the WSL side, enter: **sudo agetty -L 9600 ttyUSB0 vt100**
 
This is different from using the Linux **screen** commmand, and has some issues - for example, Control C has to be issued from the PC side. I found that **screen** just didn't work for me, and I don't really know why. I did re-install **screen** (a tip that I found online), but I found that it always returned immediately without connecting.
 
You can now connect the terminal to the dongle (you might need a null modem adaptor depending on the cable you are using) and log into WSL with the WSL default parameters. This will put you in a different user directory from the default WSL prompt. You can still explore it from the Windows side by opening a File Explorer window and going to **//wsl$** and drilling down to **Ubuntu > home > [username]** (assuming you are using Ubuntu on WSL of course).