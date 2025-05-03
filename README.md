# 27th April 2025

I flashed 2024-04-04-Patchbox.zip onto the SD card for the new PI 4b.

It booted into the desktop and recognised the pisound without issue - but it wouldn't let me join the wifi and would crash when I tried to open the piano tech demo.

I think the problem is the 1GB of memory causing it to hang.

I tried to update it, but couldn't get it onto the wifi.

Don't think the wifi like having a space in the SSID, so attached it to the old virgin wifi to run an update.

   48  sudo apt update
   49  sudo apt upgrade

Installed fluidsyth, which didn't seem to be present out of the box?

   50  sudo apt install fluidsynth

Jack doesn't seem to be running, even though patchbox thinks it is?

patch@patchbox:~ $ jack_control status
--- status
stoped


I did have a command that made a noise through Jack at one point?

patch@patchbox:~ $ patchbox
Jack service stopped!


Managed to kill enough that I could run:

$ fluidsynth -r 48000 -a alsa -o audio.alsa.device=hw:pisound /usr/share/sounds/sf2/FluidR3_GM.sf2


That connected fluidsynth to pisound.

I also need to connect my midi keyboard output to the fluidsynth input:

183  aconnect -i
184  aconnect -o
186  aconnect 28:0 129:0

```

patch@patchbox:~ $ aconnect -i
client 0: 'System' [type=kernel]
    0 'Timer           '
    1 'Announce        '
client 14: 'Midi Through' [type=kernel]
    0 'Midi Through Port-0'
client 28: 'LPK25' [type=kernel,card=3]
    0 'LPK25 MIDI 1    '
client 32: 'pisound' [type=kernel,card=4]
    0 'pisound MIDI PS-1R8MSEV'
client 131: 'RtMidiOut Client' [type=user,pid=876]
    0 'TouchOSC Bridge '
client 133: 'pisound-ctl' [type=user,pid=981]
    0 'pisound-ctl     '
```

```
patch@patchbox:~ $ aconnect -o
client 14: 'Midi Through' [type=kernel]
    0 'Midi Through Port-0'
client 28: 'LPK25' [type=kernel,card=3]
    0 'LPK25 MIDI 1    '
client 32: 'pisound' [type=kernel,card=4]
    0 'pisound MIDI PS-1R8MSEV'
client 129: 'FLUID Synth (1507)' [type=user,pid=1507]
    0 'Synth input port (1507:0)'
client 130: 'RtMidiIn Client' [type=user,pid=876]
    0 'TouchOSC Bridge '
client 133: 'pisound-ctl' [type=user,pid=981]
    0 'pisound-ctl     '
```

```
patch@patchbox:~ $ aconnect 28:0 129:0
````
  150  jack_lsp
  151  amidiauto
  153  jack_simple_client


What are the fluidsynth commands so far:

patch@patchbox:~ $ /usr/bin/fluidsynth fluidsynth -a alsa -o audio.alsa.device=hw:pisound /usr/share/sounds/sf2/FluidR3_GM.sf2

The -a option sets the audio driver that FluidSynth will use. Here are some common values:
alsa: Uses ALSA for audio output.
jack: Uses JACK for audio output.

The -m option sets the MIDI driver that FluidSynth will use. Common values include:
alsa: Uses ALSA for MIDI input.
jack: Uses JACK for MIDI input.

The -o option in FluidSynth is used to set various configuration options. It allows you to specify parameters for the audio and MIDI drivers, as well as other settings. The general syntax is -o setting=value.



FluidSynth runtime version 2.3.1

> set synth.gain 1.0
> set synth.gain 0.2
> select  0
preset: too few arguments
> chorus 1
> inst number
inst: invalid argument
> prog 0 0
> prog 1 0

> gain 0.5
> gain 1.0
> fonts
> settings
> chorus 0
> help chorus
cho_set_nr [group] n        Set n delay lines (default 3) in all or one chorus group
cho_set_level [group] num   Set output level of all or one chorus group to num
cho_set_speed [group] num   Set mod speed of all or one chorus group to num (Hz)
cho_set_depth [group] num   Set modulation depth of all or one chorus group to num (ms)
chorus [group] 0|1|on|off   Turn all or one chorus group on or off
> chorus off
> chorus 0
> set synth.sample-rate 48000.000
Warning: 'synth.sample-rate' is not a realtime setting, changes won't take effect.
> select 12 0 128 16
> inst 0
fluidsynth: error: No SoundFont with id = 0
inst: invalid font number
> inst 1
000-000 Yamaha Grand Piano
000-001 Bright Yamaha Grand
000-002 Electric Piano
000-003 Honky Tonk
000-004 Rhodes EP
000-005 Legend EP 2
000-006 Harpsichord
000-007 Clavinet
000-008 Celesta
000-009 Glockenspiel
000-010 Music Box
000-049 Slow Strings

To determine the correct MIDI channel for my keyboard, I used the noteon command in FluidSynth, which revealed that my keyboard was sending messages on channel 0. This corresponds to channel 1 in MIDI terminology. I then used the select command to assign the Harpsichord sound to this channel. Specifically, the command select 0 0 0 6 was used to select preset 006 from bank 000 of soundfont 0 on channel 0, successfully changing the instrument to Harpsichord.

Commands used:
```
noteon 0 60 127 (to check the channel)
select 0 0 0 6 (to change the instrument)
```
008-024 Ukulele

select <chan> <sfont> <bank> <preset>

select 0 0 8 24

Oh - and it sounds better to lower the gain in the synth to 1 and up the output volume and use the amps in the pisound to do the heavy lifting.

Yamaha Grand Piano (000-000)
Bright Yamaha Grand (000-001)
Electric Piano (000-002)
Honky Tonk (000-003)
Rhodes EP (000-004)
Legend EP 2 (000-005)

select 0 0 0 0
select 0 0 0 1

select 0 0 0 2
select 0 0 0 3
select 0 0 0 4
select 0 0 0 5


# Adding sounds fonts

C:\Users\gibbens\Documents\Arduino\2025\2025_03_PiSynth\sound_fonts>pscp .\D274.sf2 patch@patchbox:/home/patch/

> load /home/patch/GM1.sf2
fluidsynth: warning: Preset 'Tremolo Strings': Some invalid generators were discarded
loaded SoundFont has ID 2

> fonts
ID  Name
 2  /home/patch/GM1.sf2
 1  /usr/share/sounds/sf2/FluidR3_GM.sf2

> inst 2
000-000 Acoustic piano
000-001 Bright piano
000-002 Electric piano

[ chan 0] [ font 2] [ bank 0] [ preset 1]
> select 0 2 0 1
> gain 1
> gain 0.5

To shutdown - press and hold button for 5 seconds.




https://community.blokas.io/t/a-pisound-based-high-quality-low-latency-sound-module/2567



JACK & ALSA

ALSA (Advanced Linux Sound Architecture) and JACK (JACK Audio Connection Kit) are both crucial components in the Linux audio ecosystem, especially on specialized distributions like Patchbox OS, which is designed for audio and music production.

ALSA
ALSA is a part of the Linux kernel that provides audio and MIDI functionality. It handles low-level audio hardware operations and offers an API for sound card device drivers. 

Key features include:
- Support for various audio interfaces: From consumer sound cards to professional multichannel audio interfaces.
- Modular sound drivers: Allowing for efficient and flexible audio management.
- User-space library (alsa-lib): Simplifies application programming and provides higher-level functionality.

JACK
JACK, on the other hand, is a professional sound server daemon that provides real-time, low-latency connections for both audio and MIDI data between applications. It is particularly favored in professional audio production environments due to its ability to handle complex audio routing and synchronization 

Key features include:
- Real-time, low-latency audio processing: Essential for live audio applications.
- Inter-application audio routing: Allows audio data to be sent between different applications seamlessly.
- Cross-platform API: Supports device sharing and inter-application audio routing 3.

Relationship Between ALSA and JACK
ALSA and JACK often work together in a Linux audio setup. Here's how they relate:

- ALSA as a Backend: JACK can use ALSA as its backend to access audio hardware. This means JACK relies on ALSA to handle the low-level communication with the sound card.

- Audio Routing: While ALSA manages the hardware, JACK handles the routing of audio between different applications, providing the flexibility needed for complex audio workflows.

- Latency Management: JACK's real-time capabilities complement ALSA's hardware management, ensuring low-latency audio processing which is crucial for professional audio work 2 3.
In essence, ALSA provides the foundation by managing the hardware, and JACK builds on top of it to offer advanced audio routing and low-latency processing capabilities.

The JACK daemon is called jackd. It is the core component of the JACK Audio Connection Kit, responsible for managing real-time, low-latency audio and MIDI connections between applications and hardware.

jackd -d alsa

- Run the JACK daemon with realtime priority using the first ALSA hardware card defined in /etc/modules.conf.

-d[river] backend
-n[ame] server-name - Name this jackd instance server-name. If unspecified, this name comes from the $JACK_DEFAULT_SERVER environment variable. It will be "default" if that is not defined.


1. jackd - This command starts the JACK server. You can specify the backend and various options to configure the server.
2. jack_connect - This command connects two JACK ports. It's useful for routing audio or MIDI between applications.
3. jack_disconnect - This command disconnects two JACK ports.
4. jack_lsp - This command lists all available JACK ports. It's useful for identifying the ports you want to connect or disconnect.

## AConnect vs Jack_connect
aconnect: Manages MIDI connections between ALSA sequencer ports.
jack_connect: Manages audio and MIDI connections between JACK ports.

## amidiauto

amidiauto is a background process included in Patchbox OS that automatically sets up MIDI routings as soon as new devices are connected or new software instances are launched.
This tool simplifies the process of managing MIDI connections, making it easier to integrate hardware and software MIDI devices without manual intervention.

Once enabled, amidiauto will automatically connect your MIDI devices to the appropriate software ports whenever they are detected. This is particularly useful for setups involving multiple MIDI controllers and software synthesizers.

This takes care of a lot of use cases. However, it might not suit some more advanced MIDI routings, in particular when it is intended to interconnect a software MIDI port to another software port, or the same for ports categorized as hardware ones. This can be worked around by either manually tweaking the connections after they were automatically set up using software like patchage, aconnect or aconnectgui.


# amidiauto

Automatic MIDI Routing: Automatically sets up MIDI connections when new devices are connected or new software instances are launched 1.
Categorization: Differentiates between hardware and software MIDI ports, connecting them accordingly 1.
Ease of Use: Ideal for straightforward setups where automatic connections between hardware and software ports are sufficient 1.

# amidiminder

Persistent MIDI Connections: Keeps track of all ALSA MIDI connections and automatically reconnects them if devices are disconnected and reconnected 2.
Complex Setups: Suitable for more complex MIDI setups where specific devices need to be connected to specific ports 2.
Rules-Based Configuration: Allows defining specific connection rules in a configuration file, ensuring consistent MIDI routing even after devices are reconnected 2.
Key Differences:
Scope of Automation: amidiauto focuses on automatic connections when devices are detected, while amidiminder ensures persistent connections even after disconnections.
Configuration Flexibility: amidiminder offers more flexibility for complex setups through rules-based configuration 2 1.


192.168.86.112
patch / blokaslabs


root         650  0.0  0.1  17664  6268 ?        Ss   15:58   0:00 /usr/bin/amidiminder -f /etc/amidiminder.rules

jack         651  4.0  3.4 286608 132996 ?       SLsl 15:58   0:02 /usr/bin/jackd -t 2000 -R -P 95 -d alsa -d hw:pisound -r 48000 -p 64 -n 2 -X seq -s -S

patch        926 10.4  5.6 1107332 220892 ?      SLsl 15:59   0:20 /usr/bin/fluidsynth -is /usr/share/sounds/sf3/default-GM.sf3


Restarting fluidsynth gives me 


patch@patchbox:~ $ kill -9 926
patch@patchbox:~ $ fluidsynth -r 48000 -a jack -m jack /usr/share/sounds/sf2/FluidR3_GM.sf2

Port	Type	Meaning
system:capture_1/2	Audio	Microphone or Line input
system:playback_1/2	Audio	Audio output (to speakers/headphones)
system:midi_capture_X	MIDI	Inputs (receive MIDI from outside devices)
system:midi_playback_X	MIDI	Outputs (send MIDI to external devices)



fluidsynth-midi:midi_00
fluidsynth-midi:left
fluidsynth-midi:right

jack_connect system:midi_capture_6 fluidsynth-midi:midi_00

jack_connect fluidsynth-midi:left system:playback_1
jack_connect fluidsynth-midi:right system:playback_2
jack_connect system:midi_capture_6 fluidsynth-midi:midi_00

patch@patchbox:~ $ jack_lsp -c
Reported both ways around.

fluidsynth-midi:left
   system:playback_1

fluidsynth-midi:right
   system:playback_2

system:midi_capture_6
   fluidsynth-midi:midi_00


So just to capture everything:
fluidsynth -r 48000 -a jack -m jack /usr/share/sounds/sf2/FluidR3_GM.sf2
> gain 0.5
> load /home/patch/GM1.sf2
> load /home/patch/D274.sf2
> fonts
ID  Name
 3  /home/patch/D274.sf2
 2  /home/patch/GM1.sf2
 1  /usr/share/sounds/sf2/FluidR3_GM.sf2
> inst 3
000-000 Steinway DII
> select 0 3 0 0
> gain 0.25
> select 0 2 0 0
> select 0 1 0 0


jack_lsp -A -t
jack_lsp -c
jack_connect fluidsynth-midi:left system:playback_1
jack_connect fluidsynth-midi:right system:playback_2
jack_connect system:midi_capture_6 fluidsynth-midi:midi_00
jack_lsp -c


# What starts fluidsynth?

https://community.blokas.io/t/pisound-midi-connection-please-need-help/2036/2

https://github.com/mzero/midiminder/blob/0.70/README.md

https://github.com/whofferbert/midi-cmd-server

   └─user.slice
             └─user-1000.slice
               ├─session-1.scope
               │ ├─890 /bin/login -f
               │ └─959 -bash
               ├─session-3.scope
               │ ├─1019 "sshd: patch [priv]"
               │ ├─1039 "sshd: patch@pts/0"
               │ ├─1040 -bash
               │ ├─1438 sudo systemctl status
               │ ├─1439 sudo systemctl status
               │ ├─1440 systemctl status
               │ └─1441 less
               └─user@1000.service
                 ├─app.slice
                 │ └─fluidsynth.service
                 │   └─955 /usr/bin/fluidsynth -is /usr/share/sounds/sf3/default-GM.sf3


patch@patchbox:~/.config $ cat /lib/systemd/user/fluidsynth.service
[Unit]
Description=FluidSynth Daemon
Documentation=man:fluidsynth(1)
After=sound.target
After=pipewire.service

[Service]
# added automatically, for details please see
# https://en.opensuse.org/openSUSE:Security_Features#Systemd_hardening_effort
ProtectSystem=full
ProtectHome=read-only
ProtectHostname=true
ProtectKernelTunables=true
ProtectKernelModules=true
ProtectKernelLogs=true
ProtectControlGroups=true
# end of automatic additions
# required in order for the above sandboxing options to work on a user unit
PrivateUsers=yes
Type=notify
NotifyAccess=main
EnvironmentFile=/etc/default/fluidsynth
EnvironmentFile=-%h/.config/fluidsynth
ExecStart=/usr/bin/fluidsynth -is $OTHER_OPTS $SOUND_FONT

[Install]
WantedBy=default.target

patch@patchbox:~/.config $

patch@patchbox:~/.config $ cat /etc/default/fluidsynth
# Mandatory parameters (uncomment and edit)
SOUND_FONT=/usr/share/sounds/sf3/default-GM.sf3

# Additional optional parameters (may be useful, see 'man fluidsynth' for further info)
#OTHER_OPTS='-a alsa -m alsa_seq -p FluidSynth\ GM -r 48000'
patch@patchbox:~/.config $

jack_connect fluidsynth-midi:left system:playback_1
jack_connect fluidsynth-midi:right system:playback_2
jack_connect system:midi_capture_3 fluidsynth-midi:midi_00


Checking out my instrument choices:

1  /usr/share/sounds/sf3/default-GM.sf3
000-000 Yamaha Grand Piano
000-001 Bright Yamaha Grand
000-002 Electric Piano
000-003 Honky Tonk

> inst 3  /home/patch/D274.sf2
000-000 Steinway DII

> inst 7  /home/patch/GM1.sf2
000-000 Acoustic piano
000-001 Bright piano
000-002 Electric piano
000-003 Honky tonk piano

ID  Name
 
 6  /usr/share/sounds/sf3/default-GM.sf3
 5  /home/patch/D274.sf2
 4  /home/patch/D274.sf2
 3  /home/patch/D274.sf2
 

 fluidsynth -a alsa -o audio.alsa.device=hw:pisound /home/patch/GM1.sf2

 noteon 0 60 127

 patch@patchbox:~ $ amidiminder
no rules file specified, so no rules added
port added Midi Through:Midi Through Port-0 [14:0]
port added pisound:pisound MIDI PS-1R8MSEV [28:0]
port added LPK25:LPK25 MIDI 1 [32:0]
port added pisound-ctl:pisound-ctl [130:0]
port added RtMidiIn Client:TouchOSC Bridge [131:0]
port added RtMidiOut Client:TouchOSC Bridge [132:0]
port added FluidSynth GM:FluidSynth GM [133:0]
observed connection "pisound":"pisound MIDI PS-1R8MSEV" --> "FluidSynth GM":"FluidSynth GM"
observed connection "LPK25":"LPK25 MIDI 1" --> "FluidSynth GM":"FluidSynth GM"
observed connection "pisound-ctl":"pisound-ctl" --> "FluidSynth GM":"FluidSynth GM"



# Mandatory parameters (uncomment and edit)
#SOUND_FONT=/usr/share/sounds/sf3/default-GM.sf3
SOUNT_FONT=/home/patch/GM1.sf2

# Additional optional parameters (may be useful, see 'man fluidsynth' for further info)
#OTHER_OPTS='-a alsa -m alsa_seq -p FluidSynth\ GM -r 48000'
OTHER_OPTS='-r 48000 -a jack -m jack -p FluidSynth\ GM /usr/share/sounds/sf2/FluidR3_GM.sf2'

fluidsynth -r 48000 -a jack -m jack -p FluidSynth\ GM /home/patch/GM1.sf2

patch@patchbox:~ $ fluidsynth -r 48000 -is -p FluidSynth\ GM /home/patch/GM1.sf2

patch@patchbox:~ $ sudo systemctl restart amidiminder.service
patch@patchbox:~ $
sudo systemctl status amidiminder.service
● amidiminder.service - ALSA MIDI minder daemon
     Loaded: loaded (/lib/systemd/system/amidiminder.service; enabled; preset: enabled)
     Active: active (running) since Tue 2025-04-29 16:52:47 BST; 8s ago
   Main PID: 1811 (amidiminder)
      Tasks: 1 (limit: 3908)
        CPU: 27ms
     CGroup: /system.slice/amidiminder.service
             └─1811 /usr/bin/amidiminder -f /etc/amidiminder.rules

Apr 29 16:52:47 patchbox amidiminder[1811]: port added Midi Through:Midi Through Port-0 [14:0]
Apr 29 16:52:47 patchbox amidiminder[1811]: port added pisound:pisound MIDI PS-1R8MSEV [28:0]
Apr 29 16:52:47 patchbox amidiminder[1811]: port added LPK25:LPK25 MIDI 1 [32:0]
Apr 29 16:52:47 patchbox amidiminder[1811]: port added pisound-ctl:pisound-ctl [130:0]
Apr 29 16:52:47 patchbox amidiminder[1811]: port added RtMidiIn Client:TouchOSC Bridge [131:0]
Apr 29 16:52:47 patchbox amidiminder[1811]: port added RtMidiOut Client:TouchOSC Bridge [132:0]
Apr 29 16:52:47 patchbox amidiminder[1811]: port added FluidSynth GM:FluidSynth GM [133:0]
Apr 29 16:52:47 patchbox amidiminder[1811]: connecting pisound:pisound MIDI PS-1R8MSEV [28:0] --> FluidSynth GM:FluidSynth GM [133:0], by rule: .hw --> .app
Apr 29 16:52:47 patchbox amidiminder[1811]: connecting LPK25:LPK25 MIDI 1 [32:0] --> FluidSynth GM:FluidSynth GM [133:0], by rule: .hw --> .app
Apr 29 16:52:47 patchbox amidiminder[1811]: connecting pisound-ctl:pisound-ctl [130:0] --> FluidSynth GM:FluidSynth GM [133:0], by rule: .hw --> .app
patch@patchbox:~ $



### Button

toggle_wifi_hotspot.sh - Toggle WiFi Hotspot Mode
Holding The Button down for 3 seconds will reconfigure the WiFi of Raspberry Pi board (models with WiFi integrated) or an external SoftAP capable USB WiFi adapter to behave as an Access Point (a.k.a. Wireless Router), as well as start 'touchosc2midi' monitor which will be ready to listen and forward MyOsc data as MIDI to other software such as Pure Data. TouchOSC2MIDI must be installed for this to work, you may get it installed using 'Install Additional Software' menu in pisound-config.

By default, the AP will appear as 'Pisound', and the default password is 'blokaslabs' (without quotes). You can change the name and password by using pisound-config.

The default IP address of RPi in WiFi Hotspot mode is 172.24.1.1 which you can use for ssh, VNC or wireless OSC / MIDI data! That means that you can easily interact with your system using just your laptop or phone, no more wires apart from power supply is needed! And it gets better, if the LAN cable is connected to RPi, it will share the Internet with the connected devices over WiFi!

To send MIDI OSC messages from your other devices to Pisound, connect to the Pisound's WiFi network, and set the 172.24.1.1 IP as the host in the software you're using (such as MyOSC or TouchOSC) settings. See here for more information.


# 1th May 2025

172.24.1.1
patch / blokaslabs

patch@patchbox:~ $ aplay -l
**** List of PLAYBACK Hardware Devices ****
card 0: vc4hdmi0 [vc4-hdmi-0], device 0: MAI PCM i2s-hifi-0 [MAI PCM i2s-hifi-0]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 1: vc4hdmi1 [vc4-hdmi-1], device 0: MAI PCM i2s-hifi-0 [MAI PCM i2s-hifi-0]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 2: Headphones [bcm2835 Headphones], device 0: bcm2835 Headphones [bcm2835 Headphones]
  Subdevices: 8/8
  Subdevice #0: subdevice #0
  Subdevice #1: subdevice #1
  Subdevice #2: subdevice #2
  Subdevice #3: subdevice #3
  Subdevice #4: subdevice #4
  Subdevice #5: subdevice #5
  Subdevice #6: subdevice #6
  Subdevice #7: subdevice #7
card 3: pisound [pisound], device 0: PS-1R8MSEV snd-soc-dummy-dai-0 [PS-1R8MSEV snd-soc-dummy-dai-0]
  Subdevices: 0/1
  Subdevice #0: subdevice #0

default: /usr/bin/fluidsynth -is -r 48000 -a jack -m jack -p FluidSynth GM /usr/share/sounds/sf2/FluidR3_GM.sf2


 /usr/bin/fluidsynth -i -r 48000 -a jack -m alsa_seq -p FluidSynth\ GM /usr/share/sounds/sf2/FluidR3_GM.sf2


fluidsynth -g 1.0 -r 48000 -a alsa -o audio.alsa.device=hw:pisound -p FluidSynth\ GM /home/patch/GM1.sf2

'-g 1.0 -r 48000 -a alsa -o audio.alsa.device=hw:pisound -p FluidSynth\ GM /home/patch/GM1.sf2'


sudo systemctl disable --now jack

https://www.mankier.com/1/fluidsynth#--server
https://www.fluidsynth.org/api/fluidsettings.xml

# 3rd May 2025


aseqdump -p PORT_NUMBER
