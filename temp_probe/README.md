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
