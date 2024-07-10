---
title: "NPR Alarm Clock"
date: 2020-6-15 15:33:33 -0500
permalink: "/NPRAlarmClock/"
image: /assets/Images/NPRLogo_CMYK.jpg 
theme: minima
excerpt: "Starting my day with Morning Edition"
---

# NPR Alarm Clock
_Using selenium and a chron job to create a simple alarm clock that plays NPR local news radio every weekday morning._
What better way to wake up than with NPR's [Morning Edition](https://www.npr.org/programs/morning-edition/)? I was inspired to make this project as a way to combine Python programming practice and my enjoyment of listening to NPR in the mornings. I used a cron job on my mac to run the python script each morning at 8:10am. 

## Selenium 
The selenium package opens an instance of a Chrome browser and presses the "Play" buttons for you. The chromedriver can be downloaded [here](
https://chromedriver.chromium.org/downloads). Be sure to match the version of chromedriver with your version of Chrome. 

The main function of selenium in this case is to search the html of the [NPR Morning Edition](https://www.npr.org/programs/morning-edition/) page to find the class of the object needed to play the live radio. See the main chunk below:

```python
# set chrome driver to open without GUI
options = webdriver.ChromeOptions()
options.add_argument('headless')

# Using Chrome to access web without GUI
driver = webdriver.Chrome(executable_path='./chromedriver', options=options)  # headless (no window)

# Open the website
driver.get('https://www.npr.org/programs/morning-edition/')
```

Make sure to include the WebDriverWait lines to prevent this error:
```python
selenium.common.exceptions.NoSuchElementException: Message: no such element: Unable to locate element: {"method":"css selector","selector":".btn-live-radio"}
```
Without it your code barrels on to the next command without waiting for the webpage to update, so it thinks the element for the hidden button is missing.

## Cron job
This cron job will run the NPR_Alarm_Clock.py file every weekday morning at 8:10am. 

```
SHELL=/bin/bash
PATH=/usr/local/bin/:/usr/bin:/usr/sbin
10 8 * * 1-5 export DISPLAY=:0 && cd /path/to/python/file && python3 NPR_Alarm_Clock.py
```

- **NOTE: cron job will not execute when your computer is asleep.** I am looking into a way around this, but the simplest work around for now is to schedule your computer to wake up a minute before the cron job is set to execute-
- On a mac: System Preferences > Energy Saver > Schedule

## Features to Implement
- Ability to pause and unpause cron job (need to look into command line more)
- Include case for closing the browser window early without erroring out
- Chromedriver does not automatically update, causing the script to fail when a new version of Chromedriver is available. Looking into combining some other open source for auto-updates into this project
