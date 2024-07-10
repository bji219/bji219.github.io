---
title:  "Wireless Vinyl"
date:   2023-03-30 15:33:33 -0500
permalink: "/WirelessVinyl/"
excerpt: "Hacking a low budget vinyl record player"
image: assets/Images/WirelessVinyl/recplayer.jpg
---
# Wireless Vinyl Record Player
The slogan of my maker-enterprise would propably go something like: "Brendan Inglis, making things that already exist in a more complicated way since 1996". People might say - why not just buy a bluetooth record player? Why not just use audio cables to connect the record player to the speakers? To these people I reply, "Because". 

## Workflow
### Record Player --> LogicPro --> Airplay --> Raspberry Pi --> Focusrite 2i2 --> Speakers

## Before
My record player is a pretty cheap one - it does not have bluetooth, and the onboard speakers are low quality. To get a nice sound out of it, I need to connect audio cables from my record player into my nice speakers. This is restrictive due to the length of the cables. If I wanted to have my record player in one location or a separate room, and my speakers in another (say connected to other music gear) then I would need some kind of way to get the audio from my record player to my speakers wirelessly!

![](/assets/Images/WirelessVinyl/before.jpg)


### Shairplay Sync
At the end of the day, this was actually a pretty simple project. It took a decent amount of research (aka googling) to determine the best setup. Initially, my thought was to send the audio from the record player to the Raspberry Pi over USB (essentially a printer cable which outputs audio over USB) and then stream the audio over the internet to my computer. This proved too challenging, since the Raspberry Pi technically has no audio inputs or the necessary drivers to process input audio over USB. Finally, I happened upon [this great repository](https://github.com/mikebrady/shairport-sync) called Shairplay-Sync from Mike Brady (thanks Mike). This adds the capability to act as an Airplay receiver to the Raspberry Pi. It worked great out of the box! Now any airplay capable device can stream to the Raspberry Pi as an output. 

## After 
I can airplay the record player from anywhere in my house or apartment as long as it is connected to my laptop. I can then airplay the audio from my laptop to the Raspberry Pi, which is connected right to my audio interface and into my speakers!

![](/assets/Images/WirelessVinyl/after.jpg)
