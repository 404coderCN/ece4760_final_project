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
tags: jekyll theme yat
sidebar: []
---

## Introduction

Nowadays, robot hands are everywhere. They don’t just serve as a toy to be played with, but also exist in educational and industrial sectors. The best thing about robot hands is that they can never get tired or injured. They can help us with repetitive and mundane tasks and can operate in conditions that we cannot stand. These characteristics makes them perfect for completing tasks in extreme conditions. But the challenge of using a robot hand is that it is very difficult to operate it exactly the way we want. More specifically, we cannot control its fingers like we control ours, which limits its ability to complete delicate tasks. This led us to think if we can build a robot hand that can precisely follow our movements. 

Inspired by this idea, our final project is to build a robot hand that can mimic human fingers' movements. To accomplish this goal, we designed a control glove that collects curvature data from a set of flex sensors and transmit this data via a transceiver to the robot hand. The microcontroller connected to the robot hand then maps this curvature data to the corresponding duty cycle and thus control the servo movement using PWM. We will discuss the technical details in the following sections.


## High level design
The deisgn of this project can be split into two parts: the robot hand, and the control glove. 

## Hardware deisgn

### Robot hand
Our entire robot hand is 3D printed using PLA material. PLA is known for its high strength, low thermal-expansion, and non-toxic nature and is easily accessible in labs. These properties make PLA ideal for 3D printing. For the individual components of the hand, we used models from an open-source website—InMoov—with slight adjustments in the wrist. Table 1 shows all the components and corresponding cost. When printing these components, we chose an infill of 20% to ensure the hand is robust enough to withstand external force. After printing is completed, we assembled them using epoxy glue. For each finger joint, we inserted in short strips of filament and electrical wires to fill the holes. 

<p><img align="left" src="assets/images/banners/hand1.jpg" height="300" width="350"></p> <p><img align="center" src="assets/images/banners/hand2.jpg" height="300" width="350"></p><p><img align="right" src="assets/images/banners/hand3.jpg" height="300" width="350"></p>


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
