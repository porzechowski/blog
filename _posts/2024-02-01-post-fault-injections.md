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
  - glitching
toc: true
toc_label: "Table of Contents"
toc_icon: "cog"
---

# Breaking software with hardware

This is the first post in a series about hardware and software security.
It covers basic ideas behind fault injections and shows how can it be used.

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
Here we have a very simple snippet of the programme. There is a ```CheckIfAdmin``` function that verifies whether we should be given administrator privileges. 
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

![The idea](https://github.com/porzechowski/blog/blob/master/assets/images/fault_injection/the_idea.gif?raw=true)

## Glitch sources

Here we see the most common ways of a fault injection attacks.
We see that we can tamper with many things, clock, voltage, EM field, temperature, light…. We can we creative here.
Some of these attacks require a certain amount of preparation or circuit interference. So, we say that some are invasive and other are non-invasive. For the voltage glitch, we usually remove a couple of capacitors, from the power supply path. 
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

![glitch basic](https://github.com/porzechowski/blog/blob/master/assets/images/fault_injection/basic_glitches.gif?raw=true)

If we introduce an extra clock edge, the CPU might not be able to complete operations from the previous one, and hopefully do something unexpected.

Another simple technique is voltage glitch. We usually try to lower the voltage on the core.
Circuits rely on a stable voltage level to operate correctly. The voltage determines the speed at which transistors within the core can switch states from on to off. When voltage drops too low, the transistors may not switch states as intended, leading to computational errors or delays.

Glitching allows us to bring the system into the 'undefined zone'.
Depending on the type of glitch we will have a variable number of glitch parameters. Here we see that a voltage one has 2 parameters, the height of the pulse and the width of this pulse. 
Let's think how the core power supply looks. 
On the one hand, we supply 1.1 V to the core, so probably 1.0 V should also be enough to keep it operational.
On the other hand, between 0 or 0.9 V it won't work for us. But we will probably be able to find such a voltage between 1.0 and 0.9 V for which the core will be at the limit of "stability". 

![glitch vdd w](https://github.com/porzechowski/blog/blob/master/assets/images/fault_injection/glitch_vdd_w.png?raw=true)

We can do the same with the width. If we take the voltage away for about 10us our core probably won't notice it. However, if we switch it off for a second we will probably switch it off. 
But what will happen in between these values? For voltage this is what our search area for the right glitch parameters will look like.

![glitch vdd h](https://github.com/porzechowski/blog/blob/master/assets/images/fault_injection/glitch_vdd_h.png?raw=true)

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
Here, it is shown in CPU cycles, but it also can be microseconds. Interestingly, this graph shows that, in this case, **the pulse width is less important than the attack time**. 

![schmoo](https://github.com/porzechowski/blog/blob/master/assets/images/fault_injection/schmoo.png?raw=true)

## Threat model

It all starts with changing the operating conditions of the system.
We change the power supply, temperature, or electro-magnetic field.
This in turn causes changes at the circuit level, interfering with timing, voltage levels the operating thresholds are not met.
This leads to errors in uArchitecture – instruction execution is wrong.
And this propagates to the Instruction level – wrong opcodes.

![schmoo](https://github.com/porzechowski/blog/blob/master/assets/images/fault_injection/threat_model.png?raw=true) [^1]

## Glitching results [^2]

- **Bit flip** - is the change of the bit value to the opposite value, while this bit can be precisely selected by the attacker. A multiple bit flips also fall within in this category as long as all the target bits are selected by the attacker. For example, most of the fault attacks on neural networks utilize this model. Bit flip in memory load instruction will have different effects than bitflip during execution. Wrong instruction vs wrong address,
- **Bit set/reset** is the change of the bit value either to ‘1’ (set) or to ‘0’ (reset). Again, the assumption is that the attacker can select the bit to be set/reset. This fault model is very powerful and can be utilized for example for blind fault attacks,
- **Random byte** is a less precise fault model where a value of a particular byte changes to some random value. This is considered to be the most relaxed fault model to achieve a successful DFA,
- **Instruction skip** practically ignores the execution of the currently processed instruction. Powerful attacks can be introduced by using this fault model, such as privilege escalation, a simple key extraction, or a neural network misclassification,
- **Execution faults** occur in FPGAs where the values being processed are affected by setup violations. For example, physically unclonable functions can be attacked with this fault model, 
- **Stuck-at faults** permanently changes the value of the stored data into some other value. SIFA can be used with this fault model, and also, true random number generators (TRNGs) can be biased by using stuck-at faults,
- **Reset**,
- **Bricking the device.**

## Why would it even work?

Now, let's take crash course in assembly language and machine code. This will help us understand the issues discussed.

To explain this, let's imagine very simple processor, with some memory and an accumulator.

The Accumulator is a kind of a working storage used to store the intermediate results of arithmetic operations.

Our processor can only do 3 following things:

1. NOP - do nothing,
2. LDA - load something from memory to an accumulator,
3. ADD - add something to the value in an accumulator.
   
Each of these instructions has its own unique code – 00, 01, 02.

![CPU](https://github.com/porzechowski/blog/blob/master/assets/images/fault_injection/CPU.png?raw=true)

If we want to load something into the accumulator from address no. 5 in memory, we will issue the command: **LDA 05** in binary that's **0001 0101**

Note that depending on where our bit-flip occurs, we can either change the instruction or the address.
This means that the attacker can hypothetically write or read something in a place in memory where theoretically he should not, or change the operation being performed.

![attack](https://github.com/porzechowski/blog/blob/master/assets/images/fault_injection/machine_code.png?raw=true)

## Becoming the admin

Ok so how can we become the admin?

There are many things we could glitch but we will discuss two easiest ones.

1. First We could alter the memory of ```isAdmin``` variable. If we flip the bit from ```0``` to ```1``` we will become the admin.
   
> [Check out this article about bool in C](https://porzechowski.github.io/blog/blog/software/post-c-bool/)

2. We could try to glitch the ```if``` statement, using EM or voltage glitch. As a result we expect our instruction to mutate or being skipped.

![attack](https://github.com/porzechowski/blog/blob/master/assets/images/fault_injection/attack.png?raw=true)

# The end

Ok so this is the simple idea behind fault injections attacks.
If something is not clear please let me know I will do my best to fix this :)

There will be probably next post soon explaining defense techniques and other aspects of hardware security.

[^1]: Yuce, Bilgiday & Schaumont, Patrick & Witteman, Marc. (2018). Fault Attacks on Secure Embedded Software: Threats, Design, and Evaluation. Journal of Hardware and Systems Security. 2. 10.1007/s41635-018-0038-1.   
[^2]: J. Breier and X. Hou, "How Practical Are Fault Injection Attacks, Really?," in IEEE Access, vol. 10, pp. 113122-113130, 2022, doi: 10.1109/ACCESS.2022.3217212.

