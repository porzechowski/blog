---
title: "software countermeasures"
date: 2024-02-05
categories:
  - blog
  - software
  - security
tags:
  - C
  - fault injection
  - software
  - security
  - glitching
toc: true
toc_label: "Table of Contents"
toc_icon: "cog"
---

# Fault injection - software countermeasures

This is the second post in a series about hardware and software security.
In previous one I introduced the idea of hardware fault injections attacks.
In this one I will focus on software countermeasures that can be used to counter them.

Previous post can be found here:
> [Fault injections](https://porzechowski.github.io/blog/blog/software/post-c-bool/)

# Techniques

There are many techniques that can be used to harden our software against fault injection attacks. I will try to cover most useful ones, however extended list is available below.

- performing redundant conditional checks
- storing critical values redundantly
- random duration delays
- constant time for loops
- flow control
- dummy operations, beware to choose these wisely
- constants with high Hamming distance
- check and double check
- write secure code preferably in assembly (compile may remove redundancy)
- check after write
- MPUs
- attack handler
- compiler optimalization

## Redundancy

One of the basic simple and effective techniques is redundancy.
Like I showed [here](https://porzechowski.github.io/blog/blog/software/post-c-bool/), if statement is possible to glitch using simple techniques. We can however make an attacker life harder, by doubling the numbers of needed glitches. Consider following examples:

```c
bool isAdmin = CheckIfAdmin();
if (true == isAdmin) {
        printf("Hello Mr. Admin! \n");
	   //do important admin stuff
}
```

If we are worried about `if` being glitched we could duplicate checks.

```c
bool isAdmin = CheckIfAdmin();
if ((true == isAdmin) && (true == isAdmin)) {
        printf("Hello Mr. Admin! \n");
	   //do important admin stuff
}
```

Now attacker need to glitch twice, so it will cost him more resources to successfully glitch this statement.
However we could improve this example even further.

We will use Complement operator that flips bits. This way we are protected against attacking `isAdmin` variable and glitching `if` statement

```c
#define MY_FALSE 0xA5U
#define MY_TRUE 0x5AU

uint8_t isAdmin = CheckIfAdmin();
if(MY_TRUE == isAdmin) {
    volatile int complement = ~MY_TRUE;
    if (complement == ~isAdmin) { 
        printf("Hello Mr. Admin! \n");
        //do important admin stuff
    }
}
```
Similar technique can be used when writing to a memory.
```c
void redundant_write(volatile uint32_t *address, const uint32_t value)
{
    *address = value;
    if ((value != *address) || (value != *address))
    {
        //panic!!
    }
}
```

## Random Duration Delays

Typically, an attacker will measure time as a constant offset from the initial power-up or from some externally visible event.
Introducing random delays between externally observable events and sensitive operations makes it difficult to determine the time for fault injection.

What we need to do is define a function similar to this:

```c
void random_delay(void) {
    const uint32_t random_ticks = HAL_RNG_Get();
    delay(random_ticks);
}
```

And use it around critical parts of our code.


## Random Duration Delays

Sometimes however, we want to make sure that certain parts of our code have constant execution time to prevent side-channel attacks. For details about this attack check [this.](https://media.defcon.org/DEF%20CON%2024/DEF%20CON%2024%20presentations/DEF%20CON%2024%20-%20Plore-Side-Channel-Attacks-On-High-Security-Electronic-Safe-Locks.pdf
)

So what we want to do is to replace password checking function that usually looks like this:
```c
int checkPassword(const char *providedPassword) {
    // strcmp returns 0 for equal strings
    if (strcmp(providedPassword, correctPassword) == 0) {
        return true;
    } else {
        return false;
    }
}
```

> !! strcmp is bad and unsafe, you want to replace it with `strncpy()` or `strlcpy()`.


Into this:
```c
// Function to compare two strings in constant time
int constantTimeCompare(const char *str1, const char *str2, size_t length) {
    int result = 0;
    for (size_t i = 0; i < length; i++) {
        result |= str1[i] ^ str2[i];
    }
    return result == 0;
}
```

Now, not matter if out password is correct or wrong, the check should take very similar amount of time, so it will be harder to perform power analysis.

## Flow control

Moving forward, next technique that is simple and quite useful is flow control.

Imagine that we are preparing a communication frame for an unspecified interface, and for the frame to be valid we need perform following steps:

1. get current timestamp
2. get data from application layer
3. calculate CRC
   
We could implement simple flow control mechanism that would ensure that all 3 steps, were executed.
We do not check if the CRC or timestamp is correct, **our only concern is that calculation of CRC took place**. And usually during some final frame validation we check if our flow is correct.

In pseudocode it would look like this:
```c
void flow_control(){

    ...
    bool getTimestamp_result = getTimestamp();
    //we do not check if timestamp is correct, we only check that function succeeded
    if(true == getTimestamp_result){
        add_step_flow_control(STEP_1);
    }

    ...
    //stuff happens

    bool getData_result = getData();
    if(true == getData_result){
        add_step_flow_control(STEP_2);
    }

    ...
    //stuff happens

    bool calculateCRC_result = calculateCRC();
    if(true == calculateCRC_result){
        add_step_flow_control(STEP_3);
    }

    //verify that all steps took place
    verify_flow_control();
}
```

## Hamming distance

This sounds fancy but actually is quite easy.

Hamming distance tells us how many bits we need to flip to get from one value to another.
So, in binary representation it will look like this:

`false - 0b0000`
`true - 0b0001`

For this example hamming distance is 1, because we only need to flip 1 but to change `true` into `false`.

So from attacker perspective, the higher the hamming distance is, the harder it is to change one value into another.

This is the reason we all our constants should have high hamming distance.

```c
#define NORMAL_TRUE 1U
#define NORMAL_FALSE 0U

#define BETTER_TRUE 0xAAAA5555U
#define BETTER_FALSE 0x5555AAAAU
```

# Overall remarks

## Do not aggregate checks

Sometimes we introduce nice redundancy to our checks, however we aggregate them later, so you still need just a single glitch, only a bit later.

So instead of this:
```c
int aggregated_result = (check() == true && check() == true);
if (aggregated_result) {
    // Do something if both checks return true
}
```

Try this:
```c
if ((check() == true && check() == true)) {
    // Do something if both checks return true
}
```

## Fight with compiler

You should be very careful and do not trust the compilers.
With a normal level of optimization, most of our security measures will cease to exist.
It's worth spending some time to double-check on assembly level that our security measures still exists.

For example, function like this, with normal `-O2` optimization level...
```c
void check_CRC(void) {
    const uint32_t first_CRC = CalculateCRC();
    const uint32_t second_CRC = CalculateCRC();

    if (first_CRC != second_CRC) {
        // Panic!
        printf("CRC mismatch! System integrity compromised.\n");
    } else {
        // Do stuff
        printf("CRC check passed. System integrity intact.\n");
    }
}
```
... would look like this on assembly level:
![asm_O2](https://github.com/porzechowski/blog/blob/master/assets/images/software_countermeasures/asm_O2.png?raw=true)

But if we switch to `-O0` we will get:
![asm_O0](https://github.com/porzechowski/blog/blob/master/assets/images/software_countermeasures/asm_O0.png?raw=true)

It is easy to see that many of the techniques we can use involve fighting against the compiler.
Since


# The end

All remedies require additional resources. You should not become paranoid and protect everything. We only protect system elements that are crucial require protection.

From the attacker’s perspective, overcoming a particular countermeasure is a matter of resources. 
So all we do it

It's a good idea to start with a "cost-benefit analysis", in short: the potential cost to the attacker should be much higher than the potential gain from the attack.
This discourages attackers from attempting attacks because the risks and costs associated with an attack do not outweigh the potential benefits.




[^1]: https://youtu.be/2F6HDJ1veXY?si=sNT4sWg69Be2JU6_  
[^2]: https://research.nccgroup.com/2021/07/08/software-based-fault-injection-countermeasures-part-2-3/
[^3]: https://media.defcon.org/DEF%20CON%2024/DEF%20CON%2024%20presentations/DEF%20CON%2024%20-%20Plore-Side-Channel-Attacks-On-High-Security-Electronic-Safe-Locks.pdf

