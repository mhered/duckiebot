# Duckiebot
Two years after completing the [Self-Driving Cars with Duckietown MOOC on edx](https://www.edx.org/learn/technology/eth-zurich-self-driving-cars-with-duckietown) I decided to dust off my Duckiebot and go back to play with it to apply some of the things I learned in the last couple of years.

This repository contains the notes I collected to document this endeavor. 

## Power up
The first challenge was charging the battery after two years in order to power up the bot.

I plugged the bot and tried to power it up - Note: using the side button in the battery, not the button at the top which is only to power down. 

![](./assets/power_up.jpg)

The robot wouldn't boot. The LEDs lit up in odd colours, but the screen didn't work. After a while I noticed the fan in the Jetson would start to power up briefly and then power down again.

After searching the docs and Slack, I came to the conclusion that the battery was totally drained and followed the advice in [the docs FAQ](https://docs.duckietown.com/daffy/opmanual-duckiebot/debugging_and_troubleshooting/faq/index.html) and in [Slack](https://stackoverflowteams.com/c/duckietown/a/230/1434) to unplug the HUT cables going to motors and Jetson, and letting only the cables needed to charge the battery for 5+ hours.

![](./assets/charging_battery.jpg)

To be continued...





