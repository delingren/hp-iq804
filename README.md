# HP Touchsmart IQ804 all-in-one PC conversion project

## Context
I found this all-in-one PC at a PC recycle bin long ago (circa 2014). It was used as a guest computer in my house for a while. Now it's way too old even for that purpose. HP Touchsmart IQ804 (marketed as IQ800) is an all-in-one PC released in 2008. It was originally equipped with Windows Vista and is upgradable to Windows 7. I never tried but I doubt it'd be able to run Windows 10. There's no way it'd be able to run Windows 11. Instead of throwing it away, I decided to harvest as many components as I can. In particular, the 26 inch, 1920x1200 LCD panel is crispy clear and I really love it.

Disclaimer: as a professional software engineer, I am not well versed in electronics beyond the basics, especially analog. So I'm not really sure if what I did was correct or optimal. But things did work. If anyone reading this finds any mistakes, please leave a comment.

## Goal
Convert this all-in-one computer into an external display and docking station, reusing as many components as possible.

Primary goals: 
* Reuse the display
* Reuse the speakers

Secondary goals:
* Reuse the microphones
* Reuse the volume control buttons
* Reuse the webcam
* Reuse the power switch button
* Hook up the system fan (for cooling the inverter)
* Add a USB 2.0 hub and connect it to the 5 open USB ports
* Reuse the SD card reader assembly
* Reuse ambient LED light
* Add charging through PD
* Reuse touchscreen (? I don't really like touchscreens on computers)

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
As a design goal, the main interface with the computer would be a single USB C connection. I have a USB-C to HDMI adapter with a downstream USB-A port and PD for charging the host.
* The HDMI port would be connected to the video controller via an HDMI cable.
* The USB-A port would be connected to a downstream USB 2.0 hub for the peripherals and open USB ports.
* Optionally, the PD port would be powered by a 19V -> PD charging module. This is for laptops that can be charged via its USB C port via PD. All Macbooks can. Some PCs can't.

## Display
The idea is to use a video controller to drive the panel. There's quite a few video controllers out there. I did a lot of research on [this page](https://hackaday.io/project/179868-all-about-laptop-display-reuse) that provides a lot of info on reusing LDC panels, especially laptop panels. I have also reused a couple of LCD panels harvested from end of life laptops.

Here's a summary of the LCD panel.
* Model: [CLAA260WU11](https://www.panelook.com/CLAA260WU11_CPT_25.5_LCM_overview_2842.html)
* Interface: LVDS (30 pin), 2 channel, 8 bit
* Backlight: CCFL
* Native resolution: 1920x1200

The daughter board provides the DC power (19 Volts) for the inverter. I am keeping it this way. The inverter connector consists a bunch of Vcc and GND wires, connected directly to the 19V DC power. In addition, there is an EN and a DIM pins, connected to the motherboard. When the motherboard is powered on, I’m measuring ~4.4V on these pins.

I connected both EN and DIM to +5V and the panel lighted up properly. So I rewired them to the inverter output of the video controller. Note that I'm not using the 12V output of the inverter output of the video controller, but rather using its originally DC power.

For the video controller, I first ordred a PCB800862 controller. But I couldn’t get it to support 1920x1200. I think its firmware needed to be flashed. Then I ordered an M.MT68676.3 intended to be used for LM240WU2, which has the same resolution and is also 2 ch 8 bit, and it worked.

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
    |Black  |Shield/GND|Black  |Shield/GND|
    |Brown  |GND       |Black  |GND       |
    |Orange |3.3v	     |Red    |GND (wtf?)|

* See some discussion on this [reddit thread](https://www.reddit.com/r/AskElectronics/comments/1d7cm2i/comment/l71as52/?context=3).
* To test the amps, I cut the wires and soldered one half to 2.5mm headers. Then I plugged them to a perf board and provide 5v, 3.3v, and audio signal.
* I took measurements of the current on both 5v and 3.3v. The 3.3v line has a constant current of ~60 μA, apparently not used for driving the speakers.
* The 5v line tops at 60mA when the volume is maxed out, although the nominal wattage of the speakers is 4W (800 mA).
* It was speculated that the 3.3v pin is used for controlling the volume. However, after inspecting it with an oscilloscope, it didn’t seem to be the case. The waveform is a straight-line no matter how I adjust the volume. Instead, the amplitude of the signal changes. So I’m not sure what it is used for. But without it, the amps don’t work. It might be an ENABLE signal? My suspicion is that the motherboard would pull it low if earphones are plugged into the audio jack. I have no use for it so I'm going to fix it high.
* Instead of using regulated 3.3v, I’m simply using a voltage divider (100K + 47K) to get ~3.3v from 5v, thus eliminating a 3.3v regulator. Although I suspect 5V works too but I don't want to risk it.

## Microphones
The machine comes with 2 microphones. One left and one right. They are directly connected to the motherboard via a 5 pin connector. 

### Identification
The mics physically share the same PCB as the camera. The PCB has two clusters of wires coming out. One for the camera and one for the mics. The clusters have labels. The camera cluster has 4 wires: black, red, green, and white. They are connected to some pin headers on the motherboard. It looks like a USB connection to me. So I soldered the wires to a USB-A plug and plugged it into a laptop and it worked fine.

Now the mics. I originally thought they were passive analog mics. The markings on the microphone is "AKU2004". I was not able to find too much information, other than that they might be digital microphones, designed for laptops. It makes sense that they are digital mics. It'd be too noisy to run 30 cm wires for mic level signals inside a computer.

I also looked up the motherboard and found that its audio codec is AD1984, which does support digital microphones. I booted the computer and inspected the pins. It turned out that 2 pins had perfect 2 MHz square waves and the other two had irregular sqare waves. So that confirmed that they are digital and the 2 MHz sqare waves must be clock signals. I had no idea about digital microphones. But some research on Google led me to believe they are MEMS microphones and the output is probably PDM (pulse density modulation) format. There are two popular formats. The other is I2S. PDM signals, once filtered with a 20 kHz low pass filter, should produce just analog signal. So I did just that and connected the signals to earphones. Sure enough, the output is audible. So that confirmed that the signal is PDM. The red wires are for the right microphone, BTW.

  <img src="IMG_0130.jpeg" width="400">

I also found this [doc](https://www.akustica.com/Files/Admin/PDFs/AN40-1%201%20AKU2002CH%20Mic%20Module%20Design%20Guide.pdf) and [this](https://tzjwinfcha.pixnet.net/blog/post/25040549) (in Chinese) which indicate that they are MEMS digital microphones.

Digital microphones are not passive components. I suspected they drew power from the camera’s USB port and the remaining 4 pins were clock and data for both mics. Sure enough, if the camera is not plugged in, the mics don’t work. So that confirmed my theory. That's also why I didn't get reading in the previous step.

The next step is to interface the PDM signals with USB.

### Sampling and Decoding
There are at least two ways to do this. The easiest is to low pass filter the output and convert it to analog, then feed it to a USB sound card. This should be super easy. But what's the fun in that?! Plus, I'm an engineer. Let's over engineer the hell out of it!

To generate the clock signal, I used PIO on RP2040 and made a small change to [this example](https://github.com/raspberrypi/pico-examples/tree/master/pio/squarewave). Then I connected an earphone. I also didn't use a low pass filter (rather, the earphone is the filter). The output is definitely analog signal and it resembles the input.

So, now I am trying to directly convert the PDM digital signal into USB data frames. Some searching led me to the first [important discovery](https://www.hackster.io/sandeep-mistry/create-a-usb-microphone-with-the-raspberry-pi-pico-cc9bd5) that converts a mono microphone into a USB microphone using a Pi Pico. I tried it and it worked out of the box! I could hook up with one mic and call it good there. But I wanted to make use of both of the mics. I can either average the input of both or create a stereo mic. I suspect the latter is probably simpler.

### USB Interface
Now, to do that, I need to modify the code. First, I need to present the Pico as a stereo mic. The project aforementioned uses tinyusb library to turn the Pico into a USB audio device. So I started looking into USB audio. Concepcts I learned along the way.

* Device Descriptors
* Configuration Descriptors
* Interface Descriptors
* Endpoint Descriptors
* Isochronous Transfer

### Sampling both channels
In the hackster blog post aforementioned, the author Sandeep uses Pico's PIO to generate the clock signal at 1.024 MHz and sample the data pin once per clock cycle, or 16 times per milliesecond (16,000 kHz), then uses a soft PDM filter to down sample, filter, and produce 16 amplitude values, each a 16 bit signed integer. To do this for 2 mics, I

* Read 2 bits from two *consecutive* pins on a PIO statemachine.
* Changed PDM filter logic to separate left and right signals.
* Change PDM filter logic to filter both signals. The filter is stateful and you can't call the filter function on both signals alternatively.
* Generate output for both channels. The 16-bit signed integer values representing the amplitude are alternating. I.e. one value for the left followed by one value for the right and repeat.

I found that someone has tried a [mic array](https://github.com/CaydenPierce/MSA). I gave it a try, but the result is too noisy. Not sure what's happening. Then I decided to code it myself based on Sandeep's project and it was a success. The changes involved:
* Read 2 bits (pins 3 and 4) instead of 1 in each clock cycle.
* Use some bit gymnastics to separate the left and right channels for filtering.
* Change the USB descriptor to present as a stereo mic.
* Write data for two channels. And [here](https://github.com/delingren/mems-mic/tree/stereo_no_encoding) is the code. It's hacky and hard coded. If anyone is reading this, don't use it as a library.
* I am using pin 5 as the clock signal for both mics.

I did all the development and debugging with two Pi Picos, one as a debug probe. The final product uses an RP2040 board with a smaller foot print with fewer pins, since I only need 3 pins.

### References:
* [4-channel mic example](https://github.com/hathach/tinyusb/blob/master/examples/device/audio_4_channel_mic/src/usb_descriptors.c)
* [USB descriptors](https://www.beyondlogic.org/usbnutshell/usb5.shtml)
* [A USB audio tutorial](https://www.silabs.com/documents/public/application-notes/AN295.pdf)
* [USB microphone with a Pi Pico](https://www.hackster.io/sandeep-mistry/create-a-usb-microphone-with-the-raspberry-pi-pico-cc9bd5)
* [Pi Pico as a sound card](https://github.com/raspberrypi/pico-playground/tree/master/apps/usb_sound_card)
* [A more concrete example of using Pi Pico as a sound card](https://www.instructables.com/RP2040-USB-Sound-Card-Pulse-Density-Modulated-Audi/)
* [PDM filtering](https://www.st.com/resource/en/application_note/an5027-interfacing-pdm-digital-microphones-using-stm32-mcus-and-mpus-stmicroelectronics.pdf)
* [Audio codec data sheet](https://www.analog.com/media/en/technical-documentation/obsolete-data-sheets/AD1984.pdf)
* [Debugging Pi Pico](https://datasheets.raspberrypi.com/pico/getting-started-with-pico.pdf?_gl=1*11ism9o*_ga*MTIzODIxMTQ0OC4xNzE5OTU2NzM4*_ga_22FD70LWDS*MTcxOTk1NjczNy4xLjEuMTcxOTk1Njg1OC4wLjAuMA..)
* [Debug Probe Firmware](https://www.raspberrypi.com/documentation/microcontrollers/raspberry-pi-pico.html#:~:text=You%20can%20use%20one%20Raspberry,→%20SWD%20and%20UART%20converter.)
* [Final code](https://github.com/delingren/mems-mic/tree/stereo_no_encoding)

## Volume Control
I thought it'd be nice to be able to control the volume of the audio output. Apple devices don't support CEC, so they can't control the volume of the audio stream in the HDMI output signal. There're software solutions (e.g. [SoundSource](https://rogueamoeba.com/soundsource/)). But it'd be nice to be able to do it universally. The machine comes with three volume control buttons (up, down, and mute). After doing some research, I found this IC [MAX5486](https://www.analog.com/media/en/technical-documentation/data-sheets/MAX5486.pdf) that fits the bill. So I ordered a couple and some TSSOP24 to 2.5mm pinhead adpater. Soldering those pins was a challenge! It was about the smallest thing I could hand solder. A small iron and a chipsel tip definitely helped a lot.

Now, on to the volumn control buttons. They are mounted on a small PCB board, along with an LED for HDD. Remember those HDD activity indicators back in the days? They were on every computer, including laptops! I doubt anyone ever found them useful. Anyway, the PCB iss connected to the motherboard with a 6 pin connector. I thought it'd be easy to wire up. I thought each button would just short two pins (or a pin to the ground). However, after some poking around, it appeared that there's some resistors and capacitors on the board, probably for debouncing. What's really strange is that only the vol+ button shorts a pin to the ground. The other two have a 20K ohm resistance to the ground when open and 10K when closed. Not sure what the deal is. The HDD LED is somewhat controlled by a BJT on board and requires a +5V power.

Instead of tweaking the circuit, I decided to rewire it such that all three buttons simply short a pin to the ground. MAX5486 does its own debouncing. I also removed the LED and replaced it with my own. I am planning to use it to indicate the mute state. MAX5486 supports indicating the sound level with 5 LEDs. When muted, all LEDs are off. So I just need to wire my LED to the pin for the lowest volume level, but reversing the logic. The LEDs are supposed to be wired between Vdd and the pins. When an LED is supposed to be turned on, the pin goes down to 0V. When an LED is supposed to be turned off, the pin goes up to 2.2V, just enough to turn off the LED (2.8V is below the voltage drop of an LED). But 2.2V to ground is also not enough to drive an LED. So I'm pulling it up to 5V with a 1K resistor and it works.

Here's the final circuit for the volume control module. It takes in 5V DC power and audio input and outputs audio for the amps. It also passes through the 5V DC and generates 3.3V ENABLE signal for the amps. 

![Volume Control Schematic](volume-control.png)

As it turned out, my schematic is slightly different from the recommended one in the datasheet. The audio output from the video controller is single ended instead of differential. I.e. one wire of each channel is tied to the ground. The recommended circuit connects it to BIAS, which worked perfectly fine in prototyping and testing, because I was using the output from a cell phone for testing, whose signals are effectively differential, since the phone and the volume control module don't share a ground. Once I wired up the video controller and the volume control module, and powered them with 12V and 5V DC, things started going haywire. The audio was distorted and crackling. I did a lot of head scratching and finally realized that I was effectively shorting BIAS to the ground, which is now shared.

I used a solderable breadboard for the final product and soldered the MCU for the microphones on it as well. Then I realised that I could've used the 3.3v from the MCU dev board for the amp instead of a voltage divider. Oh well.

![Volume Control PCB](IMG_0197.jpeg)

Pinout:
* Upper row: 3.3V, 5V, 5V, GND, GND, GND
* Lower row left: Vol-, Vol+, Mute, GND, LED-, LED+
* Lower row right: GND, Mic CLK, Mic R DAT, Mic L DAT

## Webcam
The webcam is a standard USB device, with wires correctly color labled. All I needed to do is to solder the wires to a USB connector. So there's nothing too exciting here.

## Ambient LED light
There's a strip of blue LEDs at the bottom of the display, illuminating the keyboard. It's controlled by a single button on the right side panel. There's a PCB with connections to a light sensoring LED and the motherboard. It seems to be a USB device. I suppose HP has a Windows driver that controls the light based on the ambient lighting. I have no use for the light sensoring LED or the USB port. So I simply discarded those cables and parts. When provided with 5V DC power, the LED and the control button work fine. Each time the button is pressed, it cycles through 3 levels of brightness.

## System fan
The original machine has a 12V 0.4A system fan that blows into the LCD panel inverter, in addition to two fans cooling the CPU and GPU. I don't have data on the inverter but a typical wattage of a CCFL inverter is ~5 Watts. I suppose running it at 5 Volts should be sufficient to dissipate the heat. I also want to turn it on only when the inverter is turned on. Therefore I am piggy-backing on the inverter ENABLE signal of the video controller.

I have two options to control the fan: MOSFET or relay. MOSFET is quieter and low profile. So I'm inclined to use a MOSFET. There are a few issues to address though.

* The ENABLE signal is 5 Volts. So I need a logic level MOSFET.
* 2N7000, a typical logic level MOSFET, has a resistance of ~5 Ohm when open. This is probably significant for a small motor.
* The inrush current when a motor starts is significantly larger than its running current. I need to make sure the current doesn't exceed the current rating of the MOSFET.

I have no means to measure the peak current. But 3x running current is a rule of thumb for estimating the inrush current. So I connected the fan to a 5-Volt source, in series with a 5.6 Ohm resistor, simulating the resistance of the MOSFET, and measured its running current. It measured ~65 mA. If we go by the 3x rule, the peak would be exactly at the 200 mA current rating of 2N7000, cutting too close. So I decided to put two 2N7000s in parallel, which should reduce the resistance as well.

![Fan Control Circuit](fan-control.png)

## USB 2.0 Hub
Here are all the USB ports I need
* Pi Pico, internal, for the microphones
* Webcam, internal,
* CF card reader, internal
* 5 open ports on the IO panel (I just need to pass through the ports to the hub)
* Touch screen

So, the tally is 9. I ordered a 7 port hub circuit from aliexpress. It has its own 5V DC power. I am also going to connect another 4-port USB hub to it.

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
The original power switch on this machine is a pushbutton that shorts a pin to the ground on the motherboard to turn on the machine. I am hooking it up with this [electronic switch](https://www.pololu.com/product/2812).

### Indicator LEDs
The switch assembly also has a couple of LEDs, one yellow and one green. I suppose they are for standby and on states. I was going to reuse the LEDs. However, their anodes are hard wired to the ground and it wouldn't work with the electronic switch, which needs two standalone wires, instead of ground, for the pushbutton. So I desoldered the LEDs and hot glued my own SMD LED on the board and soldered its leads to the connector. All in all, this module has 4 outgoing wires: 2 for the pushbutton and 2 for the LED. The LED is connecte to the output of the 5V regulator via a 56 Ohm resistor.