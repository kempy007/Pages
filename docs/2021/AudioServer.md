---
title: Multi Room Audio with Pi Zero & Dac with piCorePlayer
parent: 2021

---
#### 03/03/2021

# Background

I wanted to do more edge, less cloud, and I had seen the balena project and quite fancied the challenge to setup multiroom audio. To do this with a computer half the size of a post it label is quite something. Whilst I didn't meet my original goal and had to compromise with the software I was still able to accomplish something useful.

# Hardware
|part|supplier|
|---|---|
| PiZeroWH (The extra H means headers solder on) | [PiHut £13.50](https://thepihut.com/products/raspberry-pi-zero-wh-with-pre-soldered-header?variant=547332849681&currency=GBP&utm_medium=product_sync&utm_source=google&utm_content=sag_organic&utm_campaign=sag_organic&gclid=EAIaIQobChMIlvCzr9mU7wIVVODtCh3-cgcbEAQYASABEgLp3fD_BwE) |
| JustBoom Digi DAC | [justboom £24](https://shop.justboom.co/collections/raspberry-pi-audio-boards/products/justboom-digi-zero-phat) |
| JustBoom Red case | [justboom £13.99](https://shop.justboom.co/collections/cases/products/justboom-digi-zero-case) |
| 5v PSU | [justboom £8.99](https://shop.justboom.co/products/official-raspberry-pi-power-supply-2-5a-uk-eu-power-supply-unit?_pos=5&_sid=f942e57f3&_ss=r) |
| Optical Cable | [justboom £4](https://shop.justboom.co/products/optical-cable-1m?_pos=3&_sid=9d0dac9a7&_ss=r) |
| micro SDHC card PNY 32gb | [argos £5.99](https://www.argos.co.uk/product/7314751) |
| Soundbar/hifi with optical input | Should have already |
|---|---|
| | £70.47 each vs Sonos port @ £399 |

# Software Config

* [download v7 std](https://docs.picoreplayer.org/downloads/) and write to sdcard
* Create/copy wpa supplicant file to root of sdcard boot volume
* boot RPI from sdcard
  * enter IP into browser and navigate
    * > Squeezelite Setting >
      * playername = something useful
      * audio output = justboom digi
    * > main >
      * resize fs = to whole sd card
      * bluetooth
        * install
        * enable rpi bluetooth
      * lms
        * install

* mount nas volumes
* open lms to control media

# Other Stuff I tried and failed with

* Balena.io Audio https://github.com/balenalabs/balena-sound 
  * Burns outs SD cards
  * Took a very long time to deploy app, multiple hours
  * Impressed with Fleet control
  * Impressed with container use
  * Impressed with ease of use

* JustBoom player
  * Works great for local player easy to setup.
  * Was unable to get multiroom working