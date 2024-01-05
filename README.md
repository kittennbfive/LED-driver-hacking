# LED-driver-hacking

## What is this?
This repo contains some notes about hacking driver boards for LCD LED backlights to use them with high power LED for lighting, ...  
It is *not* a tutorial, just a collection of notes and stuff you might find useful. You have to be familiar with boost converter basics and other electronics stuff, know how to solder, ...

## Licence and disclaimer
The content of this repo is provided under CC BY-NC-SA 4.0 and WITHOUT ANY WARRANTY! It is experimental stuff! Mains voltage can be deadly, high power LED are **really bright**, stuff can get **really hot really quick**, components can explode if things go horribly wrong and so on.

## Why?
Well, it all started with curiosity (that killed the cat as Schroedinger teached us). Quite some time ago i had to order some components from a big distributor, saw high power LED on their webpage for little money and thought "That looks interesting, lets just randomly order one". A little later i had a 24W CREE LED on my bench and i had a problem: This particular version has a forward voltage of about 36V (@600mA constant current). My lab supply does not go that high for a start, i had to put both channels in series to try the LED. Damn, that's *bright* and gets hot *really* quick! I then thought about a use for this thing and found that a (somewhat) portable lamp (mains powered) might be useful. However those cheap (and sometimes nasty) buck ("step down") converter modules with adjustable current you get from Aliexpress and similar places can only handle like 30V or maybe 35V; that's not enough. Also finding a cheap and small power supply with like 40V output proved difficult; 48V stuff does exist but as it is with industrial stuff it is not cheap (and i don't trust no-name mains power supplies...). So i put the LED aside.

## The breakthrough (kindof)
Recently i was browsing Aliexpress and saw a "LED LCD TV backlight driver panel" (that's how they named it, if you are looking try "LED backlight driver" or similar keywords). The description was rather cryptic, but it looked like this thing can supply a constant current with an output voltage up to 75V and 60W output power at 24V input voltage. And at this time i had an old laptop power brick laying around that outputs 19V. 19V... 24V... 65W... 24W LED... You guessed it - i ordered one.

## The board
The board i got is based on an BIT3260. You may or may not find the confidential datasheet on the internet. Further main components are a MOSFET ME15N10-G (datasheet available), a big inductor, some capacitors, diodes and a lot of small stuff. The output is current regulated and the current is "programmed" by shunt resistors on the low side of the LED output. I entirely reversed the board, see `schematic_driver_board.pdf`. I also made 2 pictures with my microscope showing the important bits of the circuit (`board1.jpg` and `board2.jpg`). **Please be aware that the board has a flaw that might kill the BIT3260. Read on.**

### Details
On the output side there are only 2 connections, LED+ and LED-.  
On the input side the board has 4 pins:
1. Vcc
2. On/Off
3. Vadj
4. GND

1 and 4 should be obvious.  
  
2 is simple too, just wire it to Vcc to enable the beast. **Please note that a boost converter has an almost direct connection from input to output even if switched off.** That's not a problem if you have a LED with a big forward voltage, but if e.g. you have a load resistor on the output to test the converter it might be a problem or at least something to remember!  
  
Now comes 3. Basically this pin allows you to dim the LED, but the way this is working is a bit weird (in my opinion): By providing an external voltage and through a diode (to block reverse current) you inject some current into the feedback loop or said otherwise, you add a small voltage to the voltage produced by the LED-current going through the shunt resistor(s). The range of the external voltage is from 0V (LED fully on) to about 3,3V+0,6V from the diode (LED off). That's a bit weird, as ~3,9V is not a "standard" voltage like from a DAC? If this gives you headache you may replace D3 by a schottky diode with less drop and/or reduce R14 to increase "sensibility" (i did not try).

### A fatal flaw
Power to the BIT3260 is provided via a simple voltage regulator based on a Zener and a BJT. The Zener used is a 24V one, however the BIT3260 has an absolute maximum of 18V Vcc! **You need to replace the Zener diode with a 15V version**, at least if you want to power the board with a voltage higher than 18V. Don't use a Zener with a voltage much lower as the MOSFET needs some voltage to work properly (power dissipation if not fully switched on!).

### Current regulation
It's all about Ohm's Law... The BIT3260 does control the MOSFET so that the voltage over the shunt resistor (several in parallel for power dissipation and/or value fine tuning) is equal to 0.2V. Said otherwise, just multiply your needed current by 5 (1/0.2) and you get the resistance you need. Don't forget to check power dissipation when choosing a suitable shunt resistor.

### A(nother) word of caution
For high power LED a heatsink with passive or even active cooling is **mandatory**! Dont burn yourself or your LED.

## Making my own lamp
This section describes the different parts of my "creation". I used a lot of stuff i had laying around, so the result might look a bit weird but it works. For the electronics i built and tested each module (read on) separately, that's why the inside of the case (from another PSU for a long gone analog camera if i remember correctly) looks like a mess...
![picture of lamp front](/images/picture_lamp_1.jpg)
![picture of lamp side](/images/picture_lamp_2.jpg)
![picture of lamp bottom](/images/picture_lamp_3.jpg)
![picture inside case 1](/images/picture_inside_case_1.jpg)
![picture inside case 2](/images/picture_inside_case_2.jpg)

### Mechanics
The LED and the fan are mounted on a heatsink that was bodged to the mounting of a (long gone) security camera, so the position of the LED/the light beam can be somewhat adjusted. As i was not sure about screwing a somewhat heavy assembly on the (old) plastic case i added a piece of wood that i glued to the case. Yeah, as i said, i did not want to spend a lot of money on this and had (way to much) stuff laying around...

### Power supply
I used a 19V laptop PSU that had a bad connection and was fixed but no longer needed. It can output like 65W or so, more than enough and - most important - is safe to use, as opposed to a random 24V or similar PSU that might have bad insulation, no input filtering and so on...

### Modifications of the driver
As said above i replaced ZD1 to not blow up the main IC. I also removed R8 (5.1Ω) and soldered a 1.8Ω resistor instead to change the LED current from 478mA to 550mA (my LED is rated 600mA and a little headroom is probably a good idea).

### Dimming circuit
I wanted to be able to dimm the LED with a 10kΩ potentiometer, so i made this:
![schematic of dimming circuit](/images/dimming.png)

RV1 is a small, internal potentiometer to allow fine tuning of the circuit/voltage going into Vadj.

### Active cooling (or how to (not) shave a yack)
I had a heatsink from some other device laying around that by chance had suitably spaced holes for fixing the LED. This heatsink was intended for some other component(s), that's why the LED is not centered. As it is quite small i had to add a 12V fan. For maximum comfort i wanted an automatic adjustment of the fan speed depending on the temperature of the heatsink. This part **really** gave me headache... The basic idea was to use a small buck converter from 19V to 12V (max) and put a NTC (i picked one randomly on Aliexpress, that was not a good idea too as there are several variants. Ask Wikipedia and do some maths before you order!) in the feedback-loop. To keep wires inside the feedback-loop short i glued the buck converter onto the case of the fan. The NTC was "mounted" to the heatsink with a piece of polyimide tape.
![picture of buck converter](/images/picture_buck_fan.jpg)

This basically works fine, as the temperature increases the fan gets more power; *but* if things get too hot (or the NTC is somehow shorted) the voltage for the fan may exceed 12V. That is bad. *A better/simpler solution would be a fan with a speed control input and some circuitry based on e.g. an NE555 to provide the speed control signal*, but i didn't want to order another fan...  
  
I tried to find a *simple* (few and standard components) solution to this and failed a lot of times, like with this circuit based on how feedback works in primary side switch mode power supplies.
![schematic of (non working) foldback circuit](/images/foldback.png)

In principle this can work, however in reality the buck converter got crazy and the output started to oscillate horribly once my circuit kicked in. This was somewhat expected as we have two feedback loops that will "fight" each other. I assume that with some fine tuning, a little capacitance and maybe another resistor or two this circuit could be stable, however this stuff is way over my head and i dont't even have documentation for the buck converter (from Aliexpress). *If somebody knows how to make this (maybe) work and/or has found some _simple_ to understand tutorial/documentation/... please share (open an issue)!*  
  
Long story short, i finally decided to "just" cut power to the entire device if the output of the buck converter exceeds 12V. Thinking about it that's probably the best solution because if the fan voltage goes too high it means that either the LED/heatsink is too hot or something is wrong with the NTC/fan control. After some messing with LTSpice i had this schematic that worked fine in simulation, so i built it for real. The MOSFET i had in stock are SMD (SOT23-3), so i used only SMD components for this circuit, except for the TL431 because i didn't had a SMD version in stock... While designing such a circuit you have to be *very* careful with Vgs_max, i fried one MOSFET and it was a pain to replace...
![schematic of cutoff/protection circuit](/images/cutoff_partial.png)
![picture of cutoff/protection circuit](/images/picture_cutoff.jpg)

(The TL431 in THT is on the other side of the PCB.)  
The red LED (D2) inside the circuit may look odd, but it is really important: It provides an offset voltage (remember that a *red* LED has a forward voltage of about 1.6V) so that Q1 turns off even if Vgs does not reach 0V because the TL431 can only pull down to about 1V. With the LED in place the Gate from Q1 sees a *slightly negative* voltage if TL431 triggers, so it will turn off for sure. The LED can also double as a power indicator if you want.  
  
On power on Q2 will be off and so will be the entire lamp, so i added C2 to briefly pull down the Gate of Q2 (through a voltage divider, remember the warning about Vgs_max) and make Q2 conduct. That worked fine in simulation and with a laboratory power supply, but *not* with the real power supply i wanted to use. It turns out that this thing ramps up its output voltage slowly, so there won't be a sufficient (or even any) voltage drop (as C2 charges up) that would enable the entire device. Even more headache. :-(  
![oscilloscope trace of slow rising Vcc](/images/scope_power_on_psu.png)
  
Finally i came up with the circuit on the left side that uses a second TL431 and a Zener diode to "activate" everything (ie make Q2 conduct) even if Vcc ramps up slowly.
![schematic of cutoff/protection circuit with startup/reset addon](/images/cutoff_full.png)
![oscilloscope trace for reset addon](/images/scope_reset_pwr_on.png)

This proved to work fine. If the overvoltage/overtemperature circuit triggers just switch off the entire thing, wait until it cooled down (or fix the broken NTC if that is the problem) and power it on again.

### Components used
LED: CREE CXA1512-0000-000N0HM240G (might be NRND/discontinued)  
mounting bracket: LedLink Optics LL01A00CSLB2 (available from Mouser)  
lens: LedLink Optics LL01CR-CEW60L02 (available from Mouser)  
