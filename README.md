# PD OmniTool

>  Software Defined, Flexible Power Routing, USB-C + DC Hub Open Source Hardware Device

By Jaunty Wunderkind <jaunty+wunder+kind+pd+omnitool@eldergods.com>  
Draft, April 23rd 2018

USB-C's arrival begins a new wave of perhipheral connectivity for computer users, one that grants devices the flexibility to send or receive power to what they are connected to. Thusfar, the use-cases permitted seem to be largely limited to two basic scenarios: a device plugged in to a wall charger or battery unit, or a device transfering power to another device.

The Power Delivery OmniTool design proposed here aims to pioneer a drastically more open frontier for moving managing and monitoring power & transfer across multiple devices, both as a completely software-defined forward-specification-compatible richly-informative practical hub device with which to connect a variety of devices & power systems, and as a scalable, flexible reference design for talking to USB Power Delivery devices. If successful, the PD OmniTool will serve as a ductile, future-proof charging solution for power providing & consuming devices & power systems, & will give future makers a tool & means to reproduce their own spin-offs & experiments with USB Power Delivery & power conversion.

# Table of Contents

   * [PD OmniTool - Software Defined Flexible Power Routing USB-C   DC Hub](#pd-omnitool---software-defined-flexible-power-routing-usb-c--dc-hub)
   * [Table of Contents](#table-of-contents)
   * [The Challenges of Power Delivery Today](#the-challenges-of-power-delivery-today)
   * [Power Delivery OmniTool - Flexible Power Routing](#power-delivery-omnitool---flexible-power-routing)
      * [Overview](#overview)
      * [Specifications](#specifications)
   * [Design](#design)
      * [Ports](#ports)
         * [Load switches](#load-switches)
         * [Analog CC](#analog-cc)
            * [Discrete Analog](#discrete-analog)
            * [FPGA](#fpga)
      * [DC-DC Converters](#dc-dc-converters)
      * ["Brains"](#brains)
      * ["Primary" Port](#primary-port)
         * [Data Pass-through](#data-pass-through)
   * [Additional Concerns](#additional-concerns)
   * [Alternative And Enhancements Discussion](#alternative-and-enhancements-discussion)
      * [Per Port Buck](#per-port-buck)
      * [Per-port "controller"](#per-port-controller)
      * [Data Hub](#data-hub)
   * [Appendix - Writing Guide](#appendix---writing-guide)
      * [Open Hardware Design Challenge](#open-hardware-design-challenge)
      * [Design docs](#design-docs)

# The Challenges of Power Delivery Today

USB-C and USB Power Delivery permits bidirectional power transit, yet few options exist to be able to route power around in USB effectively. Currently users rely on re-plugging laptops, phones, and wall chargers making dedicated, 1:1 connections with each other. In an age when the one connector can bring power and high speed data, users are limited to inflexible single purpose charging solutions. Even the most basic needs are not met in the Power Delivery ecosystem: there are presently no multi-device charging systems available! Yet users should be able to expect connecting a variety of their devices and battery backs, & the freedom to shuffle power across them.

In addition, maker's lack the experience and proven competence to work with USB Power Delivery directly is a crucial problem in the USB Power Delivery world. Beyond the practical convenience burden that the lack of flexible power architecture creates, would be makers are hampered by the overwhelming array of closed USB power delivery controllers designed for express implementability, at the tradeoff of having prepackaged limited feature sets. Makers have limited access to work with USB Power Delivery when they go through these solutions, and often face design limitations imposed by purpose-built controllers which largely do not cater to niche behaviors makers might be interested in.

Last, end users have very little visibility in the current USB-PD ecosystem as to how and what is happening. Purchasing a USB Power Delivery sniffer requires well over $100 and provides only the most basic protocol-level view of the complex, realtime interacitons happening across USB Power Delivery.

These three factors express the desire and need for the PD OmniTool:
* the lack of a flexible power routing device,
* the lack of existing open hardware designs to enable more USB Power Delivery designs,
* the lack of a observability to learn about USB Power Delivery.

PD OmniTool envisions a design able to inform, enable, and empower users and makers about the full range of interesting, sophisiticated capabilities that Power Delivery grants, while creating a practical, ridiculously flexible hub device.

# Power Delivery OmniTool - Flexible Power Routing

## Overview

With these high challenges in the industry, PD OmnitTool envisons a flexible & enduring architecture for interconnecting DC & USB Power Delivery devices in a radically-flexible (via hardware architecture) forward-spec-compatible (via uniquely software defined implementation) way. The PD OmniTool ties a number of ports (6 in the reference design: 4 USB-C and 2 DC) together, flexibly switching ports onto any of three switchable buses: a power providing input bus, and two variable voltage output buses. High speed data pass-through capabilities are provided to allow expected data-connection operations to continue independent of power transfer operations. Software defined architecture insures radical forward compatibility & manipulability unmatchable by off-the-shelf USB Power Delivery Controller chips.

The PD OmniTool should, for many users, serve as an adequate multi-device charging hub, powered by a high power wall wart device (design aims for at least 30V 10A). However the OmniTool's flexible architecture means it should be suitable for a far wider range of sophisticated power operations, suitable in across a huge variety of uses. In off-the-grid situations it ought be a capable solar & battery anchored hub, powering devices while charging or discharging batteries. In maker/electronics usage, it ought be capable of performing voltage or current controlled output to any of it's ports (either as spec permits or manually setting port output, ignoring USB Power Delivery protocols).

Although data is not the primary consideration of this product, one dedicated "master" USB port is assigned, that provides full high-speed pass-through that can be switched between one of two other ports. This allows a laptop to connect to a flash drive or alt-mode display or other example device, while taking advantage of the power hub.

## Specifications

* fully able to support up to 30V minimum from inputs
* draw of 10A minimum on input
* 2 high efficiency boost-buck DC DC converters powering 2 independent bus rails
* 4 USB-C ports, each with complete Configuration Channel (CC) analog front-end (AFE) phy
  * cc phy allows software-defined negotiation of USB-PD, including PD 3.0 & (likely) beyond
  * one port is designated the "primary" port, with advanced capabilities
    * can data-pass through to one of two other usb-c ports
    * can enter "sniff" mode with one specific usb-c port, tying their cc and vbus lines together but monitoring them
* 2 DC ports, for battery, wall, solar, or other electronics (eg: motor, or led drive) connections
* All ports able to switch between power consuming, and power providing roles (on any power bus)
* extensive voltage/current monitoring throughout
* Indeterminate "brains" - perhaps a small ARM core, or a common interface (Adafruit Feather, Pi Zero), or perhaps an onboard very low power ICE40 fpga.

# Design

The PD OmniTool reference design is comprised of three main systems;

1. A repeated series of "ports", either DC or USB-C, which use load-switches to tie to input our output buses. Each port also has monitoring hardware associated with it to check it's status (current, voltage).
2. Two DC-DC integrated boost-buck converters, offering two indepedent extremely-stable very flexible power conversion systems to provide output power to devices.
3. A control subsystem for managing the OmniTool.

In addition, some auxiliary systems are provided:

1. One port is designed a "primary" USB-C port, and has additional capabilities provided:
   a. The primary port can "pass through" data connections to either of two other ports, allowing high speed interconnect via USB 3 or alt-modes
   b. The primary port can "pass through" USB-PD and vbus connections to one specific port, allowing the PD OmniTool to act as a "sniffer" device between these two.

## Ports

Each of the six ports on the reference design follows a repeated template, primarily concerning a series of load switches to connect or disconnect it from various buses, as well as a monitoring chip to observe port behavior (current, voltage). USB-C ports additionally have a "Analog Configuration Channel" line they must speak, which constitutes a sizable and notable piece of the design.

### Load switches

Load-switches are used in this design extensively. Each port needs to be able to switch a high power (20V 5A) connection between an input to the PD OmniTool, or to connect it to either of two output buses. This could end up being discrete components, or perhaps a mutli-channel high-side switch could be used, for example, to connect output rails to ports. 

### Analog CC

On a USB-C connection, USB Power Delivery is negotiated on a special line called the Configuration Channel (CC). This is a low-voltage biphase mark coding signal, operating 4b5, that operates at 300 kHz. All four USB-C ports on the PD OmniTool have additional circuitry to provide these negotiating capabilities.

The PD OmniTool boasts a design here focused on open, adaptable, forward-compatible, observable operation. It does this by implementing it's own analog front-end PHYsical interface, per port. That is, the software brain of the PD OmniTool reads and writes message directly off & to the bus.

This approach contrasts many mainstream devices, which often rely on a Power Delivery "controller" that is often hardwired to perform a limited set of Power Delivery negotiations ahead of time.

As opposed to fixed controller hardware, the software based OmniTool should be able to, over time, increase the breadth & scope of messages it is capable of sending, staying current with whatever evolutions occur in the USB Power Delivery specification. We've recently seen Power Delivery 3.0 introduced, adding a raft of new status messages. Rather than rely on a new, but fixed controller which can negotiate some subset of these messages, the software defined system of PD OmniTool ought permit any message the user wishes to implement, in a manner the user wishes to implement. If the user encounters a proprietary vendor extension, they have the possibility to read & perhaps respond to that extension. If the Power Delivery specification sees a new revision to 3.1 or 4.0, the PD OmniTool will be able to adapt.

This is one of the key areas of innovation for the PD OmniTool. It aims to give the user complete ability to tinker with the USB Power Delivery specification at an entirely intimate level, granting as much freedom as possible. As well as allowing for forward- compatability, this is chosen to keep the system maximally open, to increase observability & user's connection with what is possible.

Two implementation strategies exist:

#### Discrete Analog

Existing designs for using a microcontroller to speak USB-PD CC exist. For example, ST Microelectronic's [P-NUCLEO-USB001 reference design](http://www.st.com/en/evaluation-tools/p-nucleo-usb001.html) uses primary mosfets to perform the voltage conversion from micro-controller level outputs to the low voltage differential signalling used in USB-Power Delivery. The PD OmniTool design could borrow from these available reference implementations to create the complete analog front-end needed, in a very similar fashion to these designs.

The downside of this approach is that it is component heavy. Each port requires it's own dedicated level shifting to operate.

#### FPGA

Initial read of USB Power Delivery specification indicate that it's signalling constraints are remarkably close to the behavior of many differential low voltage signals on common FPGAs. Some testing of popular open-hardware FPGA chips such as an ICE40 ought confirm whether or not sending & receiving CC messages directly via FPGA is possible.

If indeed direct interface via FPGA does work, it would be an ideal situation:
1. It would allow for the design to conceivably scale to well over a dozen ports with little more board space & cost than what is needed by each USB-C connector
1. The FPGA could handle the 4b5 & Biphase Mark Coding translations, offloading that work from the CPU.
1. It would allow the controller CPU to sleep, while the low powered FPGA handles listening, sending, & encoding.
1. Additional control logic could be offloaded to the FPGA.

Candidate parts: [Lattice UP3K ICE40 UltraPlus](http://www.latticesemi.com/Products/FPGAandCPLD/iCE40Ultra)

## DC-DC Converters

Inside the design, two boost-buck converters provide for flexible power conversion from the input bus to the desired bus status. Two buses are provided to allow for some flexibility- perhaps some devices can only charge at 9V while other devices would like to be able to charge 20V. Or perhaps one bus could be connected to a DC port, and on the fly adjusted so as to provide current-mode charging to a battery pack.

CAndidate parts: [Linear LTC7812 boost-buck converter](http://www.analog.com/en/products/power-management/pmic-multifunction/multitopology-dc-dc/ltc7812.html)

## "Brains"

Software control of this flexible power system is of high importance, as there are relatively few hardware safeguards in place. It's up to the control system to monitor the myriad onboard sensors to insure safe operating conditions, guarding against undervoltage, overvoltage, & overcurrent conditions, while also performing USB Power Delivery negotiation over CC.

Already mentioned under port design, use of a FPGA as the central control could be an ideal inbetween, offering a flexible, low power, real time control system that can be programmed via serial or i2c links.

Alternatively, control could be provided by an onboard CPU. A small ARM Cortex M0 or RISC-V core could well prove able and sufficient for running the required tasks such as the relatively low-speed CC links & sensor handling. More advanced CPUs such as a Nordic nRF or Espressif ESP32 - in easy to use module form - could provide cost-appropriate "brains" that also offer the potential for interaction over bluetooth low energy, which could open up a number of interesting scenarios for experimenters.

The author has a personal interest in running [Zephyr](https://www.zephyrproject.org), if a CPU-based approach is chosen. This provides a solid, very actively developed embedded OS, that features advanced wireless connectivity options.

Ideal parts:
* Lattice UP3K
* Atmel ATSAMD21
* Nordic nrf5240
* Espressif ESP32

## "Primary" Port

The "primary" port of the controller functions very much alike every other port, but it is granted some additional capabilities:

### Data Pass-through

The "primary" port can connect it's high speed data bus to either of two other USB-C ports. To do this, a standard USB-C and USB-2 2:1 switch is used. This allows the port to connect to either of two other ports. One might plug their laptop into the primary connection, and a monitor and a backup hard drive into the two other ports. While the user couldn't then use both of these peripherals, they could switch between the two.

Note that there is no directionality to this connection. One could plug a flash drive in to the "primary" port and send commands to the hub to switch the drive between two connected laptops.

See [Data Hub](#data-hub) for more discussion on possible improvements here.

Ideal part: [NXP PTN36043 active switch](https://www.nxp.com/products/analog/interfaces/usb-interfaces/usb-type-c-true-plugn-play/usb-redrivers/ptn36043-usb-type-c-superspeed-active-switch:PTN36043BX)

# Additional Concerns

Bootstrapping this system presents some significant design complexity, given the wide input voltages supported. There are wide input range small power buck converters available to help get, say, a modest current 3.3V bus to run on.

# Alternative And Enhancements Discussion

Some possible spins on this idea are included below:

## Per Port Buck

Evaluating the space and cost requirements of the "multiple bus" topology presented above versus a design that uses a single dedicated buck converter per output has the possibility to end up with a cheaper or smaller (or both!) design that could have more flexibility (currently PD OmniTool design is limited to two output voltages maximum; this could permit 6!).

Candidate parts: [Vishay SiC403A integrated microbuck](http://www.vishay.com/docs/62768/sic403abcd.pdf)

## Per-port "controller"

Most USB-C designs rely on "controller" chips to negotiate power. Frequently they come with their own dedicated DC-DC capabilities as well. These are compelling chips, but they often have limited operational modes, and come with a high cost. None the less, designs based around these chips are worth evaluating. Note that one is required for each port.

Candidate parts: [ISL95338](https://www.intersil.com/en/products/power-management/battery-management/multiple-cell-battery-chargers/ISL95338.html)

## Data Hub

Current data connectivity within the PD OmniTool is limited. This was largely done to limit scope, on account of this already being a complex design, and on account of high speed data connectivity being an ambiguous goal.

In a "simple" case, the existing "primary" port of the PD OmniTool could be attached to an onboard USB hub, either USB 2 or even USB 3. These connections to other ports would have to be switched, as one of the primary purposes of USB-PD is to allow charging multiple devices, yet if a user plugs in two laptops, the second laptop would likely not expect to be plugged in as a device on a switch.

Given far more resources, it would be fun to consider an altogether more flexible architecture for data routing across the PD OmniTool. There are quite a lot of possible combinations for connecting ports, likely enforcing some practical limits to such a design.

Candidate parts: [NXP CBTL08GP053 Type-C Crossbar](https://www.nxp.com/products/analog/interfaces/usb-interfaces/usb-type-c-true-plugn-play/high-speed-signal-switches/usb-type-c-high-performance-crossbar-switch-ic:CBTL08GP053EV)

# Appendix - Writing Guide

This section includes reference material used to aid in the preparation of this design.

## Open Hardware Design Challenge

This section repeats the the official prompt for the [Open Hardware Design Challenge competition](https://hackaday.io/prize/details#one) that this document is pursuing.

Requirements:

1. Discuss the challenge the project addresses
2. Discuss how hte project will alleviate or solve the problem that the project addresses
3. Publish at least one image illustrating how the project might be used. This may be a sketch, schematic, flow chart, rendering, or other type of image
4. Link to any repositories (Github)
5. Document all open-source licenses and permissions as well as any applicable third-party licenses/restrictions
6. Submit the Project to 2018 Hackaday Prize using the "Submit projet to" option.

## Design docs

[Malte Ubl's Design Docs - A Design Doc](https://medium.com/@cramforce/design-docs-a-design-doc-a152f4484c6b) outlines a number of characteristics a design doc (generally for software but used here too) ought accomplish. This is taken as significant guidance in the writing of the PD OmniTool design document.

1. Document the software design
2. Clarify the problem being solved
3. Act as a discussion platform to further refine the design
4. Explain the reasoning behind those decisions and tradeoffs made in that decisions
5. List alternative designs and why they were not chosen
6. Support future maintainers and other interested parties in understanding why the original design was chosen

Steps of a design doc:

1. meta-information
2. context, scope, goals
3. overview
4. detailed design
5. coss-cutting concerns
6. alternatives
