# The MIDI Light Show

It all started with an [innocent attempt to learn anything about electronics](https://github.com/Terkwood/hello-pi).  We liked rust 🦀, and we liked a Raspberry Pi 3 B+ 🤖, and we liked Bach 🎵.

Who doesn't, right?

## Arrangements

We went thoroughly overboard and created video for several of these productions.

### JS Bach, Fantasia in G Major

<iframe width="560" height="315" src="https://www.youtube.com/embed/QYRhCIzzoi0" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

MIDI arrangement by Jamie Holdham, licensed under [CC 1.0 Universal Public Domain Dedication](https://creativecommons.org/publicdomain/zero/1.0/)

- [The arrangment on MuseScore](https://musescore.com/user/5199981/scores/5157919).
- [Jamie Holdham's user page](https://musescore.com/user/5199981)

Johannes Roussel created the [Electric Piano Soundfont](http://www.johannes.fr/), which is used to render MIDI into audio wavelengths.

### GF Handel, Organ Concerto in G Major Op 4 No 1

<iframe width="560" height="315" src="https://www.youtube.com/embed/TB8kNX9LkIo" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


The audio was sourced through the generous efforts of fellow free-license contributors:

- Charles Adams transcribed the MIDI via MuseScore.  It is licensed under CC 4.0.  (The score itself: https://musescore.com/user/18579/scor... ; his MuseScore page: https://musescore.com/user/18579 ; the CC 4.0 license: https://creativecommons.org/licenses/...)  The MIDI itself was modified only slightly, to combine all notes into a single clef.

- Johannes Roussel created the [high-quality Church Organ Soundfont](http://www.johannes.fr/).


### Erik Satie, Ogive #2

<iframe width="560" height="315" src="https://www.youtube.com/embed/M-bh_AfZ8rU" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


This MIDI light show is powered by a Raspberry Pi with 24 LEDs.  Since each musical octave has 12 half-steps, it allows us to show two full octaves of sound before "wrapping".

In this selection, we play the second of four "Ogives" written by Erik Satie, a composer from the early 20th century. 

This description of the music is provided by Wikipedia:

An ogive is the curve that forms the outline of a pointed gothic arch. Erik Satie gave this title to a set of four piano miniatures published in 1886 at the beginning of his career. Their calm, slow melodies are built up from paired phrases reminiscent of plainchant. He wanted to evoke a large pipe organ reverberating in the depth of a cathedral, and achieved this sonority by using full harmonies, octave doubling and sharply contrasting dynamics.

[Source for this MIDI](https://commons.wikimedia.org/wiki/Image:Erik_Satie_-_Ogive_No.2.mid?uselang=zh/)

## Learn More

Learn more about the wiring used in this prototype, read the source code for the MIDI playback and LED light management, or see additional credits related to the open source software we used, at our [Github page](https://github.com/Terkwood/hello-pi).
