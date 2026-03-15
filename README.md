# ls0xx driver with serial VCOM inversion support for ZMK

This driver adds serial VCOM inversion support to ZMK releases based on Zephyr 4.1. The same functionality is available is the main Zephyr branch now, so when ZMK moves to Zephyr 4.4 it will no longer be needed.

Many ZMK builds use these displays, for example they are the display panels used on the Nice! View, yet most people do not break out or connect the EXTCOMIN pin, which is the only way to invert VCOM polarity with the in-tree driver prior to Zephyr 4.4.

## Why invert VCOM polarity?

Without regular inversion of VCOM polarity, the LCD material will suffer from electrolytic degratation, lowering contrast and increasing response time. Eventually, permanent damage is likely, though testing has shown that even several-year old displays showing significant degradation have returned to like new after a short period of inverting VCOM frequently (thank you to @MickiusMousius for testing this).

## Why not use EXTCOMIN?

Because you probably don't have it broken out, and even if you do, you may not have an extra GPIO pin. If you can, it is the best option since it allows you to invert VCOM polarity independently of sending display commands.

## Should this really be in a driver and not my application code?

No, in a perfect world this would be sent at the end of display updates. However, this would require ZMK and other applications to make their display code significantly more complex, for a specific model. If you're writing firmware from the ground-up for a specific product, sending these commands following display updates is better for ensuring that they are in sync.

## How to you prevent display updates from being corrupted by VCOM inversion commands?

There is a semaphore that delays VCOM commands until a display update has been sent, and vice-versa if needed. This may result in a slight impact to smoothness of motion, especially at very high VCOM inversion rates.

## What should the interval between VCOM inversion commands be?

Different Sharp sources have suggested either 1000 ms or 33ms (30 Hz) as the _maximum_ interval. However, these values will show noticeable flicker since the liquid crystal is not identically dark with positive and negative polarity. Too short, however, will use more power and increase the odds of motion appearing choppy due to frames waiting for VCOM inversion commands. I use 18ms (about 55 Hz) as the best balance for me.

The power consumption added is negligible at 1000 ms, however it is small but significant below 33ms. I suggest using a power profiler, but worst case it can become tens of uA or more with both extra MCU load and a small amount of power for the display to switch. Please test and share your findings!

## What should the priority of the thread be?

The lower value (higher priority) is set, the more balanced these commands will be – but at the expense of _other_, more important tasks, like reading your keyboard and outputting data. The balance isn't that important. Unlike the in-tree Zephyr driver, I'm setting the default here to 11. The in-tree value I PR'd is 3 since that matches what it was for EXTCOMIN, but that makes the priority way too high for a keyboard – that comes above the default priority for the input thread!

## Why set an interval when EXTCOMIN is set as a frequency?

Because the Hz setting for EXTCOMIN is not accurate. It's simply calculating an interval, which will not be exactly on the set frequency (because of time for the thread itself to run + calculation truncation as integer math is used). I don't want to promise a frequency when one can't be delivered!