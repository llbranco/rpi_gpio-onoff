# rpi_gpio-onoff

#HowTo: Raspberry Pi Raspbian Power on / off GPIO button
From the beginning of playing with the Pi I wanted a way to power on and off the Pi without having to unplug the micro usb cable. For the most part, this is just out of fear of wearing out the USB connection, but it is also nice to have a physical button to perform relatively common and simple tasks.

Goal

This switch should turn the Raspberry Pi on if the Pi is sleeping, and it should initiate a clean shutdown if the Pi is on. This should be a physical button, not just a software remote button.

Please be advised, this projects rating is: More Difficult – Involves soldering, use of ssh, command prompt, and coding.

There are four major steps.

Wire and connect a physical push button to the Pi. This should be a Normal Open (N.O.) momentary contact push button switch.
Install the appropriate GPIO libraries, if necessary
Create a python script to shutdown the Pi when the button is pressed
Configure the script to run at startup

#Step 1 – Connecting a switch
First, make and connect a simple “wake-up” button. To wake the Raspberry Pi up when it is asleep, all you need to do is short Pin 5 to ground. (Pin 5 is also known as GPIO03). Since Pin 6 is already at ground, you can do this by shorting Pins 5-6.

Find a contact switch

I like to use a simple momentary contact push button like this one from Radio Shack (Click on any picture to enlarge).
push_button

You can find these for a lot cheaper online, but sometimes you just want to complete a project right now! I suppose I’m just too impatient sometimes.

The important thing is that you are looking for a Normal Open (N.O.) Momentary Contact switch. This will short the two pins only when the button is pressed. If you choose a Normal Close (N.C.) button, you may have additional problems (such as the Pi perpetually resetting)

Below is a picture of the button mounted in the case:

power_button_mounted

Header Strip Plug The next thing that you need is a header strip to plug onto to the Pi. You could use a regular female header strip like this one:

header_strip

If you have an old computer case laying around (which many of us do), you can just steal one of the cables connecting a case LED to the mother board. Motherboards have the same spacing as the Raspberry Pi header. For a previous project, I used a former speaker cable:

speaker_jumper

In this case, I used a jumper like the one shown here. I used two of them, one for each lead coming from your button.

jumper_cable

The advantage is that you don’t have to do any soldering to the straight pins. Also, it allows your switch to be removable, but the contacts are pretty solid. As long as you use solid wire, place a solid wire lead on each terminal of the switch. Then you can plug the wire directly into the jumper. You want the switch to be able to unplug from the Pi so that you can mount it in a case and remove it if necessary (which always seems the case with my projects).

I recommend covering each lead as well as both contacts together with some head shrink tubing. This gives a little support to the connection and also prevents unintentional shorts. In this example, the leads are shown.

Connecting to the Pi

When you are all done, simply plug your wired switch across pins 5 and 6 on the Pi, as shown below:

open_elec_reset_detail

The reset switch is connected using the green and black jumper cables. Since the switch only provides a short when pressed, it shouldn’t matter which lead is connected to which pin.

You’ll notice that this is shown with the original Raspberry Pi Model 2, but this should work with all versions of the Pi.

That’s it. You now have a button that can wake the Raspberry Pi from sleeping mode. This is a hardware switch, nothing needs to be done in software to wake the Pi from the sleeping state, it will automatically wake when Pin 5 is tied to ground.


#Step 2 – Installing the RPi.GPIO tools
Note: All of the following commands can either be done from the command prompt on the Pi, or through an SSH session. The default username for raspbian is pi and password is raspbian

If they are not already installed, you will want to install the following packages using apt-get with the following command :

sudo apt-get install python-dev
sudo apt-get install python3-dev 
sudo apt-get install gcc 
sudo apt-get install python-pip
Next you need to get RPi.GPIO by doing the following:

wget https://pypi.python.org/packages/source/R/RPi.GPIO/RPi.GPIO-0.5.11.tar.gz
Uncompress the packages by doing the following:

sudo tar -zxvf RPi.GPIO-0.5.11.tar.gz
Next move into the newly created directory

cd RPi.GPIO-0.5.11
Next, install the module by doing:

sudo python setup.py install
or

sudo python3 setup.py install

#Step 3 – Write and install the Shutdown Script
The following commands should be done either through the shell or through SSH. By default, SSH is enabled on Raspbian.

I recommend creating a directory to hold your scripts. In my case, I created the directory under the root /home/pi directory. It can be created by doing:

mkdir /home/pi/scripts
This will make a scripts directory under the /home/pi folder.

Next, we need to write the script. I personally use nano as my editor of choice, but you can use any editor you wish. Please Note: I highly recommend retyping the script. Do not copy and paste. It is possible that if you copy and paste, there will be slight changes in formatting (i.e. comment lines wrapping and no longer being comments) or incorrect characters (particularly quotes) that will keep your script from working!!!

We will call our script shutdown.py (it is written in python). Create and edit the script by doing:

nano /home/pi/scripts/shutdown.py
The content of the script should be:

 #!/usr/bin/python
import RPi.GPIO as GPIO
import time
import subprocess

 # we will use the pin numbering to match the pins on the Pi, instead of the 
 # GPIO pin outs (makes it easier to keep track of things)

GPIO.setmode(GPIO.BOARD)  

 # use the same pin that is used for the reset button (one button to rule them all!)
GPIO.setup(5, GPIO.IN, pull_up_down = GPIO.PUD_UP)  

oldButtonState1 = True

while True:
    #grab the current button state
    buttonState1 = GPIO.input(5)

    # check to see if button has been pushed
    if buttonState1 != oldButtonState1 and buttonState1 == False:
      subprocess.call("shutdown -h now", shell=True, 
        stdout=subprocess.PIPE, stderr=subprocess.PIPE)
      oldButtonState1 = buttonState1

    time singulair inhaler.sleep(.1)
Once you have created your script, you can test it by typing the following at the command prompt:

python /home/pi/scripts/shutdown.py
It may give warnings regarding a pull-up resistor. This is OK. If error’s are encountered, resolve them first. If it is able to run, try pushing the button and see if the Pi shuts down.

Once you have the script working the way that you want it, let’s make it run at startup.

#Configure our script to run at startup
To configure our script to run at startup, it should be added to /etc/rc.local

You can do this with the following:

nano /etc/rc.local
Add the following to the file

python /home/pi/scripts/shutdown.py &
That’s it! Now your script should run at startup. Once you restart your pi, you should be able to see shutdown.py running in the background. You can verify this either by pressing the button and seeing if your pi shuts down, or doing the following from the command line:

ps -A | grep shutdown.py
Good Luck!


[source]
http://www.barryhubbard.com/raspberry-pi/howto-raspberry-pi-raspbian-power-on-off-gpio-button/
