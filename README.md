# audio-reactive-led-strip
Real-time LED strip music visualization using Python and the ESP8266 or Raspberry Pi.

![block diagram](images/block-diagram.png)

![overview](images/description-cropped.gif)

# Demo (click gif for video)

[![visualizer demo](images/scroll-effect-demo.gif)](https://www.youtube.com/watch?v=HNtM7jH5GXgD)

# Overview
The repository includes everything needed to build an LED strip music visualizer (excluding hardware):

- Python visualization code, which includes code for:
  - Recording audio with a microphone ([microphone.py](python/microphone.py))
  - Digital signal processing ([dsp.py](python/dsp.py))
  - Constructing 1D visualizations ([visualization.py](python/visualization.py))
  - Sending pixel information to the ESP8266 over WiFi ([led.py](python/led.py))
  - Configuration and settings ([config.py](python/config.py))

# What do I need to make one?
## Computer + ESP8266
To build a visualizer using a computer and ESP8266, you will need:
- Computer with Python 2.7 or 3.5 ([Anaconda](https://www.continuum.io/downloads) is recommended on Windows)
- ESP8266 module with RX1 pin exposed. These modules can be purchased for as little as $5 USD. These modules are known to be compatible, but many others will work too:
  - NodeMCU v3
  - Adafruit HUZZAH
  - Adafruit Feather HUZZAH
- WS2812B LED strip (such as Adafruit Neopixels). These can be purchased for as little as $5-15 USD per meter.

Limitations when using a computer + ESP8266:
- The communication protocol between the computer and ESP8266 currently supports a maximum of 256 LEDs.

# Installation for Computer + ESP8266
## Python Dependencies
Visualization code is compatible with Python 2.7 or 3.5. A few Python dependencies must also be installed:
- Numpy
- Scipy (for digital signal processing)
- PyQtGraph (for GUI visualization)
- PyAudio (for recording audio with microphone)

On Windows machines, the use of [Anaconda](https://www.continuum.io/downloads) is **highly recommended**. Anaconda simplifies the installation of Python dependencies, which is sometimes difficult on Windows.

If `pip` is not found try using `python -m pip install` instead.

## Hardware Connections
The ESP8266 has hardware support for [I²S](https://en.wikipedia.org/wiki/I%C2%B2S) and this peripheral is used <!-- by the [ws2812b i2s library](https://github.com/JoDaNl/esp8266_ws2812_i2s)  -->to control the ws2812b LED strip. This signficantly improves performance compared to bit-banging the IO pin. Unfortunately, this means that the LED strip **must** be connected to the RX1 pin, which is not accessible in some ESP8266 modules (such as the ESP-01).

The RX1 pin on the ESP8266 module should be connected to the data input pin of the ws2812b LED strip (often labelled DIN or D0).

For the NodeMCU v3 and Adafruit Feather HUZZAH, the location of the RX1 pin is shown in the images below. Many other modules also expose the RX1 pin.

![nodemcu-pinout](images/NodeMCUv3-small.png)
![feather-huzzah-pinout](images/FeatherHuzzah-small.png)

# Setup and Configuration
1. Install Python and Python dependencies
2. In [config.py](python/config.py):
  - Set `N_PIXELS` to the number of LEDs in your LED strip (must match `NUM_LEDS` in [ws2812_controller.ino](arduino/ws2812_controller/ws2812_controller.ino))
  - Set `UDP_IP` to the IP address of your ESP8266 (must match `ip` in [ws2812_controller.ino](arduino/ws2812_controller/ws2812_controller.ino))
  - If needed, set `MIC_RATE` to your microphone sampling rate in Hz. Most of the time you will not need to change this.

# Installation for Raspberry Pi
If you encounter any problems running the visualization on a Raspberry Pi, please [open a new issue](https://github.com/scottlawsonbc/audio-reactive-led-strip/issues). Also, please consider opening an issue if you have any questions or suggestions for improving the installation process.

## Installing the Python dependencies
Install python dependencies using apt-get
```
sudo apt-get update
sudo apt-get install python-numpy python-scipy python-pyaudio
```

## Audio device configuration
For the Raspberry Pi, a USB audio device needs to be configured as the default audio device.

Create/edit `/etc/asound.conf`
```
sudo nano /etc/asound.conf
```
Set the file to the following text
```
pcm.!default {
    type hw
    card 1
}
ctl.!default {
    type hw
    card 1
}
```

Next, set the USB device to as the default device by editing `/usr/share/alsa/alsa.conf`
```
sudo nano /usr/share/alsa/alsa.conf:
```
Change
```
defaults.ctl.card 0
defaults.pcm.card 0
```
To
```
defaults.ctl.card 1
defaults.pcm.card 1
```

# Audio Input
The visualization program streams audio from the default audio input device (set by the operating system). Windows users can change the audio input device by [following these instructions](http://blogs.creighton.edu/bluecast/tips-and-tricks/set-the-default-microphone-and-adjust-the-input-volume-in-windows-7/).

Examples of typical audio sources:
* Audio cable connected to the audio input jack (requires USB sound card on Raspberry Pi)
* Webcam microphone, headset, studio recording microphone, etc

## Virtual Audio Source
You can use a "virtual audio device" to transfer audio playback from one application to another. This means that you can play music on your computer and connect the playback directly into the visualization program.

![audio-input-sources](images/audio-source.png)

### Windows
On Windows, you can use "Stereo Mix" to copy the audio output stream into the audio input. Stereo Mix is only support on certain audio chipsets. If your chipset does not support Stereo Mix, you can use a third-party application such as [Voicemeeter](http://vb-audio.pagesperso-orange.fr/Voicemeeter/).

![show-stereomix](images/stereo-show.png)

Go to recording devices under Windows Sound settings (Control Panel -> Sound). In the right-click menu, select "Show Disabled Devices".

![enable-stereomix](images/stereo-enable.png)

Enable Stereo Mix and set it as the default device. Your audio playback should now be used as the audio input source for the visualization program. If your audio chipset does not support Stereo Mix then it will not appear in the list.

### Linux
Linux users can use [Jack Audio](http://jackaudio.org/) to create a virtual audio device.

### OSX
On OSX, [Loopback](https://www.rogueamoeba.com/loopback/) can be use to create a virtual audio device.

# Running the Visualization
Once everything has been configured, run [visualization.py](python/visualization.py) to start the visualization. The visualization will automatically use your default recording device (microphone) as the audio input.

If you encounter any issues or have questions about this project, feel free to [open a new issue](https://github.com/scottlawsonbc/audio-reactive-led-strip/issues).

# Limitations
The visualization code currently supports up to 256 LEDs. Support for additional LEDs will be added in the near future.

# License
This project was developed by Scott Lawson and is released under the MIT License.
