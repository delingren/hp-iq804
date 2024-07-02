# HP Touchsmart IQ804 all-in-one PC conversion project

## Context
I found this all-in-one PC at a PC recycle bin long ago (circa 2014). It was used as a guest computer in my house for a while. Now it's way too old even for that purpose. HP Touchsmart IQ804 (marketed as IQ800) is an all-in-one PC released in 2008. It was originally equipped with Windows Vista and is upgradable to Windows 7. I never tried but I doubt it'd be able to run Windows 10. There's no way it'd be able to run Windows 11. Instead of throwing it away, I decided to harvest as many components as I can. In particular, the 26 inch, 1920x1200 LCD panel is crispy clear and I really love it.

## Goal
Convert this all-in-one computer into an external display and docking station, reusing as many components as possible.

Primary goals: 
* Reuse the display
* Reuse the speakers

Secondary goals:
* Reuse microphones
* Reuse volume control buttons
* Reuse webcam
* Reuse power switch button
* Hook up system fan (for cooling the inverter)
* Add a USB 2.0 hub
* Reuse CF card reader assembly that includes 2 open USB ports
* Reuse ambient LED light
* Add charging through PD
* Reuse touchscreen

Components to be removed or ignored:
* Motherboard + GPU
* CPU & GPU fans
* BT module
* CD drive
* Hard drive
* 1394 connector
* WiFi antennas
* IR led
* Light sensor
* Hot start key

## Disassembly
The first step is to open up the machine and disassemble the parts, reverse engineering some components if needed. I found [these instructions](disassembly.pdf) from HP which helped with the disassembling process.

## Main interface
The main interface with the computer is a USB C connection. I have an USB C -> HDMI adapter with a USB-A port and PD.
* The HDMI port is connected to the video controller via an HDMI cable.
* The USB-A port is connected to a downstream USB 2.0 hub for the peripherals and open USB ports.
* The PD port is powered by a 19V -> PD charging module. This is for laptops that can be charged via its USB C port via PD. All Macbooks can. Some PCs can't.

## Display
The idea is to use a video controller to drive the panel. There's quite a few video controllers out there. I did a lot of research on [this page](https://hackaday.io/project/179868-all-about-laptop-display-reuse) that provides a lot of info on reusing LDC panels, especially laptop panels.

* Model: [CLAA260WU11](https://www.panelook.com/CLAA260WU11_CPT_25.5_LCM_overview_2842.html)
* LVDS (30 pin), 2 channel, 8 bit, CCFL
* Native resolution 1920x1200
* The daughter board provides the DC power (19 volts) for the inverter.
* The inverter connector consists a bunch of Vcc and GND wires, connected directly to the 19V DC power. In addition, there is an EN and a DIM pins, connected to the motherboard. When the motherboard is powered on, I’m measuring ~4.4v on these pins.
* I connected both EN and DIM to +5v DC and the panel lighted up properly. So I rewired them to the inverter output of the video controller. Note that I'm not using the 12v output of the inverter output of the video controller.
* I first ordred a PCB800862 controller. But I couldn’t get it to support 1920x1200. I think its firmware needed to be flashed. Then I ordered an M.MT68676.3 intended to be used for LM240WU2, which has the same resolution and is also 2 ch 8 bit, and it worked.

## Speakers
* There are 2 enclosures, one for the left channel and one for the right.
* Each enclosure contains 2 speakers. They seem to be identical.
* The 4 pins are the input of the 2 speakers.
* The enclosures are passive components, i.e. they contain no amps, since I measured a few ohms across the wires.
* The enclosures are connected to the daughter board, which has amps for the two channels.
* The audio output from the motherboard is fed to the daughter board through 10 wires.
* After some reverse engineering, I have found out the pinout of the output. 
  * The two 5v pins are probably for the current capacity. Both speakers still work if I disconnect either.
  * The audio outputs are single ended, with a bias voltage of ~1.62 volts.
  * Reverse engeineered pinout:

    | Color | Function | Color | Function |
    |-------|----------|-------|----------|
    |Yellow |5v	       |Orange |5v        |
    |Grey   |L+        |White  |R+        |
    |Black  |Shield    |Black  |Shield    |
    |Brown  |GND       |Black  |GND       |
    |Orange |3.3v	     |Red    |GND (wtf?)|

* See some discussion on this [reddit thread](https://www.reddit.com/r/AskElectronics/comments/1d7cm2i/comment/l71as52/?context=3).
* To test the amps, I cut the wires and soldered one half to 2.5mm headers. Then I plugged them to a perf board and provide 5v, 3.3v, and audio signal.
* I took measurements of the current on both 5v and 3.3v. The 3.3v line has a constant current of ~60 μA, apparently not used for driving the speakers.
* The 5v line tops at 60mA when the volume is maxed out, although the nominal wattage of the speakers is 4W (800 mA).
* It was speculated that the 3.3v pin is used for controlling the volume. However, after inspecting it with an oscilloscope, it didn’t seem to be the case. The waveform is a straight-line no matter how I adjust the volume. Instead, the amplitude of the signal changes. So I’m not sure what it is used for. But without it, the amps don’t work. It might be an enable signal?
* Instead of using regulated 3.3v, I’m simply using a voltage divider (100K + 47K) to get 3.3v from 5v, thus eliminating a 3.3v regulator. Although I suspect 5v works too but I don't want to risk it.

## Microphones
* The machine comes with 2 microphones. One left and one right. They are directly connected to the motherboard via a 5 pin connector. Physically, they share the same PCB board with the webcam.
* One of the 5 pins is the shield/ground. So I thought the 4 wires are the output of the analog microphones. However, it doesn’t seem to be the case. When I connect a pair to an oscilloscope I’m not getting any signal. So my guess is that the output is digital, which makes sense. Running mic level analog signals along 30 cm wires inside a computer is probably too noisy.
* Then I found this [doc](https://www.akustica.com/Files/Admin/PDFs/AN40-1%201%20AKU2002CH%20Mic%20Module%20Design%20Guide.pdf) and [this](https://tzjwinfcha.pixnet.net/blog/post/25040549) (in Chinese) which indicate that they are MEMS digital microphones.
* I also looked up the motherboard and found that its audio codec is AD1984, which does support digital microphones.
* Digital microphones are not passive components. I suspected they drew power from the camera’s USB port and the remaining 4 pins were clock and data for both mics. Sure enough, if the camera is not plugged in, the mics don’t work. So that confirmed my theory. That's also why I didn't get reading in the previous step.
* Two pins have square waves whose width is about 500 ns (2 MHz) and amplitude about 3.3v. These are very likely clock pins.

  <img src="IMG_0130.jpeg" width="400">

* The other two pins output irregular square waves. I suspected they were PDM encoded, as it’s a common format (along with I2S). To verify this, I needed to produce a 2 MHz clock signal, hook up a low pass filter on the data pin, and inspect the output. 
* To generate the clock signal, I used PIO on RP2040 and made a small change to [this example](https://github.com/raspberrypi/pico-examples/tree/master/pio/squarewave). Then I connected an earphone. I also didn't use a low pass filter (rather, the earphone is the filter). The output is definitely analog signal and it resembles the input.
* To interface the mics with the computer, I use a RP2040 as a soundcard. I found [this project](https://www.hackster.io/sandeep-mistry/create-a-usb-microphone-with-the-raspberry-pi-pico-cc9bd5) that basically does what I need, except it only works for one channel.
* I found that someone has tried a [mic array](https://github.com/CaydenPierce/MSA). I gave it a try, but the result is too noisy. Not sure what's happening.
* Then I decided to code it myself based on Sandeep's project and it was a success. The changes involved:
  - Read 2 bits (pins 3 and 4) instead of 1 in each clock cycle.
  - Use some bit gymnastics to separate the left and right channels for filtering.
  - Change the USB descriptor to present as a stereo mic.
  - Write data for two channels. And [here](https://github.com/delingren/mems-mic/tree/stereo_no_encoding) is the code. It's hacky and hard coded. If anyone is reading this, don't use it as a library.
  - I am using pin 5 as the clock signal for both mics.
* I did all the development and debugging with two Pi Picos, one as a debug probe. The final product uses an RP2040 board with a smaller foot print with fewer pins, since I only need 3 pins.
* References:
  - [USB microphone with a Pi Pico](https://www.hackster.io/sandeep-mistry/create-a-usb-microphone-with-the-raspberry-pi-pico-cc9bd5)
  - [Pi Pico as a sound card](https://github.com/raspberrypi/pico-playground/tree/master/apps/usb_sound_card)
  - [A more concrete example of using Pi Pico as a sound card](https://www.instructables.com/RP2040-USB-Sound-Card-Pulse-Density-Modulated-Audi/)
  - [PDM filtering](https://www.st.com/resource/en/application_note/an5027-interfacing-pdm-digital-microphones-using-stm32-mcus-and-mpus-stmicroelectronics.pdf)
  - [Audio codec data sheet](https://www.analog.com/media/en/technical-documentation/obsolete-data-sheets/AD1984.pdf)
  - [Debugging Pi Pico](https://datasheets.raspberrypi.com/pico/getting-started-with-pico.pdf?_gl=1*11ism9o*_ga*MTIzODIxMTQ0OC4xNzE5OTU2NzM4*_ga_22FD70LWDS*MTcxOTk1NjczNy4xLjEuMTcxOTk1Njg1OC4wLjAuMA..)
  - [Debug Probe Firmware](https://www.raspberrypi.com/documentation/microcontrollers/raspberry-pi-pico.html#:~:text=You%20can%20use%20one%20Raspberry,→%20SWD%20and%20UART%20converter.)

## Volume Control
I thought it'd be nice to be able to control the volume of the audio output. Apple devices don't support CEC, so they can't control the volume of the audio stream in the HDMI output signal. There're software solutions (e.g. [SoundSource](https://rogueamoeba.com/soundsource/)). But it'd be nice to be able to do it universally. The machine comes with three volume control buttons (up, down, and mute). After doing some research, I found this IC [MAX5486](https://www.analog.com/media/en/technical-documentation/data-sheets/MAX5486.pdf) that fits the bill. So I ordered a couple and some TSSOP24 to 2.5mm pinhead adpater. Soldering those pins was a challenge! It was about the smallest thing I could hand solder. A small iron and a chipsel tip definitely helped a lot.

Now, on to the volumn control buttons. They are mounted on a small PCB board, along with an LED for HDD. Remember those HDD activity indicators back in the days? They were on every computer, including laptops! I doubt anyone ever found them useful. Anyway, the PCB iss connected to the motherboard with a 6 pin connector. I thought it'd be easy to wire up. I thought each button would just short two pins (or a pin to the ground). However, after some poking around, it appeared that there's some resistors and capacitors on the board, probably for debouncing. What's really strange is that only the vol+ button shorts a pin to the ground. The other two have a 20K ohm resistance to the ground when open and 10K when closed. Not sure what the deal is. The HDD LED is somewhat controlled by a BJT on board and requires a +5V power.

Instead of tweaking the circuit, I decided to rewire it such that all three buttons simply short a pin to the ground. MAX5486 does its own debouncing. I also removed the LED and replaced it with my own. I am planning to use it to indicate the mute state. MAX5486 supports indicating the sound level with 5 LEDs. When muted, all LEDs are off. So I just need to wire my LED to the pin for the lowest volume level, but reversing the logic. The LEDs are supposed to be wired between Vdd and the pins. When an LED is supposed to be turned on, the pin goes down to 0V. When an LED is supposed to be turned off, the pin goes up to 2.2V, just enough to turn off the LED (2.8V is below the voltage drop of an LED). But 2.2V to ground is also not enough to drive an LED. So I'm pulling it up to 5V with a 1K resistor and it works.

Here's the final circuit for the volume control module. It takes in 5V DC power and audio input and outputs audio for the amps. It also passes through the 5V DC and generates 3.3V DC for the amps. The bias voltage od MAX5486 is (Vdd+Vss)/2 which is 2.5V in my case. It's higher than the original motherboard. It's probably not ideal but my ears can't distinguish the difference.

![Volume Control Schematic](volume-control.png)

I used a solderable breadboard for the final product and soldered the MCU for the microphones on it as well. Then I realised that I could've used the 3.3v from the MCU dev board for the amp instead of a voltage divider. Oh well.

![Volume Control PCB](IMG_0197.jpeg)

## Webcam
The webcam is a standard USB device, with wires correctly color labled. All I needed to do is solder the wires to a USB plug. So there's nothing too exciting here.

## Ambient LED light
There's a strip of blue LEDs at the bottom of the display, illuminating the keyboard. It's controlled by a single button on the right side panel. There's a PCB with connections to a light sensoring LED and the motherboard. It seems to be a USB device. I suppose HP has a Windows driver that controls the light based on the ambient lighting. I have no use for the light sensoring LED or the USB port. So I simply discarded those cables. When provided with 5V DC power, the LED and the control button work fine. Each time the button is pressed, it cycles through 3 levels of brightness.

## Power source
I'm reusing the 230 Watt, 19 Volt DC power adapter. The connector is 7.4mm OD/5mm ID, used by many Dell and HP laptops. I have a couple of Dell and HP chargers in my drawer. What's more, most old laptop chargers are ~19V and USB PD is 20V. So it's easy to replace the power adapter if it ever breaks.

The video controller uses 12-15 Volt DC. So I can't feed it the input directly. I also need 5 Volts for the amps for the speakers as well as the powered USB hub and ambient LED light. So I'll need a 12V regulator and a 5V regulator. I'm using a Pololu 12V 2.4A step down regulator and a Plolu 5V 3.2A step down regulator. The speakers are 4 Watts (0.8A). My USB hub has 7 ports and I'm plugging a Pi Pico, the webcam, the card reader and the touchscreen. All of them cannot draw more than 1A, if that. So the 5V regulator should provide plenty juice, as long as I don't plugin multiple high power devices into the open USB ports. Not to mention the host computer can also supply 500 mA via its USB port.

TODO: measure actual current of these devices using a USB power meter.

### 5V DC
Physical connections:
* Volume control module, which also powers the amps
* USB Hub, also powering the microphone MCU, the webcam the CD card reader, the touchscreen, and open ports
* Ambient light LED
* System fan (switched)

Regulator: [D36V28F5](https://www.pololu.com/product/3782)

### 12V DC
Physical connection:
* Video controller
Regulator: [D36V28F12](https://www.pololu.com/product/3786)

### PD charging
I found a module on aliexpress that takes DC input and outputs PD.

### Power switch
The original power switch on this machine is a pushbutton that shorts a pin to the ground on the motherboard to turn on the machine. I am connecting the wires to this [electronic switch](https://www.pololu.com/product/2812)

### Indicator LEDs
The switch assembly also has a couple of LEDs, one yellow and one green. I suppose they indicate standby and on states. I was going to reuse the LEDs. However, their anodes are hard wired to the ground and it wouldn't work with the electronic switch, which needs two standalone wires for the pushbutton. So I desoldered the LEDs and hot glued my own SMD LED on the board and soldered its leads to the connector. All in all, this module has 4 outgoing wires: 2 for the pushbutton and 2 for the LED. The LED is driven by the switched 19V DC with a 470 Ohm resistor. TODO: 5V DC with a ~60 Ohm resistor.

## System fan
Use a solid state relay or a MOSFET to control the fan. The relay is controlled by the EN pin of the inverter.

## USB 2.0 Hub
Here are all the USB ports I need
* Pi Pico (for the microphones)
* Webcam
* CD card reader and 2 open ports 
* 3 open ports (originally on the motherboard)
* Touch screen

So, the tally is 9. I ordered a 7 port hub circuit from aliexpress. It has its own 5V DC power. I am also going to connect another 4-port USB hub to it. The second USB hub will be physically installed at the location of the motherboard, expsing 3 ports to the outside through the cutout of the panel.
