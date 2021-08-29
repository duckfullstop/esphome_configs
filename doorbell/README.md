# 🔔 The Smart(ish) Doorbell

If you're after hacking together a smartish doorbell from a box of scraps, this project might just be for you.

This project comprises the following parts:

  * Doorbell Controller (ESP8266, and either a relay or DFPlayer + speaker (or both))
  * (optionally for homekit) A separate camera (optional)

### The Doorbell Controller

This is simply an ESP8266-compatible board (in my case, a D1 Mini) with either a simple relay (in my case, the clip-in relay shield) or a DFPlayer + simple speaker connected. You can also use both for extra shenanigans.

### The Camera

You can use any camera you want with Home Assistant - the choice really is up to you. This is only really necessary if you want to expose the whole thing to Homekit for a true "this person pressed the doorbell" experience - HK & HA require that the action of pressing the doorbell is attached to a camera object.

## How 2 Config

You probably don't want to just copy the config _ad verbatim_ - give it a read, understand how it works, and adapt it to your setup.

If you're lazy:

D1 Mini: 
  * D1 to relay, relay COM / NO to chime
  * D4 to doorbell pushbutton, rtn pushbutton to ground
  * D2 tx, D3 rx to DFPlayer rx, tx respectively

Wire a speaker to the DFPlayer per the [ESPHome Documentation](https://esphome.io/components/dfplayer.html) and load your preferred melody to a MicroSD card, then you should be good to go.

With the config as written, you'll get a ding-dong from your chime as well as a single playback of the melody on the MicroSD card. There is a debounce of 5 seconds to avoid people mashing your doorbell button and spamming the crap out of the DFPlayer and your Homekit notifications - feel free to tweak these to your preference.

### Home Assistant

I don't have access to the HA code that made this work properly - the short of it is that you'll need a Homekit [Linked Doorbell Sensor](https://www.home-assistant.io/integrations/homekit/#linked_doorbell_sensor) using manual YAML configuration (as of 2021.8).

Feel free to reach out if you have any questions.