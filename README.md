    AP#:           1
    Title:         Root Module HW
    Type: 	       HW Development
    License:       TAPR
    State: 		   Verification [editing in progress]
    Author: 	   md-agathisproject
    Issue-Tracker: https://github.com/agathis-project/salix-arctica/issues
    Product Codes: PCB-1-(1)-1, PCA-1-(1)-1

	
# AP-1 Root Module HW

## Idea
### Abstract
The root module is the first hardware component designed for the Agathis 
Gateway. It is the central processing unit of the gateway build around a Linux 
running microprocessor.  The root is build to support robust stacked systems 
sharing a vertical interconnect trunk with the branch modules. The tree system
is powered by an integrated stacked module. The root integrates a tree port 
ethernet switch whereas the branch modules are custom designed to interface 
with any communication medium. 


### Motivation and Rationale
####Motivation: NEED TO CONNECT THINGS WITH OPEN SOURCE HARDWARE

#### Rationale:
- By large, the communication interface for majority of things is one of the 
  basic physical layer interfaces such us: UART, SPI, I2C, SDIO, USB and the 
  ubiquitous GPIO.
  
- Even though todays microprocessors have these interfaces available, 
  there is no open architecture supporting a modular scalable construction 
  similar to what once PCI did with its implementation variants did for PC 
  industry, but this time applied to the new set of interfaces listed above.

- Majority of things are out-there, not necessarily in controlled housed
  conditions; they need to deal with scarcity of power supply, harsh 
  electrical and mechanical conditions, wide temperature and humidity range and
  corrosive mediums. 

- We are building here a system that easily adapts to such conditions, provides
  scalability and modularity in field conditions and is OPEN SOURCE.

- First module of this gateway is the root module, as covered by this document.


### Compatibility

There is no compatibility concern on root design; the burden of compatibility 
is with all other modules designs.

At the time of editing this document:
- A standard/specification for the trunk electrical interface is in the works.
- A standard/specification for the mechanical interface is yet to be started.
- A standard/specification for the electrical interface with the power module is
  yet to be started.

These standards are essentially derived from the root design.

So, be careful with the root design !


## Design

- design goals and overall constraints and requirements:
  - use devices with temperature range -40 to 85C or beyond; use of device
  variants manufactured for reduced temperature range such as 0-70C shall be 
  limited to prototypes for functional development; product variants for lower
  temperature ranges shall be the exception and marked as such.
  - strong preference for the use of devices with full datasheet and support 
    available free of NDAs.
  - root size excluding connectors; square with side less than 85mm
  - the trunk connector shall be capable to support speeds in excess of 5Gbps 
  per differential line.
  - total system power is limited 13.5W
  - capable to turn-off not used interfaces.
  - optimize EMC with consistent margin against most stringent standards to 
  allow operation in harshest electrical conditions.
  - capable to operate from Li-Ion 1-cell battery.
  - capable to operate from USB-OTG port power supply for development and 
  field servicing purposes.
  - support operation from PoE.
  - capable of running a main stream Linux distribution.
  - include Trusted Platform and Crypto-Authentication hardware solutions.
  - provide hardware solution for host authentication when operating as device 
  on USB-OTG port.

### Power management
#### Power strategy
- microcontroller and microprocessor to be programmed for lowest power 
consumption.
- turn off all unused features.
- VSYS rail is always on; use with care.
- VSB3P3 rail is always on; use with care.

- power states: 
  - **power-down**
	- VSB3P3 below minimum level to operate the microcontroller.
	- all devices in the system are turned off or instructed to do so by the 
	microcontroller; undisciplined branches may screw-up the root.

  - **stand-by:**
    - VSB3P3 is at a safe operating level.
    - microcontroller runs - watching its internal real time clock and 
	trunk interface for a wake-up event.
	- V3P3, V1P8 and VCORE regulators are turned off (microprocessor and its 
	external interfaces are powered down).
	
  - **active:**
    - V3P3, V1P8 and VCORE regulators are turned on (microprocessor is on).
	- unused interfaces are switched off.

#### Power tree
+++ insert diagram

#### VCORE Regulator
+++ insert schematic detail

#### V1P8 Regulator
+++ insert schematic detail

#### V3P3 Regulator
+++ insert schematic detail

#### VUSB Regulator
+++ insert schematic detail

#### Power Module Connector and POE Connectors
+++ insert schematic detail

#### Power Distribution on Trunk Connector
+++ insert schematic detail

### Microprocessor
The microprocessor used in this variant of the root is AM3356BZCZA80 made by TI.
This is an ARM Cortex A-8 32bit RISC Processor running up to 600MHz specified 
over an extended industrial temperatures range of -40 to 105C. 
It includes two Programmable Real-Time Units (PRUs) 32-Bit Load/Store RISC 
processor capable of running at 200 MHz.

#### Power, Boot, Reset and Clocking

##### Power-up and Power-down
The microprocessor uses 3 power rails (V3P3, V1P8 and VCORE) controlled by the 
microcontroller.

- RTC is disabled
- RTC LDO is disabled
- power-up sequence:
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/AM335x_powerup.PNG)
- power down sequence:
  1. turn off CLK_M_OSC by asserting PWRONRSTn low.
  2. turn off power rails in reverse order referenced to power-up order.
  
##### Boot Configuration

    Signals translation table:

    uP Config Pin   uP Pin Name  Trunk Signal   uC Port
    ==============  ===========  =============  =======
    SYSBOOT.15	    LCD_DATA     QC.1           PK7
    SYSBOOT.14      LCD_DATA     QC.0           PK6
    SYSBOOT.13      LCD_DATA     QB.1           PK5
    SYSBOOT.12      LCD_DATA     QB.0           PK4
    SYSBOOT.7       LCD_DATA     GPIO.11        PD5
    SYSBOOT.6       LCD_DATA     GPIO.10        PB3
    SYSBOOT.5       LCD_DATA     GPIO.9         PB7
    SYSBOOT.4       LCD_DATA     GPIO.8         PQ3
    SYSBOOT.3       LCD_DATA     GPIO.3         PD4
    SYSBOOT.2       LCD_DATA     GPIO.2         PB6
    SYSBOOT.1       LCD_DATA     GPIO.1         PB4
    SYSBOOT.0       LCD_DATA     GPIO.0         PC7

- SYSBOOT signals are latched on rising edge of PWRONRSTn signal:

- disable any driver connected to SYSBOOT signal for at least 10us before and 
10us after rising edge of PWRONRSTn.

- microcontroller drives the appropriate SYSBOOT configuration before driving
the rising edge of PWRONRSTn and releases the lines immediatelly after.

**SYSBOOT signals boot configuration**

    SYSBOOT.15,14	    fixed           01    == 24MHz (crystal frequency)
    SYSBOOT.13,12       fixed           00    == mandatory by specs
    SYSBOOT.11,10,9,8   don't care      XXXX    (not controlled by uC)
    SYSBOOT.7,6         fixed:          00    == select MII for EMAC1
    SYSBOOT.5	        fixed:			0     == CLK1 OUT disabled
    SYSBOOT.4,3,2,1,0 	configurable:   
		
		00001 == UART0,_,MMC0,SPI0
    	00010 == UART0,SPI0,_,_
    	00011 == UART0,_,_,MMC0
    	00100 == UART0,_,MMC0,_,_
    	00101 == UART0,_,SPI0,_
    	00110 == EMAC1,SPI0,_,_
    	00111 == EMAC1,MMC0,_,_
    	01000 == EMAC1,MMC0,_,_
    	01001 == EMAC1,_,_,MMC0
    	01010 == EMAC1,_,_SPI0
    	01011 == USB0,_,SPI0,MMC0
    	01100 == USB0,_,_,_
    	01101 == USB0,_,_,SPI0
    	01110 == RESERVED
    	01111 == UART0,EMAC1,_,_
    	10000 == _,UART0,EMAC1,MMC0
    	10001 == _,UART0,EMAC1,MMC0
    	10010 == _,_,USB0,UART0
    	10011 == _,_,MMC0,UART0
    	10100 == _,_,SPI0,EMAC1
    	10101 == _,MMC0,EMAC1,UART0
    	10110 == SPI0,MMC0,UART0,EMAC1
    	10111 == MMC0,SPI0,UART0,USB0
    	11000 == SPI0,MMC0,USB0,UART0
    	11001 == SPI0,MMC0,EMAC1,UART0
    	11010 == _,UART0,SPI0,MMC0
    	11011 == _,UART0,SPI0,MMC0
    	11100 == MMC1,MMC0,UART0,USB0
    	11101 == RESERVED
    	11110 == RESERVED
    	11111 == _,EMAC1,UART0,_

	UART0 = uP RXD(pin18@J3) and TXD(pin17@J3) on Extenstion Card Connector, 
	MMC0  = SD-Card/connector
	MMC1  = eMMC/BGA
	SPI0  = SPIA on trunk connector
	EMAC1 = MII1 (Ethernet Port 1) J5 RJ45 rear connector
    USB0  = USB-OTG on external connector J6 connector
		

#### Branch Control and Data Interfaces
+++ insert schematic details (CPU IO page)

#### LPDDR Memory
+++ insert AM335x diagram or schematic diagram

#### eMMC and SD-Card Memories
+++ insert schematic detail

#### Ethernet
+++ insert schematic detail

#### JTAG and Debug Signals
+++ insert schematic detail

### Microcontroller
#### Power, Reset and Clocking
+++ insert schematic detail

#### Root ID Eeprom
+++ insert schematic detail

#### Branch Control
+++ insert schematic detail

#### I2C buses
+++ insert schematic detail

#### Trusted Platform Module
+++ insert schematic detail

#### Crypto Authentication
+++ insert schematic detail

### USB-OTG and USB Host
+++ insert schematic detail

### Trunk interface
+++ insert schematic detail

### Test Extension Connector 
+++ insert schematic detail

## Schematic
The schematic for this design was done in Altium.
See it in V1 folder in the repo.

## Layout
The layout for this design was done in Altium.
See V1 folder in the repo.

## Mechanicals
- see embedded stacked enclosure

## Prototype
- include manufacturing details such as BOM, pcb ordering package, pca ordering package

## Verification
- board bring-up
- interfaces test plan and report
- bug report

## Integration
### EMC Compliance Test Plan
### Safety Test Plan
### Hazardous Materials Control Plan

### 
- compliance test plan and report
- interoperability test plan and report (power module, test extension card, branch modules)

## License
This design is licensed under the terms of the TAPR.
The terms of the license are available in the LICENSE.TXT file included in the 
repository.


## Attachments