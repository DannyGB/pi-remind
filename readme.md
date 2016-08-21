Raspberry Pi Appointment Reminder
=================================
I often find myself missing appointments because I'm engrossed in my work or because I've switched to a different computer and can't hear the reminder ping on my work laptop. I created this project to give me a visual reminder, an obnoxious, silent countdown timer I can set on my desk to flash lights at me as a warning before my next meeting starts.

The project uses a network connected Raspberry Pi and a [Pimoroni Unicorn HAT](https://shop.pimoroni.com/collections/raspberry-pi/products/unicorn-phat) to display the reminder.

Alerts
------

The Pi will connect to Google Calendar and check every minute for upcoming appointments then flash the LEDs for following alerts:

* White @ 10 minutes until 5 minutes
* Yellow @ 5 minutes until 2 minutes
* Multi-color swirl @ 2 minutes

If you're feeling adventurous, you can change the code to use any of the sample patterns included with the [Unicorn HAT Sample Code](https://github.com/pimoroni/unicorn-hat/tree/master/python/examples).

Indicator LED
-------------

The app uses a single indicator LED to let you know the app is working. It will illuminate a single LED along the bottom row of the Unicorn HAT and move the LED across the display every time it connects to Google to obtain calendar information. The color of the LED indicates status of the app as well:

* Blue - The app is connecting to the Google Calendar API
* Green - The app received data from the Google Calendar API, but there are no pending appointments within the next 10 minutes
* Red - The app encountered an error connecting to the Google Calendar API
* White - There is an appointment beginning within 10 minutes
* Yellow - There is an appointment beginning within the next 5 minutes
* Orange - There is an appointment beginning within the next 2 minutes

This way, even if you miss the flashing lights, you can glance at the display and still determine when your next appointment is.

Required Components
=================================
For this project, I used the following components:

+ Raspberry Pi - I used a [Raspberry Pi 2 Model B](https://www.raspberrypi.org/), but most any Pi will work. Check the Unicorn HAT documentation for supported Pi devices.
+ Power Supply - [CanaKit 5V 2.5A Raspberry Pi 3 Power Supply / Adapter / Charger (UL Listed)](http://www.amazon.com/CanaKit-Raspberry-Supply-Adapter-Charger/dp/B00MARDJZ4) from Amazon.
+ [Pimoroni Unicorn HAT](https://shop.pimoroni.com/products/unicorn-hat) from Adafruit.
+ Raspberry Pi case - [Adafruit Raspberry Pi B+ / Pi 2 / Pi 3 Case - Smoke Base - w/ Clear Top](https://www.adafruit.com/products/2258) from Adafruit.

The only required component is the Unicorn HAT as the code in this project is hand crafted for that device. Otherwise, pick whatever Raspberry Pi device, case and power supply that works best for you.


Raspberry Pi Setup
=================================
To setup the hardware, complete the following steps:

+ Mount the Pimoroni Unicorn HAT on the Raspberry Pi device
+ Place the Pi in a case
+ Power it up!

When the Pi is all ready to go, open a terminal window and update the device's software using the following commands:

	sudo apt-get update
	sudo apt-get upgrade

The first command updates the local software repositories and the second command updates the Pi OS and associated files. There’s a set of scientific libraries that are used in some of the Unicorn HAT code in the project, so you will need to install them using the following command:

    sudo apt-get install python-numpy

Next, create a directory for the project's files, execute the following commands in the terminal window:

	mkdir pi_remind
	cd pi_remind

Finally, copy the project's Python source code to the new folder and extract the files using the following commands:

	wget https://github.com/johnwargo/pi_remind/archive/master.zip
	unzip -j master.zip

If all goes well, you should see the following files in the folder:

- `LICENSE`
- `readme.md` (this file)
- `remind.py`
- `setup.py`
- `start-remind.sh`

Before you can use the project's software, you have to setup an account with Google so the app can consume the Google Calendar APIs used in this project. To setup your account, read the [Google Calendar API Python Quickstart](https://developers.google.com/google-apps/calendar/quickstart/python).

Download your Google Calendar API application's `client_secret.json` file in the project folder. Be sure to name the downloaded file using that file name. You'll need it to authorize the app to access your Google Calendar and that file name is hard coded into the Python app.

As part of that process, you'll install the [Google Calendar API Python files](https://developers.google.com/api-client-library/python/start/installation) along with date handling libraries using the following command:

    sudo pip install --upgrade google-api-python-client python-dateutil pytz

Install the Unicorn HAT libraries following the instructions on the [Pimoroni web site](http://learn.pimoroni.com/tutorial/unicorn-hat/getting-started-with-unicorn-hat):
basically opening a terminal window and executing the following command:

    curl -sS get.pimoroni.com/unicornhat | bash

With everything in place, execute the reminder app using the following command:

    sudo python ./remind.py

Before the app can access the calendar, you'll need to authorize the app to use the Google Calendar API for your calendar account. When you launch the app for the first time (using the command shown above) the browser will launch and walk you through the process. With that complete, PI Remind should start watching your calendar for events.

Note: if you ever change Google calendars (from a work to a personal calendar or from one work calendar profile to another) you'll need to whack the existing access token created during the initial startup or the Pi Reminder app. Instructions for deleting this token are available on [johnwargo.com](http://www.johnwargo.com/index.php/microcontrollers-single-board-computers/pi-reminder-%E2%80%93-delete-google-calendar-access-authorization-token.html).

Starting The Project's Application's Automatically
--------------------------------------------------

There are a few steps you must complete to configure the Raspberry Pi so it executes the the remind app on startup. If you don't already have a terminal window open, open one then navigate to the folder where you extracted the project files. Next, you need to make the project's bash script files executable by executing the following command:

    chmod +x start-remind.sh
    
Next, you'll need to open the pi user's session autostart file using the following command:  

	sudo nano ~/.config/lxsession/LXDE-pi/autostart    

Add the following lines to the end (bottom) of the file:

	@lxterminal -e /home/pi/pi_remind/start-remind.sh

To save your changes, press `ctrl-o` then press the Enter key. Next, press `ctrl-x` to exit the `nano` application.
  
Reboot the Raspberry Pi. When it restarts, both python processes should execute in its own terminal window.  

Known Issues
=================================
Reminders are triggered for canceled events. If you have your Google Calendar configured to show deleted events, pi_remind will flash its lights for those events as well. I've tried setting `showDeleted` to `false` in the call to get the calendar entry list from Google, but it does not seem to have an effect (in my testing anyway).

Revision History
=================================
2016-08-21: Removed .desktop file and replaced it with a shell script that actually works.

2016-08-16: Changed activity lights to blue for checking and green for success. That seemed to be better as it now has red for failure and green for success. Made constants out of the colors used so the app's behavior can be changed more easily in one place. 

2016-08-15: Updated the `show_activity_light` function, renamed the function to `set_activity_light` and updated it so it takes a `color` paremeter. Modified the flash_all_lights to take a `color` parameter. Renamed `flash_all_lights` and `flass_random_lights` to `flash_all` and `flash_random`.
 
2016-08-09: Added code to set the status light after flashing the LEDs. Otherwise, within 10 minutes of an event, you wouldn't see the status light indicating that the app is running. 

2016-06-28: Added red LED option for current_activity_light when there's a failure connecting to Google. Added this for a visual indicator when errors occur and the user misses the red flash.
 
2016-06-27: Fixed the progress indicator LED so it cycles from left to right and so it leaves the light blue on success so you can tell at a glance that the Pi is still running the code.

2016-06-20: Reader John McCabe submitted a patch to fix an issue where events with empty descriptions were causing an error. If the app encounters an event without a summary, it sets the output to `No Title`.

2016-06-18: Reader John McCabe submitted a patch to update the code so it takes into account the local timezone when working with dates/times. In my original code, I hardcoded my time zone into UTC conversions and intended to document this so others could update it to match their current timezone. John implemented a better solution, using some Python Date/Time utilities to properly reflect current time zone into calculations.  


***

You can find information on many different topics on my [personal blog](http://www.johnwargo.com). Learn about all of my publications at [John Wargo Books](http://www.johnwargobooks.com). 