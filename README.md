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
This root module is the first hardware component designed for the Agathis 
Gateway. It is the central processing unit build around a Linux 
running microprocessor.  The root is build to support robust stacked systems, 
sharing a vertical interconnect trunk with the branch modules. The tree system
is powered by an integrated stacked module. The root integrates a tree port 
Ethernet switch whereas the branch modules are specifically designed to interface 
with any communication medium. 

###Motivation: 

- Need to connect things with open source hardware. Not much to 
  say about it; something is a mess out there and something is missing. The 
  current PC oriented architecture was not ment for the job.

### Rationale:
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

- We are building a system that easily adapts to harsh environmental 
  conditions, provides scalability and modularity in field conditions and 
  IS OPEN SOURCE and wide open or optiamal custom solutions.

- First module of this gateway is this root module, as covered by herein.

### Compatibility

At the time of editing this document:
- A standard/specification for the trunk electrical interface is in the works.
- A standard/specification for the mechanical interface is yet to be started.
- A standard/specification for the electrical interface with the power module is
  yet to be started.

## Design

**Design goals and overall constraints and requirements:**
- use devices with temperature range -40 to 85C or beyond; use of device
  variants manufactured for reduced temperature range such as 0-70C shall be 
  limited to prototypes for functional development; product variants for lower
  temperature ranges shall be the exception and marked as such.

- strong preference for the use of devices with full datasheet and support 
  available free of NDAs.

- root size excluding connectors; square with side less than 85mm

- the trunk connector needs to support speeds higher than 5Gbps per 
  differential line.

- limit total system power to 13.5W.

- turn-off capability for all interfaces that may not be used.

- optimize EMC with consistent margin against most stringent standards to 
  allow operation in harshest electrical conditions.

- capable to operate from Li-Ion 1-cell battery.

- capable to operate from USB-OTG port power supply for development and 
  field servicing purposes.

- support operation from PoE (Power over Ethernet).

- capable to run a main stream Linux distribution.

- include Trusted Platform and Crypto-Authentication hardware solutions.

- provide hardware solution for host authentication when operating as device 
  on USB-OTG port.

### Power management
#### Power strategy

- **the Gateway must be seen as a battery operated system with opportunistic 
  access to other power sources.** This perspective helps to design a power 
  efficient gateway with extendend availability.

- uC and uP must be programmed for lowest power consumption; such us turn off 
  the christmas lights - nobody is watching anyway.
- VSYS and VSB3P3 rails are always on; use with care; pay attention to all 
  bias and leakage currents.

- hardware power states: 
  - **power-down**
	- VSB3P3 below minimum level to safely operate the uC.
	
  - **stand-by:**
    - VSB3P3 is at a safe operating level.
    - uC - watching its internal real time clock and trunk interface for a 
	wake-up/interrupt event.
	- V3P3, V1P8 and VCORE regulators are turned off (uP and its external 
	interfaces are powered down).
	- uC monitor VSYS to detect imminent power failure and turn-off all 
	remaining devices before full hardware shut-down occurs.
	
  - **active:**
    - uP is on.
	- all unused interfaces are turned off.

***
	
#### Tree Power Distribution:
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/TreePowerDistribution.PNG)

- The gateway is supplied by the power module which adapts the **DC input** 
  power to feed the **VSYS** rail or to charge a **back-up battery**.

- **VSYS** can be sourced by a *Power over Ethernet* block which can charge
  the battery as well; this PoE stage picks up the DC power feeds from the 
  Ethernet ports on root; use existing ethernet magnetics circuits - specified 
  for PoE+ operation.

- **VSYS** serves as the bulk power distributed to the gateway.

- **VSYS** may occasionally feed from USB-OTG port on root for field servicing 
  or lab development work.

- **The power module is mandatory to operate the gateway** - same way as a PC 
  motherboard needs a power adapter to run.
  
- **V1P8** and **VCORE** are local distribution rails for the root alone.

- **V3P3** is a power rail that is **ON** only for an **active** gateway; 
  it supplies the the root and branches **data interface circuits**.

- **VSB3P3** is a power rail that is always **ON**; it supplies root and the 
  branches **control interface circuits**.

- **USB-OTG** port may either: 
  - **sink power** when configured as USB Device. 
  - **source power** when configured as USB Host.
  
- The specifications for the trunk power rails are governed by the [AP-7 
  Agathis Trunk Standard.](https://github.com/agathis-project/pinus-rigida)

***
  
#### VCORE and V1P8 Regulator:
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/VCORE_and_V1P8_regulators.PNG)

- **V1P8** rail supplies followings:
  - uP
  - LPDDR memory
  - eMMC memory
  - Ethernet Phy devices

- **VCORE** rail supplies the uP (V_CORE and V_MPU connected together)

- **V1P8 and VCORE** regulators feed from VSYS through FB5, C48, C54 filters.

- These regulators are implemented with FAN5355 buck converter fabricated by 
  Fairchild/On Semi; they can be controlled over i2c; two different models 
  with distinct i2c addresses are used.

- **V1P8 and VCORE** regulators are synchronized with SYNC3M clock running at 
  nominal 3MHz and phased at 180deg to reduce the switching noise injected
  into the VSYS rail.

- **VCORE** regulator output can be switched signal between two 
  voltages (programmable over i2c) using **VSEL** signal: 
   - the default start-up output voltage for **VSEL = "low" is 1.05V** and can 
   be used as such for the initial power-up of the uP.
   - the default start-up output voltage for **VSEL = "high" is 1.2V and MUST 
   BE ADJUSTED** before use; if there is no need to switch the output
   voltage during normal operation, then the second voltage must be set to same
   value as first (safer operation).

- **V1P8 and VCORE** regulators are enabled by the **EN-VCORE and EN-V1P8** 
  signals controlled by the uC.

- the two regulators offer a light load operation mode; this mode should be 
  made available by the uC firmware to test its relevance for the system 
  performance.
  
- the PWRCLK signal is forwarded to the trunk interface to synchronize SMPS
  regulators on branches.
  
- **CORE-MON** is a remote sense signal that provides the VCORE regulator 
  feedback voltage directly from the microprocessor die.

***

#### V3P3 Regulator:
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/V3P3_regulator.PNG)

***

#### VUSB Regulator:
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/VUSB_regulator.PNG)

#### Locking switching frequency for VCORE, V1P8, V3P3 and VUSB regulators:
- VSYS power rail is shared among all branches in a tree; each branch that feeds 
  from this line may introduce noise into it that needs to be limited; worst 
  offenders are the switching mode power supplies.
- A simple method to reduce the ripple noise is to synchronize or lock the 
  switching frequency of the SMPS regulators feeding from VSYS:
  1. ripple frequency is constant and derived from same controllable source; 
     this means easier filtering; the microcontroller may even implement a 
	 spread spectrum clock if needed.
  2. this design allows to control the phase of the ripple to avoid peak 
     overlapping; this means lower ripple noise injected into the system.
  3. each branch using the PWRCLK synchronization signal can be designed to 
     add a small fixed delay to the PWRCLK fed to the next branch; in this way 
	 each branch regulator will switch at different time lowering the overall 
	 noise injected into VSYS rail.
  
- drive MSYNC3M at 2.4MHz 
- drive SYNC3M at 3MHz
- generate MSYNC3M and SYNC3M from a common 12MHz clock.  

#### Power Module Connector and POE Connectors
+++ insert schematic detail

#### Power Distribution on Trunk Connector

- VSYS    connected to pins A48-50; filtered with FB4(ferrite bead 220 Ohm @1ooMHz) C53(220R) and C106 (100n)
- VSB3P3  connected to pins A47
- V3P3    connected to pin  A6
- GND     connected to pins A32,A45,B5,B7,B16,B18,B20,B25-26,B38,B41,B44,B47,B50
   - GND is multipurpose:
      - shared as return path for VSYS,VSB3P3,V3P3 and all digital signals 
      - shield/return path for high speed controlled impedance signals (USB)
	  - shield/return path for high speed clocks (SPI and SDIO clocks)

### Microprocessor
Use AM3356BZCZA80 by TI.
This is an ARM Cortex A-8 32bit RISC Processor running at max 600MHz and is 
specified over an extended industrial temperatures range of -40C to +105C. 
It includes two Programmable Real-Time Units (PRUs) 32-Bit Load/Store RISC 
processor capable of running at 200 MHz.

#### Power, Boot, Reset and Clocking

##### Power-up and Power-down
The microprocessor uses 3 power rails (V3P3, V1P8 and VCORE) controlled by the 
microcontroller.

- Relevant uP settings:
  - uP internal RTC block is disabled
  - uP internal RTC LDO regulator is disabled

***

##### power-up sequence:
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/AM335x_powerup.PNG)
- power down sequence:
  1. turn off CLK_M_OSC by asserting PWRONRSTn low.
  2. turn off power rails in reverse order referenced to power-up order.
  
##### Boot Configuration

    Signals translation table:

    uP Config Pin   uP Pin Name   Trunk Signal   uC Port
    =============   ===========   ============   =======
    SYSBOOT.15	    LCD_DATA15    QC.1           PK7
    SYSBOOT.14      LCD_DATA14    QC.0           PK6
    SYSBOOT.13      LCD_DATA13    QB.1           PK5
    SYSBOOT.12      LCD_DATA12    QB.0           PK4
    SYSBOOT.7       LCD_DATA7     GPIO.11        PD5
    SYSBOOT.6       LCD_DATA6     GPIO.10        PB3
    SYSBOOT.5       LCD_DATA5     GPIO.9         PB7
    SYSBOOT.4       LCD_DATA4     GPIO.8         PQ3
    SYSBOOT.3       LCD_DATA3     GPIO.3         PD4
    SYSBOOT.2       LCD_DATA2     GPIO.2         PB6
    SYSBOOT.1       LCD_DATA1     GPIO.1         PB4
    SYSBOOT.0       LCD_DATA0     GPIO.0         PC7

- SYSBOOT signals are latched on rising edge of PWRONRSTn signal:
  - disable any driver connected to SYSBOOT signal for at least 10us before and 
    10us after rising edge of PWRONRSTn.
  - microcontroller drives the appropriate SYSBOOT configuration before driving
    the rising edge of PWRONRSTn and releases the lines immediately after.

**SYSBOOT signals boot configuration**

    SYSBOOT.15,14	    fixed           01    == 24MHz (crystal frequency)
    SYSBOOT.13,12       fixed           00    == mandatory by specs
    SYSBOOT.11,10,9,8   don't care      XXXX    (not controlled by uC)
    SYSBOOT.7,6         fixed:          00    == select MII for EMAC1
    SYSBOOT.5	        fixed:			0     == CLK1 OUT disabled
	SYSBOOT.4,3,2,1,0 	configurable:   00001 == UART0,_,MMC0,SPI0
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

	Legend:	
	=======
	UART0 = uP UART0_RXD(pin18@J3) and UART0_TXD(pin17@J3) on Test Extension Card J3 Connector
	MMC0  = uSD-Card J2 connector
	MMC1  = eMMC U14
	SPI0  = SPIA on trunk J1 connector
	EMAC1 = MII1 (Ethernet Port 1) J5 RJ45 rear connector
    USB0  = USB-OTG on external J6 connector
	_     = unavailable device 
		
***
		
#### Branch Control Interfaces:
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/uP_branch_control.PNG)


***

#### Branch Data Interfaces:
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/uP_trunk_data_circuits.PNG)


***

#### Branch USB Interfaces


#### LPDDR Memory:
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/uP_LPDDR.PNG)

***

#### eMMC and SD-Card Memories:
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/uP_emmc_sdcard.PNG)
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/emmc_sdcard.PNG)

***

#### Ethernet:
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/uP_mii.PNG)
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/eth_phy.PNG)


***

#### uP Clocks, Reset, JTAG and UART0 Signals:
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/uP_rst_jtag_clk_uart0.PNG)


### Microcontroller
#### Power, Reset and Clocking
+++ insert schematic detail

#### Root ID Eeprom
+++ insert schematic detail

#### Branch Control
+++ insert schematic detail

#### I2C buses
+++ insert schematic detail

***

#### Trusted Platform Module:
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/tpm.PNG)

***

#### Crypto Authentication:
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/Crypto_auth.PNG)

### USB-OTG and USB Host
+++ insert schematic detail

***

### Trunk Connector:
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/Trunk_connector.PNG)

***

### Test Extension Connector:
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/Test_extension_card_connector.PNG)

## Schematic 
[The schematic](https://github.com/agathis-project/salix-arctica/blob/master/V1/SCH-1-1-1.pdf)
for this design was captured in Altium and is available as pdf and project 
file package in V1 folder of this repo.


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