---
title: Connecting a Photoresistor to an Arduino
tags: arduino
---

A photoresistor is a resistor whose resistance decreases with increasing
incident light intensity, using a photoresistor we can make arduino
sense the intensity of light around it. Let's look at the hook up,

![diagram](/images/post/photoresistor.png)

Using the
[playground](http://www.arduino.cc/playground/Learning/PhotoResistor)
article as reference, resistor (100K) and photoresistor are connected in
series. +5V goes to resistor, ground goes to photoresistor, the junction
where the resistor and photoresistor meets goes to analog 0. Digital 13
is used for the LED.

![hookup](/images/post/arduino-photoresistor.jpeg)

![hookup](/images/post/arduino-photoresistor-1.jpeg)

In my room photoresistor reads around 80, when i put my thumb on it
making it dark, it reads around 500. I wanted the LED to light up when
its dark.

    int lightPin = 0;  //define a pin for Photo resistor
    int threshold = 250;

    void setup(){
        Serial.begin(9600);  //Begin serial communication
        pinMode(13, OUTPUT);
    }

    void loop(){
        Serial.println(analogRead(lightPin)); 
    
        if(analogRead(lightPin) > threshold ){    
            digitalWrite(13, HIGH);
            Serial.println("high"); 
        }else{
            digitalWrite(13, LOW);
            Serial.println("low"); 
        }
    
        delay(100);
    }

So i picked a number in between and used it as threshold, if the reading
is above the threshold it turns the LED on when its below threshold it
turns the LED off.

And the result is,

<p id='preview'>Player</p>
<script type='text/javascript' src='/swfobject.js'></script>
<script type='text/javascript'>
	var s1 = new SWFObject('/player.swf','player','400','300','9');
	s1.addParam('allowfullscreen','true');
	s1.addParam('allowscriptaccess','always');
	s1.addParam('flashvars','file=/video/arduino-photoresistor.mp4');
	s1.write('preview');
</script>

Download [sketch](/code/arduino/photoresistor/photoresistor.pde) -
[fritzing](/code/arduino/photoresistor/photoresistor.fz)
