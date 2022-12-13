---
layout: post
title: Glove-controlled robot hand
subtitle: an ECE4760 final project
author: Jerry Jin
categories: jekyll
banner:
  video: https://vjs.zencdn.net/v/oceans.mp4
  loop: true
  volume: 0.8
  start_at: 8.5
  image: https://bit.ly/3xTmdUP
  opacity: 0.618
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  subheading_style: "color: gold"
tags: microcontroller 
sidebar: []
---

## Introduction

Nowadays, robot hands are everywhere. They don’t just serve as a toy to be played with, but also exist in educational and industrial sectors. The best thing about robot hands is that they can never get tired or injured. They can help us with repetitive and mundane tasks and can operate in conditions that we cannot stand. These characteristics make them perfect for completing tasks in extreme conditions. But the challenge of using a robot hand is that it is very difficult to operate it exactly the way we want. More specifically, we cannot control its fingers like we control ours, which limits its ability to complete delicate tasks. This led us to think if we can build a robot hand that can precisely follow our movements.

Inspired by this idea, our final project is to build a robot hand that can mimic human fingers' movements. To accomplish this goal, we designed a control glove that collects curvature data from a set of flex sensors and transmits this data via a transceiver to the robot hand. The microcontroller connected to the robot hand then maps this curvature data to the corresponding duty cycle and thus controls the servo movement using PWM. We will discuss the technical details in the following sections.



## High level design
The purpose of the project is to build a robot hand whose actions could be controlled wirelessly by a sensor glove. The implementation consists of three major parts: assembling a 3D-printed robot arm whose gesture would be controlled by servos, soldering flex sensors to fingers of the sensor glove which would be connected to GPIO ports of a RP2040 microcontroller, and establishing wireless communication between the glove and the robot arm with nRF2401 transceivers. The required hardware components are listed as following and the 3D printing arm parts prototype could be referenced in the following website:

<table>
  <tr>
    <th>Components</th>
    <th>Quantity</th>
    <th>Shopping Website</th>
  </tr>
  <tr>
    <td>Raspberry Pi Pico</td>
    <td>2</td>
    <td>From lab</td>
  </tr>
  <tr>
    <td>nRF24L01 + Transceiver</td>
    <td>2 </td>
    <td><a class="highlight-link" href="https://www.amazon.com/HiLetgo-NRF24L01-Wireless-Transceiver-Module/dp/B00LX47OCY/ref=asc_df_B00LX47OCY/?tag=hyprod-20&linkCode=df0&hvadid=380013417597&hvpos=&hvnetw=g&hvrand=15043867926934858616&hvpone=&hvptwo=&hvqmt=&hvdev=c&hvdvcmdl=&hvlocint=&hvlocphy=9005779&hvtargid=pla-815756879405&psc=1&tag=&ref=&adgrpid=77922879259&hvpone=&hvptwo=&hvadid=380013417597&hvpos=&hvnetw=g&hvrand=15043867926934858616&hvqmt=&hvdev=c&hvdvcmdl=&hvlocint=&hvlocphy=9005779&hvtargid=pla-815756879405" target="_blank"       rel="noreferrer">Amazon</a></td>
  </tr>
  <tr>
    <td>Servo motor</td>
    <td>5</td>
    <td><a class="highlight-link" href="https://www.amazon.com/ETMall-Digital-Helicopter-Compatible-Raspberry/dp/B08CH2SJLR/ref=asc_df_B08CH2SJLR/?tag=hyprod-20&linkCode=df0&hvadid=459643686207&hvpos=&hvnetw=g&hvrand=15996783978418188023&hvpone=&hvptwo=&hvqmt=&hvdev=c&hvdvcmdl=&hvlocint=&hvlocphy=9005779&hvtargid=pla-947377955673&psc=1" target="_blank"       rel="noreferrer">Amazon</a></td>
  </tr>
    <tr>
    <td>Flex sensor</td>
    <td>3</td>
    <td><a class="highlight-link" href="https://www.adafruit.com/product/182?gclid=Cj0KCQjwqoibBhDUARIsAH2OpWhIXkzbfrneTXbraNZmYftoxgLD-aamazqFndEY7rLEM5Z75axPEasaAroxEALw_wcB" target="_blank"       rel="noreferrer">Adafruit</a></td>
  </tr>
  <tr>
    <td>Glove</td>
    <td>1</td>
    <td><a class="highlight-link" href="https://www.amazon.com/-/es/DEX-FIT-resistentes-el%C3%A1stico-inteligente/dp/B07HHZ4PRZ/ref=sr_1_43_mod_primary_new?__mk_es_US=%C3%85M%C3%85%C5%BD%C3%95%C3%91&crid=1PFRXFN4KW460&keywords=electrical%2Bglove&qid=1667437203&qu=eyJxc2MiOiI0LjAxIiwicXNhIjoiMy43MyIsInFzcCI6IjIuODcifQ%3D%3D&sbo=RZvfv%2F%2FHxDF%2BO5021pAnSA%3D%3D&sprefix=electrical%2Bglove%2Caps%2C105&sr=8-43&th=1&language=en_US" target="_blank"       rel="noreferrer">Amazon</a></td>
  </tr>

</table>





## Hardware deisgn
We will discuss the design of our hardware in two parts; the robot hand and the control glove.
### Robot hand
Our entire robot hand is 3D printed using PLA material. PLA is known for its high strength, low thermal-expansion, and non-toxic nature and is easily accessible in labs. These properties make PLA ideal for 3D printing. For the individual components of the hand, we used models from an open-source website—InMoov—with slight adjustments in the wrist. Table 1 shows all the components and corresponding cost. When printing these components, we chose an infill of 20% to ensure the hand is robust enough to withstand external force. After printing was completed, we assembled them using epoxy glue. For each finger joint, we inserted short strips of filament and electrical wires to fill the holes.

<div><img align="left" src="https://404codercn.github.io/ece4760_final_project//assets/images/banners/hand1.jpg" width="200" height="300"><img align="center" src="https://404codercn.github.io/ece4760_final_project//assets/images/banners/hand2.jpg" width="200" height="300"><img align="right" src="https://404codercn.github.io/ece4760_final_project//assets/images/banners/hand3.jpg" width="200" height="300"></div> 

The movement of the fingers are achieved by pulling on the wire that goes through each joint. We used braided fishing wire to link the fingers together as it can withstand a relatively large amount of force, which is necessary considering that the MG996R servo motor can exert up to 11 kg/cm of stall torque. The wires are routed to the wrist of the robot hand and wraps around a servo wheel.


After the mechanical side of the robot hand was complete, we started adding on servos. Each of the five servos is responsible for controlling one finger. These servos can rotate their shafts within a range of angles and by defining the duty cycle of the pulse-width modulation, we can explicitly turn them to a certain angle. Referencing the datasheet, we know that the MG996R servos take in a PWM signal of 50Hz and the on-time can vary from 1ms to 2ms, corresponding to 0 to 180 degrees. Given the raspberry pi pico’s default frequency of 125MHz, we calculated a new warp value of 20000 and a clock divide value of 125 to achieve a resultant frequency of 50Hz. These two values are carefully selected so that 0.001 and 0.002 are both integer multiples of the reciprocal of 125MHz/warp value. We allocate the selected GPIO explicitly for the PWM function using the gpio_set_function. Then, we set up PWM’s wrap value and divide value to the two numbers we found before. The last step is to select an initial duty cycle and enable the PWM channel. To test the actual behavior of the servos, we connected their signal pin to a GPIO on the raspberry pi pico and observed the reaction of the servos when PWM channel level is setted to 1000 and 2000 respectively. Using this method, we found that the fingers are straight when the duty cycle is set to 1ms and curls up when set to 2ms. The behavior of the thumb is reversed due to reversed wire connection. After 

### Control glove
On the control glove side, the goal is to collect curvature data from the flex sensors and send them to the robot hand using a transceiver. Under the hood, the flex sensors work as variable resistors. By connecting them in series with 10K resistors and connecting wires in between to ADCs on the microcontroller, we can determine the change in voltage level which reflects the bending of flex sensors. When the flex sensors are bent, the materials inside get close together and the resistance increases. 

## Conclusion
Overall, our project successfully fulfilled the initial proposal. The core of our project, wireless control of the robot hand, functioned well in the sense that the robot fingers will move accordingly as the bend of flex sensors. Even when multiple flex sensors are bent simultaneously, the delay of robot fingers’ movement is negligible. In terms of safety relating to our system, we set the lower and the upper bound for the turning degree of servo motors to make sure the hardware will not break. Moreover, to ensure security of our mechanical hauling system, we choose to use the fishing thread that could sustain 30 lb to connect all joints of the robot finger with the servo motor. 

According to users’ feedback, there are still some drawbacks of our system that need to be improved in the future. First and foremost is the hardware problem. After attaching the flex sensors onto the glove, the data collected will sometimes be inaccurate and fluctuate, causing the incorrect movement of robot fingers. This problem can be caused by either inappropriate attachment method of flex sensors to the glove, or by the possible damage of flex sensors. 

Another improvement we can make is that we currently only enable three flex sensors to work simultaneously, because there are only three ADC channels on the Raspberry Pi Pico board. To control all five fingers together, the on-time of the PWM for the servo connected to the little finger is determined by the same ADC channel used for the ring finger, and the movement of thumb is set to be same as the index finger. 

Lastly, the flex sensors are too sensitive that they sometimes send incorrect sensor data when bended slightly. When the sensors are bend over a certain angle, the value also wraps around. This could cause redundant back and forth movements of robot fingers. 
The proposed solutions to the above drawbacks will be discussed in the next section.


**Solution and improvements:**

To address the above three main drawbacks, here are what we could do in the future:

(1)	Modify the way we attach flex sensors to the glove. Instead of only fixing two ends of the flex sensor, we could design a cover for the flex sensor and fix the cover to the glove, enabling the sensor to fit tightly with the glove.

(2)	Use mux in our design. This way we can select multiple inputs to one ADC channel, and we could receive and convert data from five flex sensors and control the five fingers independently.  

(3)	Adding an average function in our code. This function is responsible for taking average of received sensor data over a short period of time and determine the movement of robot fingers. In this way we could reduce much of the back and forth movement because of a few inaccurate data.

Intellectual Property Consideration:
Our design of robot hand is based on the open-source project called <a class="highlight-link" href="https://inmoov.fr/hand-and-forarm/" target="_blank" rel="noreferrer">inMoov</a>, including all 3D-printing files and the instructions for assembling the robot hand. 
For the transceiver driver module for Raspberry Pi Pico, we referred to the public Github page of <a class="highlight-link" href="https://github.com/AndyRids/pico-nrf24" target="_blank" rel="noreferrer">AndyRids</a>.






Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]: https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

$ a \* b = c ^ b $

$ 2^{\frac{n-1}{3}} $

$ \int_a^b f(x)\,dx. $

```cpp
#include <iostream>
using namespace std;

int main() {
  cout << "Hello World!";
  return 0;
}
// prints 'Hi, Tom' to STDOUT.
```

```python
class Person:
  def __init__(self, name, age):
    self.name = name
    self.age = age

p1 = Person("John", 36)

print(p1.name)
print(p1.age)
```
