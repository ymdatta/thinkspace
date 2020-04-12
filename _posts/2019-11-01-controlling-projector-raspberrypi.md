---
layout: post
title: 'Controlling a Projector Automatically from Raspberry Pi'
date: '2019-11-01 21:21:21'
tags: kernel, module
---
# Table of Contents

1.  [Aim of this Experiment](#org57f6ed9)
2.  [Introduction](#orgdf0c986)
3.  [Preparation](#org9dd52b6)
    1.  [Components Used](#org2d8fdb6)
    2.  [Board Setup:](#orga10a229)
    3.  [Frameworks/Modules/packages used](#orgc2d0c46)
4.  [Testcases](#org3855cc0)
    1.  [Case 1:](#org9529368)
        1.  [Instructions:](#org9b4384d)
        2.  [Expected Output:](#orgf733723)
    2.  [Case 2:](#org474aed0)
        1.  [Instructions:](#org9852433)
        2.  [Expected Output:](#org5e48455)
    3.  [Case 3:](#orga8d3e7a)
        1.  [Instructions](#org76ae6bd)
        2.  [Expected Output](#org5d88207)
5.  [Observations](#org57c84d5)
6.  [Analysis](#orgd8de749)
    1.  [General](#org4559e54)
    2.  [Hardware](#org9b2c33a)
    3.  [Software](#org2e777ed)
7.  [Questions from Assignment](#org5deae27)
    1.  [How do you distinguish an IR emitter from a receiver?](#org4f38404)
    2.  [We can't see IR. How do you check if emitter is working?](#org331b92a)
    3.  [What is the frequency of IR receiver?](#org12ee394)
    4.  [Why is the IR receiver built to a specific frequency?](#org82f5919)
    5.  [Which pin on HDMI connector is used to detect connect/disconnect?](#orgca10ea8)
    6.  [From the Device Tree, which GPIO controller manages the PIN?](#org8dea205)
8.  [Deviations from Assignment](#org7780c70)
9.  [Conclusions](#orge90f23c)
10. [Future work](#orga0f5557)
11. [References:](#org8a1225f)
12. [Appendices:](#orgaec8ea8)
    1.  [Makefile](#org91f046a)
    2.  [udev rules](#org7960743)
    3.  [Code pieces](#orgb38f871)



<a id="org57f6ed9"></a>

# Aim of this Experiment

To control the projector automatically from Raspberry Pi.


<a id="orgdf0c986"></a>

# Introduction

Raspberry Pi can be used to deliver presentations directly to our 
class projector. When HDMI cable is unplugged at the end of the
presentation, we have to use the remote control to switch off
the projector. But, the Raspberry Pi can detect the HDMI disconnect
and has the capability to emit IrDA signals to switch off the
projector automatically.   

This experiment is about the case where we connect Infrared(IR)
emitter to a Raspberry Pi board and program it so that the
projector is turned off when HDMI cable connected to the Raspberry Pi
is unplugged.


<a id="org9dd52b6"></a>

# Preparation


<a id="org2d8fdb6"></a>

## Components Used

-   Raspberry Pi 2 Model B V1.1
-   TSOP1738 IR Receiver
-   IR LED transmitter
-   NPN transistor
-   Resistors
-   MF, FF Jumper wires


<a id="orga10a229"></a>

## Board Setup:

-   Raspberry pi was configured to receive IR signals via GPIO pin 23,
    and to transmit signals via GPIO pin 22.
    
    This was done in /boot/config.txt so that these configurations
    can be loaded at boot time.
    
    ![img](/assets/GPIO/boot_gpio.png)

-   We can connect an IR LED directly to GPIO 22 (for transmitting) on
    Raspberry Pi, but the LED's output signal will be too weak, and
    the IR transmitter will have a very limited range. 
    
    We solve this using a transistor, which amplifies the current output
    from a pin, thus increasing IR LED's signal strength.[1]


<a id="orgc2d0c46"></a>

## Frameworks/Modules/packages used

-   When kernel was upgraded from 4.19 to 4.19.x, the lirc-rpi module
    got replaced by gpio\_ir\_tx and gpio\_ir\_recv modules.   
    
    ![img](/assets/GPIO/mod_gpio.png)

-   **ir-ctl**[2] is used to receive and transmit raw IR signals.


<a id="org3855cc0"></a>

# Testcases


<a id="org9529368"></a>

## Case 1:


<a id="org9b4384d"></a>

### Instructions:

-   SSH into Raspberry Pi.
-   Connect an IR receiver to the GPIO 23.
-   Open the Terminal, and wait for the signal by doing
    *ir-ctl &#x2013;receive=signal.txt &#x2013;device=/dev/lirc-rx*
-   Send an IR signal to the receiver by pointing an IR transmitter
    in the IR receiver direction. (Use projector's remote for this)


<a id="orgf733723"></a>

### Expected Output:

-   Open signal.txt, it contains raw IR signal, in the form of 
    alternative spaces and pulses.


<a id="org474aed0"></a>

## Case 2:


<a id="org9852433"></a>

### Instructions:

-   To be done after Case 1
-   SSH into Raspberry Pi.
-   Connect an IR transmitter by method mentioned in Board setup.
-   Point the transmitter in the projector direction.
-   Open the Terminal, send the signal received in Case 1 &#x2013; signal.txt
    by doing, *ir-ctl &#x2013;send=signal.txt &#x2013;device=/dev/lirc1/*


<a id="org5e48455"></a>

### Expected Output:

-   Projector should behave in the same way as it would when it receives
    the same signal from the remote controller.
    (For Ex: If we send a power signal via IR transmitter, the projector
    should turn off)


<a id="orga8d3e7a"></a>

## Case 3:


<a id="org76ae6bd"></a>

### Instructions

-   ssh into the Raspbery Pi
-   Insert the hdmi\_rpi.ko module in the kernel.
-   Check whether the module is inserted by doing *lsmod*.
-   ssh out of the Raspberry Pi
-   Prepare the circuit according to the Board setup section mentioned above.
-   Attach the HDMI cable to the Raspberry Pi.
-   Switch on the monitor.
-   Reboot the Raspberry Pi with HDMI cable attached if there's no display
    on the screen.
-   IR LED emitter on the Board should be pointed towards projector.
-   Unplug the HDMI cable.


<a id="org5d88207"></a>

### Expected Output

-   When HDMI cable is unplugged, the projector should automatically
    be switched off in 3-4 seconds.


<a id="org57c84d5"></a>

# Observations

-   Device tree blobs are stored in /boot directory. From our model name
    on the Board, we can find the blob file used.
    
    ![img](/assets/GPIO/dt_blob.png)

-   We can get information from device tree blob using *fdtget* utility[3].
    
    For example, to get list of all properties of hdmi node:
    
    ![img](/assets/GPIO/hdmi_props.png)
    
    To get gpio information regarding particular property of the hdmi node,
    we see that there is a property called *hpd-gpios*. If we look through it:
    
    ![img](/assets/GPIO/hpd_gpios.png)
    
    The first value indicates the phandle of the controller, second value
    indicates the GPIO pin used to detect HDMI connection and the third value
    indicates the default value of the pin.

-   If we look at the GPIO pin's value, after exporting it to userspace, we
    see that it's value changes whenever HDMI is plugged/unplugged. It starts
    with 1 (see image above and previous point), and when HDMI is plugged, it's 
    value becomes 0 and becomes 1 when it's unplugged.
    
    ![img](/assets/GPIO/unplug_hdmi.png)

-   When i ran a function which sleeps in the timer callback, i got this error
    and the system froze.
    
    ![img](/assets/GPIO/bug_sleep.png)

-   *tvservice* utility can be used to get information about the HDMI. It internally
    processes the EDID blocks to get information.
    
    ![img](/assets/GPIO/tvservice.png)
    
    We can also monitor the plugging and unplugging events using *tvservice -M*.
    
    ![img](/assets/GPIO/tv_mon.png)
    
    We can get EDID tag using *tvservice* utility. We can then display the
    binary in suitable format.
    
    ![img](/assets/GPIO/tv_edid.png)

-   From the kernel module, i have written, when hdmi is unplugged from the
    Raspberry pi, an uevent is generated with specific environment.
    
    ![img](/assets/GPIO/udev.png)
    
    In the above picture, we see that when HDMI is unplugged a 'change' event
    is generated on the device.

-   From the kernel module i have written, when hdmi is unplugged, i have
    included a *printk* statement, so that it prints out 'HDMI Unplugged'
    to dmesg.
    
    ![img](/assets/GPIO/dmesg.png)


<a id="orgd8de749"></a>

# Analysis


<a id="org4559e54"></a>

## General

-   When the Raspberry Pi was booted, two device files were created in */dev* directory. They
    were *lirc0* and *lirc1*. When i was transmitting the signal, for the first time, i used
    *lirc0*, it worked well. When i booted my system again, i tried to use *lirc0* again for
    transmitting, but i got the error by saying that, **dev/lirc0** cannot transmit.
    
    After reading through gpio-ir, i realised that creation of receiver and transmitter devices
    is automatic. i.e, we cannot always be sure that *lirc1* is a receiver or transmitter.
    
    I was able to solve this problem with the help of *ir-ctl*. It has a option called 'features'
    which lists the features of the lirc device.
    
    ![img](/assets/GPIO/irctl_f_lirc.png)


<a id="org9b2c33a"></a>

## Hardware

-   When i first used TSOP1738 IR Receiver, i thought the middle pin was the signal, and
    the other two were Vcc and Ground. I didn't have any reason behind this, i just with
    this.
    
    I then tried testing this receiver, i didn't get any information from the sensor,
    my first thought was that maybe the sensor is not a good one, i tried using a different
    sensor, even then i was not able to receive any data. I was sending IR signals through
    the remote, my second though was maybe my remote is wrong. But this was debunked when
    i checked the LED of remote through my mobile camera which didn't have any filters. I 
    then realised that there is no problem with my remote.
    
    This went on for a long time, when i was discussing this with a senior of mine, he 
    asked me whether i consulted data sheet of TSOP sensor before using it. When i checked
    the data sheet of the sensor, i realised that my assumption about the which pins are which,
    of the signal were wrong. I then made correct connections, i was able to receive IR data
    from the remote.

-   Another problem i faced was with the IR LED transmitter. When i tried to send the signal
    i received via TSOP sensor, there was no change in the projectors status. (Signal was
    received from the projector's remote)
    
    But, when i tried to receive this signal from the TSOP sensor (i saved the signal from the
    projector in a file, and this file was used for sending), i was able to receive the signal,
    i wasn't able to understand why the projector was not receiving.
    
    After a while, when i checked the IR LED from my mobile phone's camera, which didn't have
    any IR filter, i saw a very dim light. I realised that the strength of the signal is not
    enough to reach the projector. I then used a transistor to solve this problem, and then
    when i transmitted a signal, projector was able to receive it.


<a id="org2e777ed"></a>

## Software

-   I have written a platform device called *hdmi*, which has a sysfs attribute called state,
    which when read gives either 0 or 1. 1 meaning no HDMI connected, and 0 meaning HDMI is
    connected.

-   One of the most interesting problem in this assignment was polling, how do we poll, so that
    regardless of when HDMI is unplugged, the device *hdmi* can recognize it. The answer to this
    was timers.
    
    Another interesting problem was detecting the HDMI device itself.
    
    To solve the second problem, i went through the firmware code of Raspberry pi, looking for
    an interface which may help with my problem, i then found the interface   
    
        int rpi_firmware_property(struct rpi_firmware *fw,
        			  u32 tag, void *tag_data, size_t buf_size)
    
    Now going through the accompanying documentation [4, 5]. I realised that this function
    reads the specified EDID block from attached HDMI/DVI device[4, 5]. I then started to
    use this function in the timer, but when i began to run the module, i got the following
    error and laptop froze.
    
    ![img](/assets/GPIO/bug_sleep.png)
    
    I began searching for the reason behind this error, i talked to few people from the
    \#kernelnewbies channel of IRC. They then suggested to look at the error closely, and
    then i saw at the top there was "BUG: scheduling while atomic". I then realised that
    there was scheduling happening where it is supposed to be atomic. But, when i looked
    at my code, there was no scheduling/wait. I then went into *rpi\_firmware\_property()*
    call, i.e, what is called underneath, these were the calls.   
    
    *rpi\_firmware\_property()* -> *rpi\_firmware\_property\_list()* -> *rpi\_firmware\_transaction()*.  
    
    In the *rpi\_firmware\_transaction*, there was a mutex lock. Now, i was able to connect
    the pieces and realised this was the reason for scheduling in the timer. So, i searched
    for something similar which executes at a particular time, but where the execution
    can have locks (or) can sleep etc. Reading LDD3[7], from chapter 7[7], i realised
    that we can use workqueues to solve this problem.   
    
    So, every time a timer expires, work is pushed to the workqueue. This work is nothing
    but finding whether HDMI is connected or not. After making it work now, using workqueues,
    i realised that we can't rely on EDID blocks to know whether the HDMI is connected or not.
    We can only get information about the connected device like Vendor ID, Product name etc   
    
    So, i tried to search for another method. By this time, i started looking into device tree
    and realised that i can use GPIO 46, to detect whether the HDMI is connected or not.
    
        static int gpio_get_value(unsigned int gpio)
    
    It returns 0 if HDMI is connected, and 1 if it isn't.

-   We don't need to create a device to detect HDMI, but to send the event detection to user
    space we need to. 
    
    From the device, the uevent can be sent using *kobject\_event\_env()*. This was really interesting
    and i realised this is the way how actually uevents are sent from kernel space code.

With this, we now have a device which will send a uevent whenever it detects that HDMI is unplugged.


<a id="org5deae27"></a>

# Questions from Assignment


<a id="org4f38404"></a>

## How do you distinguish an IR emitter from a receiver?

We can supply a voltage to the LED, and check using the photo camera
to see if light emitted is IR.

Generally speaking IR receiver sensors(like TSOP) have three legs but IR emitter LED's
only have two. So, that's one more way. Also, if IR Receiver is a two legged diode, then
it would be mostly in black color as opposed to emitter which is generally not dark.


<a id="org331b92a"></a>

## We can't see IR. How do you check if emitter is working?

We can use our mobile phone's camera to check that. Of course it shouldn't have
IR filters.


<a id="org12ee394"></a>

## What is the frequency of IR receiver?

From the Wikipedia article [8], IR frequency is between 33 and 40 kHz or 
between 50 and 60 kHz.  From the article, the most frequently used NEC protocol,
specifies a frequency of 38kHz.


<a id="org82f5919"></a>

## Why is the IR receiver built to a specific frequency?

If the IR receiver is not built for a specific frequency, it picks signals from
all the heat sources around it. So, to avoid picking those signals, its built
at a specific frequency. [9, 10]


<a id="orgca10ea8"></a>

## Which pin on HDMI connector is used to detect connect/disconnect?

Pin 19.   

From the schematic [11],

![img](/assets/GPIO/schem.png)


<a id="org8dea205"></a>

## From the Device Tree, which GPIO controller manages the PIN?

From the device tree, getting information using *fdtget* of hdmi node's
hpd-gpios property

![img](/assets/GPIO/hpd_gpios.png)

The first value indicates the phandle of the controller, second value
indicates the GPIO pin used to detect HDMI connection and the third value
indicates the default value of the pin.

![img](/assets/GPIO/qgpio.png)

The above image shows the gpio controller which handles HDMI detection
gpio pin. (We can see that phandle value is 0x0d (= 13 in base 10), which is same as
first parameter of *hpd-gpios* property in device tree)


<a id="org7780c70"></a>

# Deviations from Assignment

-   The Assignment also asked to automatically power on the projector
    when the HDMI wire is plugged into Raspberry Pi.
    
    This is not feasible because, unless the projector is powered on
    HDMI's GPIO doens't receive a signal. This was checked by looking
    for changes in the *value* attribute of GPIO 46 Pin.
    
    So, unless we plug in the HDMI cable and power the projector on,
    the HDMI remains undetected by GPIO.


<a id="orge90f23c"></a>

# Conclusions

-   I felt this project was very close to a real life project, because
    we were writing logic to control an actual device. I learned a lot
    from it.

-   I was able to appreciate the facilities the kernel provides for
    talking to devices.

-   I realised that the place to get information is from the code itself,
    not from googling for it.

-   I was able to appreciate the power of the modules, what it can do,
    the fact that we can send a uevent from the device itself felt very
    powerful to me.

-   I made sure that i won't using any userspace code as far as detecting
    the HDMi event and sending uevent is concerned. This was one of my design
    decision, and due to this i learnt a lot. We could have just written
    a bash script on *tvservice*, but i chose to write a kernel module
    and appreciate the facilities the kernel provides.

-   Though i used GPIO pin for detecting presence of HDMI, and given the
    fact that the pin used may be different for different boards, i still
    feel that this is the elegant way, because we are getting information
    directly from the device tree.


<a id="orga0f5557"></a>

# Future work

Right now, because of restrictions to other classrooms with projectors, 
this was tested on only one projector, this has to be tested on different
projectors with different signals.


<a id="org8a1225f"></a>

# References:

1.  Dmitri Popov. IR Remote Control " Raspberry Pi Geek. Retrieved November 21, 2019 from <http://www.raspberry-pi-geek.com/Archive/2015/10/Raspberry-Pi-IR-remote>
2.  Anon. Home. Retrieved November 21, 2019 from <https://www.mankier.com/1/ir-ctl>
3.  Anon. FDTGET(1) - man page online: user commands. Retrieved November 21, 2019 from <https://www.venea.net/man/fdtget(1)>
4.  Raspberrypi. raspberrypi/documentation. Retrieved November 21, 2019 from <https://github.com/raspberrypi/documentation/tree/JamesH65-mailbox_docs/configuration/mailboxes>
5.  Raspberrypi. raspberrypi/firmware. Retrieved November 21, 2019 from <https://github.com/raspberrypi/firmware/wiki/Mailbox-property-interface>
6.  Greg Kroah-Hartman, Alessandro Rubini, and Jonathan Corbet. Linux Device Drivers, 3rd Edition. Retrieved November 21, 2019 from <https://www.oreilly.com/library/view/linux-device-drivers/0596005903/>
7.  Greg Kroah-Hartman, Alessandro Rubini, and Jonathan Corbet. Linux Device Drivers, 3rd Edition. Retrieved November 21, 2019 from <https://www.oreilly.com/library/view/linux-device-drivers/0596005903/ch07.html>
8.  Anon. 2019. Consumer IR. (November 2019). Retrieved November 21, 2019 from <https://en.wikipedia.org/wiki/Consumer_IR>
9.  Anon.Retrieved November 21, 2019 from <https://learn.sparkfun.com/tutorials/ir-communication/all>
10. Chris Young. Infrared Transmit and Receive on Circuit Playground Express in C . Retrieved November 21, 2019 from <https://learn.adafruit.com/infrared-transmit-and-receive-on-circuit-playground-express-in-c-plus-plus-2/understanding-infrared-signals>
11. Anon. Schematics. Retrieved November 21, 2019 from <https://www.raspberrypi.org/documentation/hardware/raspberrypi/schematics/README.md> (Raspberry Pi 2 Model B)


<a id="orgaec8ea8"></a>

# Appendices:


<a id="org91f046a"></a>

## Makefile

![img](/assets/GPIO/Makefile.png)


<a id="org7960743"></a>

## udev rules

![img](/assets/GPIO/97-hdmi.rules.png)


<a id="orgb38f871"></a>

## Code pieces

![img](/assets/GPIO/code_1.png)

![img](/assets/GPIO/code_2.png)

where envp is *char \*envp[] = {"SUBSYSTEM=hdmi", NULL};*.

