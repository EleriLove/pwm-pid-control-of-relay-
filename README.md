# pwm-pid-control-of-relay-
working on a script for esp32 written in arduino which controls a relay based on a thermistor. 


I will be uploading code of this later today as I just woke up, and want to take a look through the code one more time with some better comments to explain weirdness. 

other than that.... yeah this should be fun! 



edit 1:

added code, code represents a middle stage between having had a working settup that was inacurate due to adc jitter caused by high impedence. 

esp32 config resistor was moved to a 4.7k Ohm resistor and a 5k thermistor instead of a 10k calibration resistor and a 10k thermistor. 

going to go to steinheardt-hart as the beta thermistor algorythm seems to read low. this will verify without question what the beta value of the aliexpress thermistor is and make it perfectly calibrated without question. 

edit 2:

branched this code to work for a lab oven I'm working on, check out the new branch. these will be insimultaneous development. 



edit... 50..000000000?

we have commands implemented! 
we can finally change the temperature without reprogramming the whole esp32! 
command format temp=setpoint

whatever you want the set point to be you need to have it directly following the word temp= NO SPACES!
other commands we might implement are the ability to change the PID values! that would be... interesting! 
the command for that would be something like "tune= 80 4.6 8" which would parse out to be P 80 I 4.6 and D 8, but that's for next time! current command lengths are only able to be 4 characters long 



