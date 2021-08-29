# 🐙 The Agile Immersion Heater & Smart Hot Water System

_Because Hot Water should be a lot, lot smarter than it is._

This project is designed to be used with Octopus Energy's _Agile_ tariff to run your hot water at _exactly_ the cheapest time, every night, **without fail.** (It can also be used if you're not on _Agile_ - indeed, it also works great on _Go_, _Go Faster_, and other Economy plans!)

_"But duck."_, I hear you ask, _"why wouldn't I just use (insert other solution here)?"_

This particular solution has a raft of huge benefits over other implementations of smart hot water heating:

  * The call to heat is made by the ESP, _not_ remotely triggered by a remote instance. This means that **if your Home Assistant instance dies overnight for whatever reason, you will still have hot water the next day** - regardless of how long it is dead for.
  * This is a full replacement for your existing immersion heating circuitry - thanks to the raft of temperature sensors, you can **roughly estimate how much hot water you have remaining in the tank** - for example to trigger a notification so you don't end up taking a cold shower. Also, just _imagine_ the graph porn!
  * Due to the temperature sensors, you can also set a desired target temperature remotely - meaning that with agile plans, you can **overheat water as an energy sink during negative price periods** (please read FAQ below)

**⚠️ IMPORTANT: WORKING WITH MAINS ELECTRICITY IS EXTREMELY DANGEROUS, ESPECIALLY WITH HIGH LOAD DEVICES SUCH AS IMMERSION HEATERS (AND THE SMALL TERMINALS OF THE 1PM). CONSULT A QUALIFIED ELECTRICIAN BEFORE ATTEMPTING THIS PROJECT! ENSURE GOOD CONNECTIONS ARE MADE WITH THE SMALL SCREW TERMINALS OF THE SHELLY - SHORTS ARE EXTREMELY EASY, ESPECIALLY WITH BRAIDED CABLE, AND YOU MAY WISH TO CONSIDER USING FERRULES. GOOD EARTHING BETWEEN ALL PARTS OF THIS CONFIGURATION ARE ESSENTIAL. FOLLOW APPROPRIATE ELECTRICAL CODE AND REGULATIONS FOR YOUR AREA. I AM NOT RESPONSIBLE FOR INJURY, LOSS OF PROPERTY, DEATH, OR STOCK MARKET VOLATILITY IF YOU ATTEMPT THIS PROJECT.**

**Hot water should be run occasionally to avoid the buildup of harmful bacteria. Don't leave the system completely off for extended periods.**

**⚠️ STILL HERE? TURN OFF ALL MAINS ELECTRICITY AT YOUR CONSUMER UNIT, AND VERIFY THAT WHAT YOU ARE WORKING ON IS ISOLATED BEFORE PROCEEDING.**

## On the Hardware Side...

The hardware is incredibly simple: take an off-the-shelf [Shelly 1PM](https://shop.shelly.cloud/shelly-1pm-wifi-smart-home-automation-1), flash using the programming header following [Tasmota's guide](https://tasmota.github.io/docs/devices/Shelly-1PM/#serial-flashing), clip the [Temperature Sensor Add-On](https://shop.shelly.cloud/temperature-sensor-addon-for-shelly-1-1pm-wifi-smart-home-automation#312) on it like a little backpack, and then wire as such, reading the bold important text above before you try it:

  * N - Neutral (bus with immersion heater)
  * L1 - Mains Live
  * O - Output to Immersion Heater
  * Temperature Backpack sensors work as a bus - red to red, black to black, etc. The included wago clones are super useful for this, even if not very tidy.
  * **Do not neglect earthing: see above**

The first temperature sensor should go somewhere on your tank to provide a good reading of your tank's average temperature - consider cutting a hole in the insulation and sticking down with a little bit of thermal interfacing material, such as paste. Attach the second to your outgoing (hot) pipe, and the third goes on your incoming (cold) pipe.

**⚠️ IMPORTANT ELECTICAL NOTE**: _DO NOT_ yield to the temptation to stick the first sensor down your immersion heater's thermostat borehole. This is extraordinarily risky - If the ESP does not shut off the relay due to a software fault, or the relay welds closed, energy will keep flowing until something overheats and melts / catches fire, or your water boils and does untold damage to your CH system. Just please don't do it. _But you already knew this because you spoke with a qualified electrician, didn't you?_ (Of course, if you're one of those lucky people who have two boreholes at the head of your tank, disregard this and stick it down the spare one.)

Turn up the thermostat on your immersion heater to the highest temperature that you are likely to want as a failsafe (I set mine to 70)

## Configuration

The configuration file has a couple of important things to point out:

  * `autoheat_fallback_cron` sets a fallback cron schedule that will run a hot water cycle if one has not been triggered within the last 18 hours. Set to a default cheap time of day for your tariff situation. (It is presumed that HA has not updated the target runtime if this has to run - it basically makes sure you're going to have some kind of hot water, even if it's not the cheapest)
  * `autoheat_schedule_entity_id` is the `entity_id` of a _Home Assistant_ datetime that holds the next scheduled run time. For best results with _Agile_, use badguy99's excellent [OctoBlock](https://github.com/badguy99/octoblock) with a specific block set up for your immersion heater - then just point this variable at that entity.

You should also set the `address`es of your temperature sensors appropriately - you can find out how to discover these in the [ESPHome documentation](https://esphome.io/components/sensor/dallas.html?highlight=dallas#getting-sensor-ids). 

## Automation

I have unfortunately lost the automation YAML that I used for popping notifications in HA - if you have some examples to contribute, please feel free to PR them in here.

This whole setup is event driven, which avoids problems if you restart your HA instance mid-cycle - simply listen for the following events and act on them:

 * `moe.duck.esphome.immersion.boost.cyclestarted'` - Boost was requested (by calling the `immersion_boost` service)
 * `moe.duck.esphome.immersion.autoheat.scheduled.cyclestarted` - Started heating by request of programmed target time
 * `moe.duck.esphome.immersion.autoheat.fallback.cyclestarted` - Started heating because we hit the fallback time, and it's been 18 hours since the last programmed time (see configuration above)
 * `moe.duck.esphome.immersion.cyclefinished` - Immersion heater finished

For tracking how much hot water you have left, simply use a template sensor - again, apologies that I've lost the YAML for this, but the short story is `((tank_temp - inlet_temp) / thermostat_target) * 100`. I find that warning at about 20% available is a good idea, though experiment for your own purposes.


## FAQ

 * Q: What's this about overheating my water?
   * A: If you're on Agile and experiencing a negative plunge price (i.e you're being paid to use electricity), you can really take advantage of the situation by turning on as much as humanly possible. Running this setup in combination with OctoBlock will already make the most of the deepest part of the plunge, but you might also want to consider setting the target temperature even higher, such as to use even more electricity (and make more profit). This isn't presently accounted for in the config, but you can either implement it here or have a Home Assistant automation adjust the maximum thermostat temperature.
   * A2: The reason this has an FAQ entry is because **this can be dangerous**: by its very nature, you are overheating your water past where it would normally go, which has all the dangers of overheated water - scolding your hands, pressure in the system, etc. Only fiddle with the maximum temperature if you know what you are doing. I personally wouldn't go much past 70c. 
  * Q: How does this work?
    * A: Read the code, it's fairly straightforward. It's basically some clever lambdas and logic flows.
  * Q: What if I go away? Surely putting energy into my hot water when I'm not there to use it is a bit silly?
    * A: Turn off the `immersion_auto` switch in HA (which is memorised across restarts) and the heater will not run. Please note that you should take appropriate precautions to avoid the build up of dangerous bacteria such as [Legionella](https://www.hse.gov.uk/healthservices/legionella.htm) - consider running to at least 60c once or twice a week as a precaution. **Again, I'm not responsible if you poison yourself - if unsure, just leave it on**
  * Q: This is awesome! Who even are you?
    * A: I'm _duck._, a very bored non-binary software engineer who occasionally writes some neat stuff. Do check out my github for the other stuff I've done. You can also reach out to [@duckfullstop](https://twitter.com/duckfullstop), if you are so inclined.