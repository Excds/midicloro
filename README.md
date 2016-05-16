# MIDIcloro - MIDI clock generator and router
By David Ramström

## Introduction
MIDIcloro does the following:
* Listens to MIDI data from up to 4 USB input devices (sequencers, keyboards, controllers) and merges it into 1 output.
* Adds MIDI clock, polyphonic chords, velocity and routing of channels to the MIDI data sent to the output.
* Settings can be controlled in real-time via MIDI CC sent from the input devices.
* Runs stand-alone on the Raspberry Pi, without the need of a mouse, keyboard or monitor. Start-up script included.
* Plain old MIDI connectors (5-pin DIN) are supported by using a USB MIDI interface.


## Example use-cases
Sequencer/keyboard rig - Add clock, chords, velocity and channel routing to the data sent from a MIDI sequencer. The settings are controlled via knobs on the sequencer. Connect a USB MIDI keyboard as well to mix notes from the sequencer with some live jamming on the keys.

Gameboy as sequencer - Same scenario as above, but using a Nintendo Gameboy as MIDI sequencer. Translation of Gameboy link port data to MIDI needs to be handled before sent to MIDIcloro, i.e. you need: Gameboy, Arduinoboy, Raspberry Pi and a USB MIDI interface/cable.

Clock only - Use MIDIcloro as a master clock. You can connect a MIDI controller to set the tempo with a knob.


## MIDI clock
MIDI clock messages are sent immediately after starting MIDIcloro. An incoming start message resets the clock and the start message is forwarded to the output. Stop messages are also forwarded to the output. Clock signals are sent even after a stop message is received, so that connected equipment relying on MIDI clock can still be played using a keyboard. Any incoming clock messages are filtered out. If MIDI clock is disabled, no clock messages are generated and all incoming start, stop and clock messages are forwarded to the output.

There are two ways of setting the clock tempo. It can be set directly using the configured tempo MIDI CC message: new tempo = configured offset + MIDI CC value (0-127). The tempo can also be set by tapping the tempo MIDI CC message every beat. The first tap will set the tempo directly, and the each of the following taps will recalculate and set the tempo according to the tap interval.


## Chord mode
Each device and MIDI channel has its individual chord mode setting. By setting a chord mode, every incoming note will generate other notes, creating a chord. Chord mode is set via the chord mode MIDI CC. The CC value range 0-127 is divided into intervals of 8 to set the following modes:
* OFF
* MINOR3
* MAJOR3
* MINOR3_LO
* MAJOR3_LO
* MINOR2
* MAJOR2
* M7
* MAJ7
* M9
* MAJ9
* SUS4
* POWER2
* POWER3
* OCTAVE2
* OCTAVE3


## Channel routing
Each device and MIDI channel also has its own routing setting. By changing the channel routing, the current channel will be changed to another of the 16 MIDI channels. Notes, CC messages and other MIDI data with channel will be routed to the new target channel. The routing is set to the current channel by default, and is controlled via the configured routing MIDI CC message. The MIDI CC value range (0-127) is divided into intervals of 8, where 0-7 sets channel 1, 8-15 channel 2 and so on.


## Velocity mode
Each device and MIDI channel has a velocity setting set via MIDI CC as follows: CC value=0 => OFF (velocity from the input device is unchanged), CC value=1-126 (all notes get velocity=value), CC value=127 (toggles random velocity mode on/off).
Random velocity mode: all notes get a random velocity between the latest CC value and a configurable offset.
Velocity is scaled to squeeze the whole 0-127 range in between CC value 8-120.
The velocity setting is mirrored to all other input devices with a number higher than the current device. The mirroring can be turned off in the settings.


## Installation
Install dependencies:

`sudo apt-get install libasound2-dev`

`sudo apt-get install libboost-system-dev`

`sudo apt-get install libboost-program-options-dev`

`sudo apt-get install libboost-regex-dev`

Download the latest binary and make it executable:

wget https://github.com/ledfyr/midicloro/releases/download/v1.4/midicloro

`chmod +x midicloro`

Connect your devices and start the interactive configuration:

`./midicloro -c`

Available input and output ports will be listed and you will be prompted to select which ones to use. You will then be prompted for the rest of the parameters listed below. When the configuration is completed, MIDIcloro will start.

Run MIDIcloro with the current settings:

`./midicloro`

Automatically start MIDIcloro on boot:
Download the latest version of the auto-start script startm.sh and make it executable.

wget https://github.com/ledfyr/midicloro/releases/download/v1.4/startm.sh

`chmod +x startm.sh`

Configure MIDIcloro with the inputs/output you wish to use and verify that it works.
In the file `/etc/rc.local`, add a call to the startm.sh script with the path to midicloro as parameter. Place it before the last exit command. IMPORTANT - add a `'&'` to let startm.sh run as a background process, otherwise your system will hang on boot.

Example (midicloro and startm.sh are downloaded to /home/pi):

`sudo nano /etc/rc.local`

Add the following line above `exit 0` in the bottom of the file. Do not forget the `'&'`.

`/home/pi/startm.sh /home/pi &`

MIDIcloro will now start when booting up, without the need of logging in (this makes it possible to run MIDIcloro on a headless Raspberry Pi).

If the program does not start automatically on boot, try to run it as:

`sudo ./midicloro`

If you get errors regarding missing libraries, make sure that the `libasound2-dev` and `libboost` libraries exist in `/usr/lib/`.


## Settings
The settings are stored in midicloro.cfg. The interactive configuration (see above) is recommended the first time you run MIDIcloro since it detects all connected MIDI devices. The configuration file midicloro.cfg can also be edited manually (a restart of MIDIcloro is needed for the changes to take effect). All parameters are displayed below with default values (and explanations) where applicable:

input1 =

input2 =

input3 =

input4 =

output =

enableClock = true (enable or disable clock)

ignoreProgramChanges = true (ignore or allow incoming program change MIDI messages)

initialBpm = 142 (this is the clock tempo used when starting MIDIcloro)

tapTempoMinBpm = 80 (lower limit for tempoMidiCC tapping - the tempo will be set using the tempoMidiCC value for taps slower than this)

tapTempoMaxBpm = 200 (upper limit for tempoMidiCC tapping - the tempo will be set using the tempoMidiCC value for taps faster than this)

bpmOffsetForMidiCC = 70 (this offset is added to the tempoMidiCC value to set the tempo)

velocityRandomOffset = -40

velocityMultiDeviceCtrl = true

tempoMidiCC = 10 (MIDI CC number for setting the tempo)

chordMidiCC = 11 (MIDI CC number for setting the chord mode)

routeMidiCC = 12 (MIDI CC number for setting the channel routing)

velocityMidiCC = 7 (MIDI CC number for setting the velocity mode)


## Build and compile
Install the dependencies (see INSTALLATION). Also make sure gcc/g++ is installed.

Compile MIDIcloro with `make` or the following command:

`g++ -Wall -D__LINUX_ALSA__ -o midicloro midicloro.cpp rtmidi/RtMidi.cpp -DBOOST_DATE_TIME_POSIX_TIME_STD_CONFIG -lasound -lpthread -lboost_system -lboost_program_options -lboost_regex`


## Gameboy usage
To use MIDIcloro with the Nintendo Gameboy, you need the following: Gameboy, flash cart with LSDJ, Arduinoboy, Raspberry Pi, USB MIDI interface/cable.
Start by installing MIDIcloro on the Raspberry Pi by following the instructions above. Connect the Gameboy to the Arduinoboy and connect the MIDI out port on the Arduinoboy to a MIDI in port on the USB MIDI interface (this port must be configured as an input in MIDIcloro). Set LSDJ and the Arduinoboy to MIDI out mode and start MIDIcloro on the Raspberry Pi.

To control the clock tempo, chord mode, velocity and channel routing, send MIDI CC messages from LSDJ using the X command.
The high byte of the X command sets the CC number as follows (high byte value = CC number): 0=1,1=2,2=3,3=7,4=10,5=11,6=12.
The low byte of the X command sets the CC value from 0 - 120 in steps of 8 (low byte value = CC value): 0=0,1=8,2=16, ... ,E=112,F=120.
The default CC numbers are 7 (X3\_) for velocity mode, 10 (X4\_) for clock tempo, 11 (X5\_) for chord mode and 12 (X6\_) for channel routing. The default clock tempo offset is 70.

Examples (using default values):

X4A - If sent once: sets clock tempo to 150 (offset 70 + value 80 = 150).

X4A - If sent every beat: sets clock tempo to 150 the first time and calculates the tempo according to the interval between the messages the following times.

X51 - Every note sent on the current channel will generate three MIDI notes creating a minor chord.

X52 - Same as above but generating a major chord.

X53 - Same as above but generating a minor-low chord.

X54 - Same as above but generating a major-low chord.

X50 - Disables the chord mode.

X60 - Route MIDI data sent on the current channel to MIDI channel 1.

X64 - Route MIDI data sent on the current channel to MIDI channel 5.

X6F - Route MIDI data sent on the current channel to MIDI channel 16.


## Supported USB MIDI devices
Devices stated to be class compliant or known to be working in Linux will probably work with MIDIcloro on a Raspberry Pi.
Please contact me if you find a working/non-working device not listed here and I will update the list.

Known working devices:

IK Multimedia iRig Keys

E-MU XMidi 1X1

Known problematic devices (not working out-of-the-box):

M-Audio Midisport Uno


## Other info
MIDIcloro is built and tested on a Raspberry Pi Model B+ running Raspbian Jessie. It may work without modifications on other Linux/Unix systems with ALSA support. MIDIcloro can also probably be built from source for Mac OS X and Windows without too much effort.

MIDIcloro uses RtMidi to handle the MIDI communication. Many thanks go to the author of RtMidi, Gary P. Scavone, for creating this great MIDI API. See rtmidi/readme for license and other information regarding RtMidi.


## Contact
Ledfyr at chipmusic.org
david.ramstrom (at) gmail \_dot\_ com


## License
MIDIcloro is licensed under the MIT license, see LICENSE for details.

