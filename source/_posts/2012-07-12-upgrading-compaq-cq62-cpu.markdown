---
layout: post
title: "How-to: Upgrade the CPU of a Compaq CQ62"
comments: true
date: 2012-07-12 12:00
categories: 
- guide
---

I've wanted to document all the steps I went through when upgrading my notebook's CPU and finally made a Blog where I could do it!

But before we continue, some background information and some precautions are in order:

## Useful Resources

HP Compaq CQ62-A20SB Materials:

* [Product Specifications](http://h10025.www1.hp.com/ewfrf/wc/document?docname=c02286055&tmp_task=prodinfoCategory&cc=us&destPage=product&dlc=en&lc=en&product=4267247)
* [Service Manual](http://h10032.www1.hp.com/ctg/Manual/c02542472.pdf)

Thermal Paste:

* [Comparison of thermal pastes](http://benchmarkreviews.com/index.php?option=com_content&task=view&id=150&Itemid=62)
*  <p id="thermalpaste"/>  [Comparison of application methods](http://benchmarkreviews.com/index.php?option=com_content&task=view&id=170&Itemid=1) <p id="thermalpaste"/>

## Disclaimer

I take no responsibility whatsoever for any damages caused by following this guide. I only intent to document the steps and will provide warnings where necessary,
but if you choose to follow this guide, proceed at your own risk.

## Precautions

To prevent any damages to the hardware due to static electricity I took the following precautions:

* Anti-static wristband
* No carpet at the workplace

## Other things you will need

* Philips screwdriver
* Flat-bed screwdriver
* Thermal paste & cleaner
* Something to put the screws in (preferably seperated so you can tell which are which)

## Some remarks about the manual

All the steps are described pretty well in the manual except for the following:

Optical drive bay

* You first need to remove a screw before you can continue to push out as per manual's instructions

LCD Display

* Unplugging the connectors of the display is sufficient for reaching the CPU

Keyboard

* The keyboard is attached with little glue pads that need some force before they come loose

Top Cover

* This also requires some force to come loose

## Getting Started

### Step 1: Prepare the notebook & removing the battery

* Shut down the notebook (no hibernation/sleep)
* Unplug the power adaptor
* Remove the battery
   * Push the slider sidewars
   * Pull out the battery
* _Press the power button_ to discharge any electricty left in the circuit

![Battery removed](/assets/images/notebook/3_BATTERY.jpg "Battery removed")

### Step 2: Remove the WLAN/ HDD cover plates

Unscrew the two cover plates which are both attached with 2 screws each

### Step 3: Remove the HDD

Pull out the HDD bay and disconnect the SATA connector

![Cover plates and HDD removed](/assets/images/notebook/4_COVER_PLATES_CROPPED.jpg "Cover plates and HDD removed")

### Step 4: Remove the RAM modules

![Ram modules](/assets/images/notebook/5_WLAN_RAM_RTC.jpg "WLAN modules")

* Loosen the clips
* Pull out the RAM modules at an angle

### Step 5: Remove the WLAN module

* Unplug the main and auxiliary antenna
* Remove the screw
* Pull the WLAN module out at an angle

### Step 6: Remove the RTC Battery

* Unplug the connector
* Remove the RTC battery which is attached with an adhesive strip to the bottom of the case

![WLAN/RAM/RTC Battery removed](/assets/images/notebook/7_WLAN_RAM_RTC_REMOVED.jpg "WLAN/RAM/RTC Battery removed")

### Step 7: Remove the Optical Drive Bay

* Remove the screw above where the RTC Battery used to be
* Push the pin in the hole(the one that has an arrow pointing at it) with a small screwdriver

### Step 8: Remove the Keyboard

Remove 6 screws marked with a keyboard icon spread over the bottom of the case

* 3 of the screws are inside the battery compartment

Turn the notebook around (upside up)

* Unhinge the keyboard (you can push it out through the bottom of the notebook case)
* Carefully pull up the top of keyboard so the adhesive strips come loose (mind to not pull to much on the connector)
* Unlock the ZIF (Zero Insertion Force - they mean it) connector
* Pull out the keyboard

![Keyboard disconnected](/assets/images/notebook/9_KEYBOARD_DISCONNECTED.jpg "The keyboard is disconnected")

### Step 9: Remove the Top Cover

_Caution! The weight of the notebook has diminished enough to make the LCD the heaviest part and it is no longer in a balanced position when placed on a desk. Make sure to support the LCD when proceeding. Or, if possible, rest the notebook on the back of the LCD when removing the cover._

* Turn the notebook back around (upside down)
* Remove 4 + 9 + 3 screws, all marked with a triangle
    * 4 screws are inside the battery compartment
    * 9 screws are spread across the bottom of the notebook (mostly around the edge)
    * Turn the notebook around (upside up)
    * Remove 3 more screws in the keyboard compartment
* Remove all the connectors visible in the keyboard compartment
    * 1 at the top left (power connector)
    * 1 at the bottom left (connecting the speakers)
    * 2 in the middle (touch pad and touch pad buttons)
* _Support the display_
* Lift the rearside of the top cover
    * Quite a lot of force is needed to unhook the clips, primarily around the display hinges
    * Pull out the top cover at an angle towards the display. Make sure to support the LCD because of the shift in weight
    
![The top cover](/assets/images/notebook/11_TOP_COVER.jpg "The top cover")

### Step 10: Remove the System Board

![Top cover removed](/assets/images/notebook/12_TOP_COVER_REMOVED.jpg "The top cover is removed")

After having removed the top cover you can finally see the system board.

Over on the left:

![Speaker and display cables](/assets/images/notebook/15_DISPLAY_SPEAKER_CONNECTOR.jpg "Speaker and Display cables")

* Remove the speaker connector (middle left)
* Remove the display connector (top left)
* Unroute these 2 cables (Mind the adhesive strips)

![Speaker and display cables disconnected](/assets/images/notebook/16_DISPLAY_SPEAKER_DISCONNECTED.jpg "Speaker and display cables are disconnected")

Over at the right:

* Unroute the WLAN cables
* Disconnect the USB ZIF connector
* Disconnect the power cable

![USB cable disconnected](/assets/images/notebook/14_USB_WLAN_CABLES_DISCONNECTED.jpg "USB cable is disconnected")

![Power cable disconnected](/assets/images/notebook/17_POWER_CONNECTOR_DISCONNECTED.jpg "Power cable is disconnected")

At the front (closest to you):

* Disconnect that other random connector

![Mysterious connector](/assets/images/notebook/12_TOP_COVER_REMOVED_MYSTERIOUS_CONNECTOR.jpg "The mysterious connector")

Finally:

* Remove 4 more screws holding the system board in place (1 is hidden over at the fan)
* Pull out the board at an angle towards the right side of the notebook (the system board swivels on the left side where all the external connectors are normally located)

![The hidden motherboard screw](/assets/images/notebook/15_DISPLAY_SPEAKER_CONNECTOR_SCREW.jpg "The hidden motherboard screw")

### Step 11: Remove the Heatsink

![The system board](/assets/images/notebook/19_SYSTEM_BOARD_TOP.jpg "The system board")

We're in the danger zone people...

![The heatsink](/assets/images/notebook/20_SYSTEM_BOARD_BOTTOM.jpg "The heatsink")

* Disconnect the cooling fan cable on the system board ( And make a mental note _TO PUT IT BACK LATER_ )
* Loosen the 4 screws pressing the heatsink down on the CPU in the correct order (Check the _manual_ for the correct ordering)
* Pull the heatsink up

### Step 12: Remove the CPU

The CPU is held in place with a locking screw. Turn it _counter clockwise_ half a turn.

![CPU](/assets/images/notebook/22_SYSTEM_BOARD_HEAT_SINK_ZOOMED.jpg "CPU")

You can now remove the CPU but be careful _not to bend the pins_ while doing so. Alse, note how it was oriented in the socket (There is a mark on one of the corners, a triangle, that identifies how it should be fitted)

### Step 13: Prepare the replacement CPU and the Heatsink

If it was a second hand CPU, clean it first to remove the thermal paste.
Do the same to the relevant parts of the heatsink. 

Make sure to check if there are _other components_ making use of the Heatsink. For instance, a GPU, that might also need cleaning and new thermal paste.

Now apply the thermal paste as per [your preferred method](#thermalpaste).

### Step 14: Installing the CPU

Place the CPU in the socket in the _same orientation_ as it originally was in. If placed correctly, it will fall in almost by itself. _Do not use any force_ as that may bend a pin. As said before, if placed correctly it will not need force to fit.

Lock in the cpu with the locking screw and fasten it by turning it half a turn _clockwise_.

### Step 15: Installing the Heatsink

_Caution! Make sure the cooling fan's connector is out of the way so it doesn't get locked in between the heatsink and the system board when fitting the heatsink_.

* Put the heatsink in place
* _Connect the cooling fan cable_
* Fasten the screws in reverse order pressing the heatsink against the CPU.

### Reassembling the notebook

Just reverse all the instructions going from step 10 back to 2.