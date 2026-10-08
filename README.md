# Extension of the T4.x USB audio input and output. #
This suite of files enables 2-, 4-, 6- or 8-channel USB I/O to be used with the Teensy 4.x Audio library. It was originally developed based on Teensyduino 1.59, and the core files have since been brought into sync with Teensyduino 1.60. No representations are made as to whether it might work starting from older versions!

## Installation ##
Until such time as these changes are merged into the Teensyduino release package, installation is a manual process. These instructions assume you are working from an unmodified installation - if you have already modified any of the files involved you will need to figure out how to merge these changes yourself.

**It is strongly recommended that you keep safety copies of any files that you overwrite or delete, so that if something goes wrong you can go back to a working installation.**

### Core files ###
These provide the multi-channel USB audio capability.
* Locate the core files folder, e.g. for Arduino 1.8.19 on Windows `C:\Program Files (x86)\Arduino\hardware\teensy\avr\cores\teensy4`
* Copy the contents of the `changedCorefiles` folder into the cores folder, overwriting as necessary

### GUI files ###
These provide the Design Tool with ability to create an audio design using multi-channel USB objects. 
* Locate the audio library GUI folder, e.g. for Arduino 1.8.19 on Windows `C:\Program Files (x86)\Arduino\hardware\teensy\avr\libraries\Audio\gui`
* Copy the contents of the `changedGUI` folder into the GUI folder, overwriting as necessary

### Config files ###
These provide the Arduino IDE with extra entries in the Tools menu, allowing selection of the USB channel count. 
* Locate the config files folder, e.g. for Arduino 1.8.19 on Windows `C:\Program Files (x86)\Arduino\hardware\teensy\avr`
* If you have no `boards.local.txt`
  * simply copy this in from the `changedConfigfiles` folder
* else 
  * merge the contents of `changedConfigfiles/boards.local.txt` into your existing file
* Make a safety copy of `platform.txt`, then delete the one in the config folder
* If you are using Arduino IDE 1.x (e.g. 1.8.19)
  * copy `changedConfigfiles/TD-platform.txt` into the config folder, and rename to `platform.txt`
* else (you are using Arduino IDE 2.x)
  * copy `changedConfigfiles/BM-platform.txt` into the config folder, and rename to `platform.txt`
  * ensure the IDE is *not* running
  * find the Arduino 2.x cache folder, e.g. `C:\Users\<user name>\AppData\Roaming\arduino-ide\` and delete it

## In use ##
* In the updated Design Tool, place one usb, usb_quad, usb_hex or usb_oct input and/or output object(s) in your design, and export as usual
* In the Tools menu, ensure you have configured the `USB Type` to include Audio, and select the correct number of `USB channels`
* If your sketch uses a USB I/O object that is wider than configured, you will get a compile-time error - usually *many* errors
* Only **one** USB input object (`AudioInputUSB`, `...Quad`, `...Hex`, `...Oct`) and **one** USB output object (`AudioOutputUSB`, ...) may be used in a sketch. Their ring buffers and USB state are static, so a second object would share and corrupt them.
* Windows is very bad at detecting changes to the USB descriptor: see the examples for how to use the `set_usb_string_serial_number.h` file to change the serial number according to the channel count and sample rate, which seems to force Windows to recognise the changes
* In addition to the USB channel count, the Tools menu also has options to
  * set the sample rate to 44.1kHz, 48kHz or 96kHz: these seem to work OK, but may not be supported by all audio hardware
  * set the audio block sample count to 128 (normal), 16, or 256 samples. This is highly experimental, and many audio objects work badly if the sample count is changed. It is hoped that future revisions of the Audio library will be more resilient to changing this parameter, which will be of use for (a) low-latency applications using 16-sample blocks, or (b) keeping the audio interrupt rate reasonable at 96kHz  sample rate by using 256-sample blocks

## Examples ##  
Examples can be found in `src/main_usbOutput.ino` and `src/main_usbInput.ino` (and `src/USBmultiChannelTest/USBmultiChannelTest.ino`)

## Technical details ## 
Main features are:
- Switched from UAC1 to UAC2 standard.
- 8 channels can be streamed from and to the USB host. (Change `USB_AUDIO_NO_CHANNELS_480` in `usb_desc.h` if you can't get the updated Tools options to work. For more than 8 channels one need to extend `CHANNEL_CONFIG_480` and the feature unit descriptor in `usb_desc.c`)
- 16 or 24bit pcm audio can be streamed. (Change `AUDIO_SUBSLOT_SIZE` in `usb_desc.h` if needed. `AUDIO_SUBSLOT_SIZE 2` means each sample is 2bytes large, `AUDIO_SUBSLOT_SIZE 3` changes the sample size to 3bytes/24bit)
- Feedback of the usb input to the host improved in order to prevent buffer under- and overruns
- USB output is able to duplicate or skip single samples in order to prevent buffer under- and overruns. This is not a perfect solution, but is an improvement to the current implementation.
- USB input: Parameters of the PI controller that computes the feedback can optionally be set at the constructor.
- USB input and output: The target number of buffered samples can be configured. (Can e.g. be increased if buffer under- or overruns occur.)
- USB input and output provide information about their status (getStatus) like if and how many buffer over- and under-runs occurred.
- Volume and mute set by the host (UAC2 feature unit): the volume range reported to the host is `FEATURE_VOLUME_MIN_DB` (default -60 dB) to 0 dB in 0.5 dB steps (UAC2 uses units of 1/256 dB). `AudioInputUSB::volume()` returns the corresponding linear gain (0.0 - 1.0, 0 if muted). `USBAudioInInterface::features.volume_db256` holds the value in 1/256 dB, `features.volume` the linear gain scaled to 0 - 255.
- 24 bit audio from the host is rounded to the 16 bit samples of the audio library. Define `AUDIO_USB_RX_DITHER` to add TPDF dither before rounding.
- High speed feedback: by default the feedback value is sent in samples per polling interval (16.16), which works with Windows, macOS and Linux. Define `AUDIO_FEEDBACK_PER_MICROFRAME` to send it in samples per microframe as described in USB 2.0, section 5.12.4.2. (Both are identical if the polling interval is one microframe.)

Tested with Teensyduino 1.59 + Arduino IDE being 1.8.19 and Visual Studio Code + Platformio/Teensy platform version 5.0.0
