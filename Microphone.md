# Microphone
## Identification
The mics physically share the same PCB as the camera. The PCB has two clusters of wires coming out. One for the camera and one for the mics. The clusters have labels. The camera cluster has 4 wires: black, red, green, and white. They are connected to some pin headers on the motherboard. It looks like a USB connection to me. So I soldered the wires to a USB-A plug and plugged it into a laptop and it worked fine.

Now the mics. I originally thought they were passive analog mics. The markings on the microphone is "AKU2004". I was not able to find too much information, other than that they might be digital microphones, designed for laptops. I booted the computer and inspected the pins. It turned out that 2 pins had perfect 2 MHz square waves and the other two had irregular sqare waves. So that confirmed that they are digital and the 2 MHz sqare waves must be clock signals. I had no idea about digital microphones. But some research on Google led me to believe they are MEMS microphones and the output is probably PDM (pulse density modulation) format. There are two popular formats. The other is I2S. PDM signals, once filtered with a 20 kHz low pass filter, should produce just analog signal. So I did just that and connected the signals to earphones. Sure enough, the output is audible. So that confirmed that the signal is PDM. The red wires are for the right microphone, BTW.

The next step is to interface the PDM signals with USB.

## Sampling and Decoding
There are at least two ways to do this. The easiest is to filter the output and convert it to analog, then feed it to a USB sound card. This should be easy. But what's the fun in that?! Plus, I'm an engineer. Let's over engineer the hell out of it!

So, now I am trying to directly convert the PDM digital signal into USB data frames. Some searching led me to the first [important discovery](https://www.hackster.io/sandeep-mistry/create-a-usb-microphone-with-the-raspberry-pi-pico-cc9bd5) that converts a momo microphone into a USB microphone using a Pi Pico. I tried it and it worked out of the box! I could hook up with one mic and call it good there. But I wanted to make use of both of the mics. I can either average the input of both or create a stereo mic. I suspect the latter is probably simpler.

## USB Interface
Now, to do that, I need to modify the code. First, I need to present the Pico as a stereo mic. The project aforementioned uses tinyusb library to turn the Pico into a USB audio device. So I started looking into USB audio. Concepcts I learned along the way.

Device Descriptors
Configuration Descriptors
Interface Descriptors
Endpoint Descriptors
Isochronous Transfer

References:
https://www.beyondlogic.org/usbnutshell/usb5.shtml
https://www.silabs.com/documents/public/application-notes/AN295.pdf