The ‘MIDI Piano Instructor’ is a tool aimed at improving the piano keyboard self-teaching experience with direct interaction between the student and software (or human, remote) teacher.
The new level of interaction is reached when the student is guided on the notes to play or exercise on visually with LEDs placed directly on top of the piano keys.

![](/Images/1.jpg)

The scheme is simple : a Personal Computer running music-teaching software or regular score composing/playing software (Sonar, Cakewalk, Sibelius, Rosegarden,..) is connected trough its MIDI out port to the MIDI in port of the ‘MIDI Piano Instructor’. Optionally the MIDI through port of the MIDI instructor can be connected to the MIDI in port of the electronic keyboard or piano. The same MIDI messages that play the music or the exercises on the piano, light up the LEDs on top of the piano keys. The student replicates the exercises watching directly the keyboard without diverting attention from the piano keyboard to look at the PC screen. Voice messages in synch with the exercises (some music composing software support it) might add an extra layer of interaction.

Videos are on [Youtube here](https://www.youtube.com/watch?v=i7AfiCQQ4nE&ab_channel=iam5volt), [and here](https://www.youtube.com/watch?v=I-nwL57EFvU&ab_channel=iam5volt)

One further layer of interactivity is introduced when the MIDI out port of the keyboard is connected to the MIDI input port of the PC, closing the loop and providing teaching software with feedback on the student’s performance. This is done by some of music teaching software already.
The piano does not need to be electronic, it can be a stringed piano or even a pipe organ:  in this case the MIDI through port of the MIDI Piano Instructor might be connected to a MIDI sound generator.
No modification is necessary to the musical instrument as the LED bar is self contained and can be placed on the piano or organ with some velcro strap.
A real human instructor could sit in a different place than the student, with his instructions being played visually and audibly on the student’s keyboard. In this case some sort of server should run on the students’ support PC but I’m not addressing that here.
The video herebelow shows the action quite better than anything else.

The MIDI file plays the song out of its MIDI out port on the sound card game port. The MIDI out port is connected to the MIDI in port of the MIDI instructor which reads a specific selectable channel of the stream (‘note on’ and ‘note off’ messages only). The same MIDI stream is available at the transparent MIDI through port of the MIDI instructor which is connected in my case to the MIDI in port of my piano.
The circuit is designed for 88 keys full piano keyborads but it can be easily used on smaller sized keybords just leaving the unused LEDs out keeping as a reference the ‘middle C’ note or ‘DO centrale’ of the keyboard.

![](/Images/2.jpg)

##How MIDI works in the MIDI Piano Instructor
I wanted this thing to respond to MIDI note-playing messages turning on and off the LEDs instead of actually playing notes.
To do so I had to study MIDI fundamentals and found that the information needed are not many, actually. The basic information needed to understand what makes this thing work are herebelow. Detailes can be found on the internet, of course. The more interesting and consistent source I found here ( http://www.indiana.edu/~emusic/etext/MIDI/chapter3_MIDI.shtml ).
Basically MIDI communication consists of serial packets of bytes (31250 baud, 1 start bit, 1 stop bit, no parity) forming messages sent through an optoisolated port.
Baud rate is different from that of usual serial asynchronous ports as well as different is the physical interface, optical instead of electrical. The main reason for it being optical is for electrical isolation to avoid ground loops which may reflect in ‘hum’ or electrical noise heard in the audio output of the instruments connected. Luckily enough serial communication is controlled by the hardware of the parties so we have to concentrate on the messages content only.
MIDI messages fall into five categories but the one and only necessary here is the Channel voice messages and within this category, ‘note on’ and ‘note off’ are two messages necessary to control the MIDI instructor. These two messages do exactly that: turn notes on and off.
Each ‘note on’ and ‘note off’ message (like all channel voice messages) includes a channel number associated. Almost all MIDI devices are equipped to receive MIDI messages on one or more of 16 selectable MIDI channel numbers. A device will respond to messages sent to the channel (one or more)  it is associated to and ignore all other messages for other channels. There’s an omni mode wherein an instrument will accept and respond to all channel messages, regardless of the channel number. We won’t care about this mode.
Channel voice messages convey information about whether to turn a note on or off, what patch (instrument voice) to change to, how much key pressure to exert and more, but again we just need the ‘note on’ and ‘note off’ messages.
The format of note on and note off messages is this :
Status Byte	Data Byte 1	Data Byte 2	Message	Legend
1000nnnn	0kkkkkkk	0vvvvvvv	Note Off	n=channel number;  k=key 0-127(60=middle C); v=velocity (0-127)
1001nnnn	0kkkkkkk	0vvvvvvv	Note On	n=channel number;  k=key 0-127(60=middle C); v=velocity (0-127)
Key is the number uniquely associated with each piano key; velocity is a number representing how hard the key must be pressed : 0 is no press or key released, 127 is maximum force on the key.

The packets of bytes (the messages) begin with a status byte followed by one or two data bytes.
Status bytes begin with a ’1′ (1xxx xxxx).
Data bytes begin with a ‘ 0 ‘ ( 0xxx xxxx).
The following message turns on note 60 (decimal, ‘middle C’ or ‘DO centrale’) with max velocity on channel 5. Note that channel number in the messages is diminished by 1, that is a nnnn = 0 in the message corresponds to a channel number 1.
status byte	data byte1	data byte2
1001 0100	00111100	01111111
When the note has to be turned off, the following message is sent :
status byte	data byte1	data byte2
1000 0100	00111100	01111111
The following will do the same, turn off the note:
status byte	data byte1	data byte2
1001 0100	00111100	00000000
That is, play (again) the note with velocity (key pressure) = 0;

When consecutive messages would be note on or note off to the same channel, it is common to send just sequencies of data byte1 and data byte2, leaving out status byte that would be the same for all of the messages. This is called ‘Running Status’ and is useful to save bandwidth when large amount of messages must be sent through MIDI (large scores with many instruments).

## The circuit

![](/Images/3.jpg)

The MIDI electrical ports are based on standard MIDI interface as in the MIDI specifications. For MIDI in I used a dual optoisolator because I have a bunch of those and are equivalent to the single version recommended in the MIDI specifications, 6N137, with different pinout.
For the schematic click the picture. It is divided in three parts : the main controller with MIDI in and through ports and the LEDs drivers, then the LEDs array and the MIDI PC interface. The latter is the standard MIDI interface as seen everywhere.
It is worth of note that the shield (ground) connection is connected only at the MIDI through port to maintain a shield on the driving side of a MIDI connection while guranteing electrical isolation at the input side. As I said already, this is necessary to interrupt ground loops that could cause nasty ‘hums’ at the audio outuput of the instrument connected in the MIDI loop.
The help understand how this isolation scheme works, consider that the MIDI through (as well MIDI out) port is connected to a MIDI in port as it is the MIDI instructors’ : MIDI trough (or MIDI out) drives the LED of the optoisolator of the MIDI in port of its counterpart.
The MIDI in levels is converted into electrical 5V and used to drive the UART RX port of the microcontroller, the ATmega168 running at 16MHz.
The microntroller decodes the note on and note off messages and set/reset accordingly a matrix of bits in its register to keep track of which notes are currently on and off. Every 1/1000th of a second the register matrix is replicated on the physical LED matrix.
The LEDs are arranged in 8 rows of 12 LEDs each, incidentally one octave. The notes however are not stored starting from a ‘C’ note or ‘DO’, rather strating from the leftmost key in an 88 keys piano, the ‘A’ note or  ‘LA’.
I’m talking of 88 keys here, but this circuit can be used on any smaller keyboard : unused LEDs should be just left out from the matrix keeping as reference ‘middle C’ note or ‘DO centrale’ : middle ‘C’ (MIDI note 60) is always the ‘C’ note near the center of the keyboard.
The LEDs are driven by driver ICs suitable to source/sink the current required by the LEDs. A parallel driver to source current from the positive to the LEDs’ anodes, a serial to parallel driver to sink the cathodes to ground.
I used blue high intensity LEDs for the bar so I included a LED intensity control. Different colours can be used but care should be used to set the correct values of the 12 current limiting resistors. Blue LEDs require higher forward voltage to have a suitable current flow and light the LED. As an example, red LEDs require a lower forward voltage: in case, 390-470 Ohm should be fine. As a second though LEDs with wide viewing angle would be better.
In case anyone is asking, I’d leave RGB LEDs to mood lights.
The controller fits an aluminum extruded box with external 9Vcc power supply and is connected to the LEDs matrix with DB-25 (Cannon) connectors. Flat crimp cable completes the connection.
The LED bar is worth somea dditional notes.
I originally planned to make it out of cherry wood that would better match the look of my son’s Korg piano but then resorted to regular white plastic cable raceways. The thin cables that make the array of LEDs fit well inside a 1″ x 0.25″ (25mm x 10mm) Iboco raceway.
First I cut it at lenght, then marked with a pencil the distances of each key, white and black, on two lines to accomodate the white along a lower line and the black along an upper one.
With the aid of a piece of  perfoboard (spaced 0.1″ x 0.1″) I marked the hole for the second lead of each LED so as to mount them on the top face of the raceway with the leads going through it. On the opposite side I then soldered a 1×2 piece of perfoboard at each LED to hold the in place. Not bad, eh ? It looks a long work of patience, and it was.
But nothing compared to wiring them.
The LEDs are connected with cathode in parallel note by note (‘C’ cathode to next ‘C’ cathode) and anodes of each group of 12 LEDs in parallel (first 12 notes of the keyboard ‘C’ anode to ‘B’ anode, then the next 12 and so on).
The job came out easier when I decided not to cut the wires at each LED to start the next connection from the same point. Instead, I spliced the plastic insulation with my nails (it hurts in the run, almost everything has a cost) and wrapped the now-bared portion around the lead to solder to. Then I cut the extra portion of the LED lead and moved on to the next connection.
Some white Velcro strap on the back the bar and on top of the piano keyboard keeps the bar in place while not hurting the piano.

## Software

Software is written in ‘C’. Interrupt routines are used to time LED matrix scan.
Two separate time outs are used to turn off a row’s configuration and output the next one. A potentiometer controls the delay between the two operations to determine how long the LEDs remain lit hence controlling the luminosity.
MIDI-Through mimics MIDI-In in hardware so as not to create unnecessary delays to the MIDI messages going to the piano.
The circuit includes a pushbutton to change MIDI channel number to respond to. Pushing the pushbutton the LED bar is cleaned and MIDI current channel number is displayed lighting the LED from central ‘C’ plus current MIDI channel number : channel 1 will light up central ‘C’, channel 2 will light up following ‘C#’ and so on.
The LED will stay on for a few seconds then normal operation will be resumed, pushing the pushbutton while the LED is still displaying MIDI channel number will increase the MIDI channel number by 1. It will loop to channel 1 after channel 16 is reached.
The micro runs with watchdog fuse off and clock divider fuse turned off.

![](/Images/4.jpg)

This is the rear view of the box: from left to right, MIDI-Through port, LEDs intensity control, channel selector and connector for DC in.
The circuit and software are very simple and could be easily modified to make it work on an Arduino (I used a 16MHz clock not by chance) but I’m not planning to do so.
