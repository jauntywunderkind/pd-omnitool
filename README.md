# PD OmniTool - Software Defined Flexible Power Routing USB-C + DC Hub

By Jaunty Wunderkind <jaunty+wunder+kind+openpowerhuhb@eldergods.com>
Draft, April 23rd 2018

USB-C's arrival begins a new wave of perhipheral connectivity for computer users, one that grants devices the flexibility to send or receive power to what they are connected to. Thusfar, the use-cases permitted seem to be largely limited to two basic scenarios: a device plugged in to a wall charger or battery unit, or a device transfering power to another device.

The Power Delivery OmniTool design proposed here aims to pioneer a drastically more open frontier for moving managing and monitoring power & transfer across multiple devices, both as a completely software-defined forward-specification-compatible richly-informative practical hub device with whichto connect a variety of devices & power systems, and as a scalable, flexible reference design for talking to USB Power Delivery devices. If successful, the PD OmniTool will serve as a ductile, future-proof huband charging solution for power providing & consumingdevices, & will give future makers a tool & means to reproduce their own spin-offs & experiments with USB Power Delivery & power conversion.

# TOC

* [PD OmniTool - Software Defined Flexible Power Routing USB-C   DC Hub](#pd-omnitool---software-defined-flexible-power-routing-usb-c--dc-hub)
* [The Challenges of Power Delivery Today](#the-challenges-of-power-delivery-today)
* [Power Delivery OmniTool - Flexible Power Routing](#power-delivery-omnitool---flexible-power-routing)
   * [Overview](#overview)
   * [Specifications](#specifications)
* [Design](#design)
   * [Ports](#ports)
      * [Analog CC](#analog-cc)
      * [Load switches](#load-switches)
   * [Rails](#rails)
   * [Repeatable Port and Bus-bar Architecture](#repeatable-port-and-bus-bar-architecture)
* [Alternative And Enhancements Discussion](#alternative-and-enhancements-discussion)
   * [Per Port Buck](#per-port-buck)
   * [Data Hub](#data-hub)
* [Writing guide](#writing-guide)
   * [Open Hardware Design Challenge](#open-hardware-design-challenge)
   * [Design docs](#design-docs)



# The Challenges of Power Delivery Today

USB-C and USB Power Delivery permits bidirectional power transit, yet few options exist to be able to route power around in USB effectively. Currently users rely on re-plugging laptops, phones, and wall chargers making dedicated, 1:1 connections with each other. In an age when the one connector can bring power and high speed data, users are limited to inflexible single purpose charging solutions. Even the most basic needs are not met in the Power Delivery ecosystem: there are presently no multi-device charging systems available! Yet users should be able to expect connecting a variety of their devices and battery backs, & the freedom to shuffle power across them.

In addition, maker's lack the experience and proven competence to work with USB Power Delivery directly is a crucial problem in the USB Power Delivery world. Beyond the practical convenience burden that the lack of flexible power architecture creates, would be makers are hampered by the overwhelming array of closed USB power delivery controllers designed for express implementability, at the tradeoff of having prepackaged limited feature sets. Makers have limited access to work with USB Power Delivery when they go through these solutions, and often face design limitations imposed by purpose-built controllers which largely do not cater to niche behaviors makers might be interested in.

Last, end users have very little visibility in the current USB-PD ecosystem as to how and what is happening. Purchasing a USB Power Delivery sniffer requires well over $100 and provides only the most basic protocol-level view of the complex, realtime interacitons happening across USB Power Delivery.

These three factors express the desire and need for the Open Power Hub:
* the lack of a flexible power routing device,
* the lack of existing open hardware designs to enable more USB Power Delivery designs,
* the lack of a observability to learn about USB Power Delivery.

Open Power Hub envisions a design able to inform, enable, and empower users and makers about the full range of interesting, sophisiticated capabilities that Power Delivery grants, while creating a practical, ridiculously flexible hub device.

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
  * one port is designated the "primary" port, & can data-pass through to one of two other usb-c ports
* 2 DC ports, for battery, wall, solar, or other electronics (eg: motor, or led drive) connections
* All ports able to switch between power consuming, and power providing roles (on any power bus)
* USB "master" whose high-speed connections passes-through to one of two other connected devices
* extensive voltage/current monitoring throughout
* Indeterminate "brains" - perhaps a small ARM core, or a common interface (Adafruit Feather, Pi Zero), or perhaps an onboard very low power ICE40 fpga.

# Design

The PD OmniTool 

## Ports

Each port follows a repeatable architecture, primarily concerning a series of load switches to connect or disconnect it from various buses, and a monitoring chip to observe port behavior.

### Analog CC

USB-C ports have additional circuitry required to negotiate their status with the connected device, across the Configuration Channel wire of the USB-C port.

On USB-C ports, an analog front-end PHY design is put in place to interface with the Configuration Channels (CC) used to negotiate USB Power Delivery status. This insures forward-compatibility. For example Power Delivery 3.0 was recently introduced, vastly expanding the variety of messages devices can communicate. Because the 

### Load switches

Load-switches are used in this design extensively. Each port needs to be able to switch a high power 

## Rails

Inside the design, two buck-boost converters provide for flexible power conversion from the input bus to the desired bus status. Two buses are provided to allow for some flexibility- perhaps some devices can only charge at 9V while other devices would like to be able to charge 20V. Or perhaps one bus could be connected to a DC port, and on the fly adjusted so as to provide current-mode charging to a battery pack.

## Repeatable Port and Bus-bar Architecture
The Open Power Hub design envisions a single repeatable tile of a single USB-C "port", connected to two high power internal bus bars.



# Alternative And Enhancements Discussion

## Per Port Buck

## Data Hub

# Writing guide

## Open Hardware Design Challenge

[This](https://hackaday.io/prize/details#one) is the official prompt for the Open Hardware Design Challenge competition this document is pursuing.

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
