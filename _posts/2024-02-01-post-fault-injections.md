---
title: "Hardware fault injections"
date: 2024-02-01
categories:
  - blog
  - hardware
tags:
  - C
  - fault injection
  - hardware
  - security
  - compilation
toc: true
toc_label: "Table of Contents"
toc_icon: "cog"
---

# Breaking software with hardware

## Problem

I would like to start with a question: 

**Do you agree with the statement that the hardware guarantees the correct execution of the program?**

Let’s think about this for a second. How often do we take hardware for granted?

Let’s take a look at a simple example.

```c
bool isAdmin = CheckIfAdmin();
if (isAdmin == true) {
        printf("Hello Mr. Admin! \n");
	   //do important admin stuff
}
```
Here we have a very simple snippet of the programme. There is some sort of ```CheckIfAdmin``` function that verifies whether we should be given administrator privileges. 
The problem is that we are not an administrator, and have no right to become one, so this function will ALWAYS verify us negatively. 
For simplicity's sake, we'll assume that the ```CheckIfAdmin``` function was written by a very competent person, so we're not able to fool it software-wise.
But don't worry, this doesn't derail our dreams of becoming an admin. There is hope, and that hope is to provoke the hardware to do something unexpected... to behave in an unexpected way.

## The idea

Let's look at the idea. We have a processor that executes our program.
To cheat the software, we will hit the processor using external forces, or in other terms, we will glitch the processor by introducing a fault.

As a result, one of the possible behaviors of the processor is to skip the instruction which is currently being executed. This might alter currently executing program in various ways.
One possibility is that processor might skip current instruction so it will not check the ```if``` statement!

This means we can bypass the code that checks whether we should become an admin.

The whole idea looks like this:

![Alt Text](/assets/images/fault_injection/the_idea.gif)

## Glitch sources

Here we see the most common ways of a fault injection attacks.
We see that we can tamper with many things, clock, voltage, EM field, temperature, light…. We can we really creative here.
Some of these attacks require a certain amount of preparation or circuit interference. So we say that some are invasive and other are non-invasive. For the voltage glitch, we usually remove a couple of capacitors, from the power supply path. 
If, on the other hand, we want to attack with a laser, it is a good idea to get rid of the outer part of the chip, in practice it can be filed down or etched with acid.

Faults don't have to be just HW they can also be SW, an example of this is the Rowhammer attack, which is an attack on DRAM. 

- Clock
  - Overclocking
  - Clock glitching
- Heat
    - Overheating
- Supply
    - Underfeeding
    - Voltage glitching
- Electro-magnetic:
    - EM pulse
- Optical:
    - Light pulse
    - Laser
- Focused ion beam
- Software:
    - DVFS interface
    - Memory Disturbance (Rowhammer)
  
## Basic glitches

Two most basic glitches are clock and voltage.
The idea looks like this:

![](https://github.com/porzechowski/blog/blob/master/assets/images/fault_injection/basic_glitches.gif?raw=true)

If we introduce an extra clock edge, the CPU might not be able to complete operations from the previous one, and hopefully do something unexpected.

Another simple technique is voltage glitch. We usually try to lower the voltage on the core.
Circuits rely on a stable voltage level to operate correctly. The voltage determines the speed at which transistors within the core can switch states from on to off. When voltage drops too low, the transistors may not switch states as intended, leading to computational errors or delays.

![Alt Text](/assets/images/fault_injection/glitch_vdd_h.png)
![Alt Text](/assets/images/fault_injection/glitch_vdd_w.png)

Glitching allows us to bring the system into the 'undefined zone'.
Depending on the type of glitch we will have a variable number of glitch parameters. Here we see that a voltage one has 2 parameters, the height of the pulse and the width of this pulse. 
Let's think how the core power supply looks. 
On the one hand, we supply 1.1 V to the core, so probably 1.0 V should also be enough to keep it operational.
On the other hand, between 0 or 0.9 V it won't work for us. But we will probably be able to find such a voltage between 1.0 and 0.9 V for which the core will be at the limit of "stability". 
We can do the same with the width. If we take the voltage away for 10us our core will probably not notice it, if we switch it off for a second something will probably happen. 
But what will happen in between these values? For voltage this is what our search area for the right glitch parameters will look like.

## Schmoo plot

In practise when we research the chip, we will try to create so called schmoo plot.
The schmoo - on the right. This is the characteristics of the circuit's behaviour to our glitch. 
The green colour indicates the normal behaviour of the circuit, the yellow colour is a reset due to voltage loss. 
What interests us the most is this small red area. It tells us that there are certain pulse parameters that allows us to modify the processor instructions without a reset.
**Usually, to draw such plot we order the CPU to perform very simple task and we check the result.**
If the result is correct, we are in a green zone, if there is no response are probably in a yellow zone... BUT! If the response is incorrect, jackpot! Successful glitch !!
Usually, we repeat the test for fixed parameters many times.
In practise, if 1 out of 100 tests returned with modified response we consider it to be a high reproducibility rate.
On the left side, we see another very important parameter that is common to most attacks, i.e. the attack time or delay. It is usually counted since some significant observable event, like a device reset.
Here, it is shown in CPU cycles, but it also can be micoseconds. Interestingly, this graph shows that, in this case, **the pulse width is less important than the attack time**. 

![Alt Text](/assets/images/fault_injection/schmoo.png)