---
title:  "Ultrasonic Synth"
date:   2021-11-27 15:33:33 -0500
# categories: jekyll update
permalink: "/UltrasonicSynth/"
image: assets/Images/UltSynth/accrylic_case.JPG
excerpt: "Playing music without touching an instrument."
---

# Controlling A Hardware Synthesizer with an Ultrasonic Distance Sensor

### Inspiration
After receiving a raspberry pi 3 for my birthday, I wanted to test the possibilities for integrating hardware and software. Inspired by this project on [raspberry pi's website](https://projects.raspberrypi.org/en/projects/ultrasonic-theremin/0), I decided to get my hands on an ultrasonic distance sensor and see what I could do.

### Sonic Pi Integration and Modified Code
Initially I followed the sonic pi tutorial for the ultrasonic theramin. Raspberry pi provides a wiring diagram and bill of materials for the HC-SR04 ultrasonic distance sensor which I tested using a breadboard that I had lying around. 

![](/assets/Images/UltSynth/test_circuit.jpg)

After wiring everything up, I played around with the example python script for reading the sensor data and sending it to Sonic Pi via OSC messaging (see below).

```python
from gpiozero import DistanceSensor
from time import sleep

from pythonosc import osc_message_builder
from pythonosc import udp_client

sensor = DistanceSensor(echo=17, trigger=4)
sender = udp_client.SimpleUDPClient('127.0.0.1', 4559)

while True:
    pitch = round(sensor.distance * 100 + 30)
    sender.send_message('/play_this', pitch)
    sleep(0.1)
```

Sonic Pi was receiving the signal from the Python Script with the following:

```
live_loop :listen do
    use_real_time
    note = sync "/osc/play_this"
    play note[0]
end
```

Before I could get sound out of my stereo, I [configured the raspberry pi](https://bji219.github.io/Raspi/) sound configuration files to interface with my Focusrite 2i2. The raspberry pi is a great option to make your own bluetooth stereo system. I used [raspotify](https://github.com/dtcooper/raspotify) by dtcooper, which works like a charm. 

### Python Code (why use Sonic Pi?)
Playing around with the Sonic Pi software synths is fun, but why stop there? I have a Behringer Poly D, and why not try making some distance-based bloops and bleeps? Rather than trying to configure the MIDI settings on Sonic Pi v3.1 (raspberry pi version, not supported by Sam Aaron), I decided to skip the middle man and use only Python. This way, the signal from the ultrasonic distance sensor can interact directly with a hardware sythnesizer using MIDI rather than software. There is less latency and quicker sleep times do not cause tearing in the audio. I installed the [python mido package](https://mido.readthedocs.io/en/latest/) to be able to work with python and MIDI objects and modified the original code:

```python
# Ultrasonic Distance Sensor- MIDI Send to Poly D
from gpiozero import DistanceSensor
from time import sleep
import mido
import random 

from pythonosc import osc_message_builder
from pythonosc import udp_client

# Initialize the pins on the Raspberry Pi
sensor = DistanceSensor(echo=17, trigger=4)

# Initialize Poly D as i/o
inport = mido.open_input('POLY D MIDI 1')
outport = mido.open_output('POLY D MIDI 1')

# Import custom functions
from Chord_Lib import Chord_Lib
from Rand_Rhythm import Beat

# Read the ultrasonic distance center and calculate midi send
while True:

    if mode == "chord":
        # 40, E2 was 87 and 21; within piano CHORD range of midi notes
        pitch = round(sensor.distance * 60 + 40) 
        # Resert outport
        outport.reset()

        # call chord library function, get random chord
        chord = Chord_Lib(pitch, chord_mode)
        
        # Send notes to Poly D
        note1 = mido.Message('note_on', note = chord[0])
        note2 = mido.Message('note_on', note = chord[1])
        note3 = mido.Message('note_on', note = chord[2])
        outport.send(note1)
        outport.send(note2)
        outport.send(note3)
    
        if len(chord) >3:
            # print(chord[3])
            note4 = mido.Message('note_on', note = chord[3])
            outport.send(note4)

    elif mode == "theramin":
        pitch = round(sensor.distance * 87 + 21) # wider range for arp and theramin 

        # Play single note baed on distance
        rand_note = mido.Message('note_on', note = pitch)
        outport.send(rand_note)
    else:
        old_pitch = round(sensor.distance * 87 + 21)
        new_pitch = Chord_Lib(old_pitch, arp_mode)   
        msg = mido.Message('note_on',note = new_pitch)
        outport.send(msg)
    # Random division between chord or note sends
    rest = Beat(tempo, mode)
    sleep(rest)

```

The script sends a MIDI command through USB MIDI to the Poly D. The note value is calculated based on the distance of your hand (or foot, or whatever) from the sensor. Mido also includes functionality for sending MIDI control commands and pitch bends, so there are a lot of possibilities.

### Mode Selection
I included some user inputs to expand the functionality of the sensor. Why generate a single random note when you could create an arpeggiator, or a random chord, or have some kind of theramin effect? With these couple lines the user can choose.

```python
# User inputs: 
#mode = input("Choose synthesizer mode [chord, arp, or theramin]: ")
# check = False

def get_choice(mode):
  choice = ""
  while choice not in mode:
      choice = input("Choose a synthesizer mode [%s]:" % ", ".join(mode))
  return choice

mode = get_choice(["chord", "arp", "theramin"])

print("You chose", mode, "Mode.")

if mode == "chord":
    tempo = int(input("Input a Tempo in BPM: "))
    chord_mode = input("Choose a chord mode [maj, min, sus, or rand]: ")

    
elif mode == "arp":
    tempo = int(input("Input a Tempo in BPM: "))
    arp_mode = input("Choose an arpeggiator style [arp_maj, arp_min, arp_sus, or arp_rand]: ")
    
elif mode == "theramin":
    tempo = "theramin"
    
else:
    print("Invalid input.")

```

### Random Chord Generator
Playing single notes is great, and adjusting the glide on my synth allowed the notes to be more [legato](https://en.wikipedia.org/wiki/Legato), but what about the other possibilities? The Poly D has 4 oscillators, which means it is capable of [paraphonicly](https://www.sweetwater.com/insync/paraphonic/) playing 4 note chords. By using the ultrasonic distance sensor reading to calculate the root, we can calculate pitches relative to the root as well as a handful of chords. For sake of musicality I narrowed the note range to be that of a [piano](https://newt.phys.unsw.edu.au/jw/notes.html) (from A0 to C8, or MIDI note 21 to 108).

```python
root = round(sensor.Distance*87+ 21)

# Notes relative to the root
notes = ['m2','maj2','m3','maj3','p4','tritone','p5','aug5','p6','m7','maj7','oct','b9','ninth','shp9','tenth','elvnth','shp11','twlth','b13','thirt']

# Create dictionary of notes based on the root
note_dict = {}
count=1
for i in notes:
	note = root+count
	note_dict[i] = note
	count+=1	
 
# Create chord dictionary
chord_dict = {'min_triad':[root,note_dict.get('m3'),note_dict.get('p5')],
			  'maj_triad':[root,note_dict.get('maj3'),note_dict.get('p5')],
			  'aug':[root,note_dict.get('maj3'),note_dict.get('aug5')],
			  'dim':[root,note_dict.get('m3'),note_dict.get('tritone')],
			  'sus2':[root,note_dict.get('maj2'),note_dict.get('p5')],
			  'sus4':[root,note_dict.get('p4'),note_dict.get('p5')],
			  'min6':[root,note_dict.get('m3'),note_dict.get('p5'),note_dict.get('p6')],
			  'min7':[root,note_dict.get('m3'),note_dict.get('p5'),note_dict.get('m7')],
			  'maj6': [root, note_dict.get('maj3'), note_dict.get('p5'), note_dict.get('m6')],
			  'maj7': [root, note_dict.get('maj3'), note_dict.get('p5'), note_dict.get('maj7')],
			  'min_maj7': [root, note_dict.get('m3'), note_dict.get('p5'), note_dict.get('maj7')],
			  'dom7': [root, note_dict.get('maj3'), note_dict.get('p5'), note_dict.get('m7')]};

# Random Chord (3 or 4 notes)
choice = random.choice(list(chord_dict.values()))

# Create MIDI messages to send to Poly D
note1 = mido.Message('note_on', note = choice[0])
note2 = mido.Message('note_on', note = choice[1])
note3 = mido.Message('note_on', note = choice[2])

if len(choice) >3:
	note4 = mido.Message('note_on', note = choice[3])
	print(note1,note2,note3,note4)

```

Now the only limitation is my imagination when adding 3 or 4 chord entries to the dictionary. While this is great for generating chords based on one reading, the next step would be to generate random chord *progressions* based on the root (to be determined). I accomplished an alternative to this by creating a random rhythm function which varies the sleep time of the sensor reading. 

```python
import random
# Generate random rhythm 
def Beat(bpm, mode_in):

	if mode_in == "arp":
	# Time between chord sends (default is 1/8th note based on BPM)
		rest = (60/bpm)*0.5
		return(rest)

	elif mode_in == "chord":
		# Convert BPM to milliseconds
		whole = (60/bpm)*4
		half = (60/bpm)*2
		quarter = (60/bpm)*1
		eigth = (60/bpm)*0.5
		sixteenth = (60/bpm)*0.25

		# Choose random division
		array = [whole,half,quarter,eigth,sixteenth]
		division = random.choice(array)
		# print(division)
		return(division)

	elif mode_in == "theramin":
		# Rest time for theramin is very quick
		return(0.01)
```

The code changes the rate of sleep for the sensor reading depending on which mode the user chose. Since one of the user inputs is also the tempo in beats-per-minute, all the notes are in time as well. 


### Laser Cut Snap-Fit Case
The extra wiring floating around on the proto-board seemed to interfere with the effectiveness of the distance sensor, and if I ever wanted to disconnect the sensor from the onboard pins of the raspberry pi, it would be nice to have a module that I can easily reattach. I decided to design a simple snap-together case made of acrylic to accomplish this. I designed the box to house the wiring and circuitry and to fit within a smartphone camera tripod. 

![Image](/assets/Images/Assembly_Img.PNG)

The box was designed in Solidworks using a 3D model of the HRC sensor that I found online. I exported each piece of the box as a DXF file and set up the laser cut in Rhino 7. I first tested the print in thin plywood before wasting any acrylic.

![Image](/assets/Images/UltSynth/laser_cutwood.jpg)

![Image](/assets/Images/UltSynth/wood_assm.jpg)

The box fit together pretty well considering the plywood was not the same thickness as the acrylic, so it was time to test the good material. The cut was successful, and the pieces fit together nicely. The next step was to solder the circuit together within the box. 

Lacking heat-shrink, I used some black electrical tape to keep the soldered connections from shorting on the pins. I used some foam to keep the resistors off the back of the board, and to press the sensors out through the front holes of the box and keep everything snuggly in place. Lastly, I used some 50/50 epoxy to create strain-relief for the wiring to prevent any loosening of connections. 

![Image](/assets/Images/UltSynth/integratedcircuit.jpg)

The final result came out great- the sensor and circuit is now modular and can be used at leisure on the raspberry pi. It also looks cool- I like being able to see the circuitry through the enclosure. 

![Image](/assets/Images/UltSynth/accrylic_case.JPG)


### Demo
Now all these images and code snippets are great and all, but who cares about them if the thing sounds terrible? Rest assured, I recorded some short clips of myself playing the instrument and included them here for your sonic enjoyment.

#### Theramin Mode
<iframe width="100%" height="300" scrolling="no" frameborder="no" allow="autoplay" src="https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/tracks/1191737329%3Fsecret_token%3Ds-1axgrQMtDV8&color=%23ff5500&auto_play=false&hide_related=false&show_comments=true&show_user=true&show_reposts=false&show_teaser=true&visual=true"></iframe><div style="font-size: 10px; color: #cccccc;line-break: anywhere;word-break: normal;overflow: hidden;white-space: nowrap;text-overflow: ellipsis; font-family: Interstate,Lucida Grande,Lucida Sans Unicode,Lucida Sans,Garuda,Verdana,Tahoma,sans-serif;font-weight: 100;"><a href="https://soundcloud.com/brendostudio" title="Brendan Inglis" target="_blank" style="color: #cccccc; text-decoration: none;">Brendan Inglis</a> · <a href="https://soundcloud.com/brendostudio/theramin/s-1axgrQMtDV8" title="Theramin" target="_blank" style="color: #cccccc; text-decoration: none;">Theramin</a></div>


#### Chord Mode (with random rhythm)
<iframe width="100%" height="300" scrolling="no" frameborder="no" allow="autoplay" src="https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/tracks/1191737515%3Fsecret_token%3Ds-IH34LnP7kwi&color=%23ff5500&auto_play=false&hide_related=false&show_comments=true&show_user=true&show_reposts=false&show_teaser=true&visual=true"></iframe><div style="font-size: 10px; color: #cccccc;line-break: anywhere;word-break: normal;overflow: hidden;white-space: nowrap;text-overflow: ellipsis; font-family: Interstate,Lucida Grande,Lucida Sans Unicode,Lucida Sans,Garuda,Verdana,Tahoma,sans-serif;font-weight: 100;"><a href="https://soundcloud.com/brendostudio" title="Brendan Inglis" target="_blank" style="color: #cccccc; text-decoration: none;">Brendan Inglis</a> · <a href="https://soundcloud.com/brendostudio/chord-gen/s-IH34LnP7kwi" title="Chord Gen" target="_blank" style="color: #cccccc; text-decoration: none;">Chord Gen</a></div>


#### Arpeggiator with Bass sound
<iframe width="100%" height="300" scrolling="no" frameborder="no" allow="autoplay" src="https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/tracks/1191737677%3Fsecret_token%3Ds-iSbnaEmcYc7&color=%23ff5500&auto_play=false&hide_related=false&show_comments=true&show_user=true&show_reposts=false&show_teaser=true&visual=true"></iframe><div style="font-size: 10px; color: #cccccc;line-break: anywhere;word-break: normal;overflow: hidden;white-space: nowrap;text-overflow: ellipsis; font-family: Interstate,Lucida Grande,Lucida Sans Unicode,Lucida Sans,Garuda,Verdana,Tahoma,sans-serif;font-weight: 100;"><a href="https://soundcloud.com/brendostudio" title="Brendan Inglis" target="_blank" style="color: #cccccc; text-decoration: none;">Brendan Inglis</a> · <a href="https://soundcloud.com/brendostudio/bass-arp/s-iSbnaEmcYc7" title="Bass Arp" target="_blank" style="color: #cccccc; text-decoration: none;">Bass Arp</a></div>

Not having to play any notes with my hands frees me up to manipulate the filter knobs to get the cool sounds above!

*use the Force, Luke*
![Image](/assets/Images/UltSynth/whole_rig.jpg)
