First Project: Create a battery-powered wifi-connected air temperature probe to measure the air temperature in the beer fermenting area. This will inform decisions about what yeasts to use (and therefore what beers to make) and provide data on temperature fluctuations over time that might affect beer quality.

Components:
- ESP32
- TSic 501F temperature IC
- 5v regulator
- 7v lipo

# Temperature IC

Needed something with <2(degrees symbol?) Farenheit range and ideally less than that. Settled on a TSic 501F analog IC which seemed simple and easy to use. I should have gone with the digital one, for reasons I will explain next:

## Calibrating the ADC

TL;DR the adc is 1) noisy and 2) not calibrated. After taking a bunch of readings off of the power supply I calculated a calibration curve and plugged my OG measurements back in to see how inaccurate things were looking. No greater than 0.12V off the curve, but how many degrees does that represent?

### Volts per degree

[Temp Probe Datasheet](https://www.ist-ag.com/sites/default/files/attsic_0.pdf)

Datasheet says that the temp range is -10C to 60C over 1V. So 1000mV / (60 - -10)F = 14.3mv/degrees C. Unfortunately, with calibrated accuracy of 120mV, that's a lot of degrees C to be off by, and even more degrees F. Let's do the math in reverse to see what we actually need ADC-wise. To be more specific, how sensitive must the ADC be to utilize the .1 degree sensitivity of the temperature probe?

### Necessary ADC sensitivity

This is a pretty simple calculation. The sensitivity needed is the range of volts (1000mV) divided by the number ofgraduations we care about. So, as seen above, the number of graduations for 70C where you want 1 degree of sensitivty is 70. But how about for 1 degree F? A one degree change in celcius is a 1.8 degree change farenheit. So the 70 celcius range is 70 x 1.8 = 126 graduations.

Thus for 1 degree F of sensitivity we need 1000mv / 126 = 7.94mV sensitivity.

In general the equations are

(farenheit) `Sensitivity needed (mV) = 1000 / (70 * 1.8 / F)`
(celcius) `Sensitivity needed (mv) = 1000 / (70 / C)`

### Fixing the problem

So we have a problem: we want to measure .1 degree F temperature difference. This requires 0.794mV of ADC sensitivty which the ESP32 adc is unlikely to give us in our current setup. How to solve?

- Switch to digital temperature probe (pro: good measurements, con: weird 1-wire protocol)
- Increase the output voltage range (pro: no equipment change, con: not sure how to do accurately)
- calibrate better (pro: can do now, con: labor intensive, limited by accuracy of bench supply)
- cut out more noise (pro: can do now, con: slower response curve)

I may, in the meantime, order a digital temperature IC. For now I will try to calibrate better and cut out more noise.

## Calibrate better

My first calibration was across the range 0V -> 3.3V. Given that we really only care about temperatures between (approx) 4C (40F) and 38C (100F), we can tighten up the voltage range significantly. Let's find the signal range for these measurements, and then we can calibrate just within this range to increase the analog accuracy.

A note about the ADC. The ADC is 12 bits (0 -> 4095) across 0 -> 3300mV. This means that we can, at best, get 3300 / 4096 =.8mv steps in signal. So we just fundamentally can't (without scaling the signal) get the .1F accuracy that we want.

According to the datasheet the signal range is -10C -> 60C = 0V -> 1V. So, 4C = 14/70 * 1V and 38C = 48/70 * 1V which gives the effective range .20V -> .68V. If we calibrate exactly within that range, we may get more accurate results.

