    AP#:           1
    Title:         Root Module HW
    Type: 	       HW Development
    License:       TAPR
    State: 		   Verification [editing in progress]
    Author: 	   md-agathisproject
    Issue-Tracker: https://github.com/agathis-project/salix-arctica/issues
    Product Codes: PCB-1-(1)-1, PCA-1-(1)-1

	
# Root Module HW

## 1. Abstract
The root module is  the core component of the Agathis Gateway which is build by 
stacking the root module and branch modules, sharing a vertical interconnect 
sub-system. The root integrates a tree port Ethernet switch using two ports to 
connect directly with the external world and one internal port to connect -
through the microprocessor - with a plurality of things over media interfaces 
located on branches. 

## 2. Rationale:
- As this is the first module of the Agathis Gateway, the rationale for this AP
  is made from a system perspective.

**Current technology:**
- By large, the interface used by majority of things is one of the basic 
  physical layer interfaces such us: UART, SPI, I2C, SDIO, USB and the ubiquitous 
  GPIO.

- Although today's microprocessors have these interfaces available for embedded 
  applications, there is no architecture supporting a modular scalable construction.

- A large number of things located in uncontrolled environments; they need to 
  deal with the scarcity of power supply, harsh electrical and mechanical 
  conditions, wide temperature and humidity ranges and corrosive atmosphere.

- Practically, there is no open scalable gateway designed to interface with 
  things in these environments.
  
- Current solutions to interface with things in un-controlled environments use 
  industrial single boards computers assembled in custom solutions with limited 
  or no scalability or field flexibility.
  
**Agathis Gateway solution:** 

- **A stackable field reconfigurable gateway** that can adapt to evolving 
  things and grow with their numbers and diversity.

- **A gateway that talks the language of things**, be it UART, I2C, SPI, SDIO,
  USB, PCIe or even General Purpose IOs.

- **A gateway that can be installed where the things are** from the sea bed to 
  Mars surface, from a floor plant to a deep mining operation. 

- **An open source gateway** - that supports competitiveness and fast 
  development of new interfaces with things.
  
- First module of this gateway is the root module, as covered herein.

### 3. Content

#### 3.1. Design goals and overall constraints and requirements:

- operational environmental temperature range at zero internal power 
  dissipation: -40C to 85C.
  - this is achieved by using components with maximum operational temperature 
    of 85C.

- operational environmental temperature range at maximum performance power 
  dissipation: -40C to 75C:
  - internal power dissipation increases device temperature; for the entire 
    module to achieve 85C maximum temperature, the devices running hot should 
    have higher maximum operational temperature.

- strong preference for devices with full datasheet and free support.

- root size excluding connectors - square, side less than 85mm

- trunk connector supports speeds higher than 5Gbps per differential line.

- total system power max 13.5W.

- all devices with current consumption higher than 0.1mA are turn-off capable.

- optimize EMC for consistent margin against most stringent standards; 
  characterize these margins during system validation.

- capable to operate from Li-Ion 1-cell battery.

- capable to operate from USB-OTG port power supply for development and 
  field servicing purposes.

- support PoE operation as Powered Device and Power Supply Equipment.

- capable to run a main stream Linux distribution.

- include Trusted Platform Module and Crypto-Authentication hardware solutions.

- provide hardware solution for host authentication when operating as device 
  on USB-OTG port.

#### 3.2. Power management

##### 3.2.1. Power design strategy:

- **the Gateway must be seen as a battery operated system with opportunistic 
  access to other power sources.** This perspective helps design a power 
  efficient gateway with extended availability.

- uC and uP must be programmed for lowest system power consumption; such us:
  turn off the Christmas lights - nobody is watching anyway; the battery life
  time is far more precious.
  
- hardware power states: 
  - **power-down**
	- VSB3P3 below minimum level to safely operate the uC; the uC knows the 
	  availability of the system power and always does a controlled
	  power-down of all devices before the voltage supply goes critical.
	
  - **stand-by:**
    - VSB3P3 is at a safe operating level.
    - uC - watching its internal real time clock and trunk interface for a 
	wake-up/interrupt event.
	- V3P3, V1P8 and VCORE regulators are turned off (uP and its external 
	interfaces are powered down).
	- uC monitor VSYS to detect imminent power failure and turn-off all 
	remaining devices before full hardware shut-down occurs.
    - battery reserve is monitored and used for power management.
	
  - **active:**
    - uP is operational and controls its data traffic with branch modules.
	- all unused interfaces are turned off.
	- battery reserve is monitored and used for power management.

***
	
##### 3.2.2. Tree Power Distribution:
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/TreePowerDistribution.PNG)

- The gateway is supplied by the power module which adapts the **DC input** 
  power to feed the **VSYS** rail or to charge a **back-up battery**.

- **VSYS** serves as bulk power distributed to the entire gateway.
  
- **VSYS** can be sourced by an optional *Power over Ethernet* block installed
  on the power module; this PoE block is wired to the Ethernet magnetics on 
  root over two connectors; the magnetics must support PoE+ power levels.

- USB-OTG port in device mode should be able supply a limited current to the 
  power module to feed the VSYS and charge the battery.
  
- **V1P8** and **VCORE** are local distribution rails (root only).

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
  
##### 3.2.3. VCORE and V1P8 Regulator:
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/VCORE_and_V1P8_regulators.PNG)

- **V1P8** rail supplies uP, LPDDR, eMMC, Ethernet Phy

- **VCORE** rail supplies uP (V_CORE and V_MPU connected together)

- **V1P8 and VCORE** regulators feed from VSYS through FB5, C48, C54 filters.

- **VCORE and V1P8** regulators are implemented with FAN5355 buck converter 
  fabricated by Fairchild/On Semi; this device can be controlled over i2c; 
  two different models with distinct i2c addresses are used.

- **V1P8 and VCORE** regulators are synchronized with SYNC3M clock running at 
  nominal 3MHz and phased at 180deg to reduce the switching noise injected
  into the VSYS rail.

- **VCORE** regulator output can be switched between two voltage levels, 
   programmable over i2c, using **VSEL** signal: 
   
   - this feature can be used to change the uP operating power  points (OPP).
	 
   - default start-up output voltage for **VSEL = "low" is 1.05V** and can 
     be used as such for the initial power-up of the uP.
	 
   - default start-up output voltage for **VSEL = "high" is 1.2V and MUST 
     BE ADJUSTED** before use; if there is no need to switch the output
     voltage during normal operation, then the second voltage must be set to same
     value as first (safer operation).	 
	 
- **V1P8 and VCORE** regulators are enabled by the **EN-VCORE and EN-V1P8** 
  signals controlled by the uC.

- **V1P8 and VCORE** regulators offer a light load operation mode configurable 
  over i2c; this mode should be made available by the uC firmware to test its 
  relevance for the system performance.
  
- **V1P8 and VCORE** regulators have the output voltage programmable over i2c
  interface; this feature must be supported by the uC firmware for the system 
  performance tests and to increase power efficiency; 1% change on operating 
  voltage leads to about 2% change in power consumption.
	
- **PWRCLK** signal is forwarded to the trunk interface to synchronize SMPS
  regulators on branches; this feature may help to improve the system noise 
  improvement.
  
- **CORE-MON** is a remote sense signal that provides the VCORE regulator 
  feedback voltage directly from the microprocessor die.
  
- use testpoints t60 and t58 to determine VCORE load current.

- use testpoints t64 and t67 to determine V1P8 load current.

***

#### 3.2.4. V3P3 Regulator:
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/V3P3_regulator.PNG)


- **V3P3** regulator use a buck-boost converter TPS6306 that allows operation from 
  VSYS below 3.3V.
  
- **V3P3 and VUSB** regulators use same IC type TPS6306 and share MSYNC3M 
  signal.
  
- **V3P3 regulator** is turned ON respective to uP power-up/power-down sequence.
  
- drive (uC) **MSYNC3M** HIGH to turn ON the power saving mode; this mode
  needs to be available for system testing to evaluate its contribution to 
  overall performance.
  
- default: drive (uC) **MSYNC3M** with a clock signal of 2.4MHz 
  (2.2MHz to 2.6MHz) as described in 3.2.6. to allow decreasing system noise.
  
- drive (uC) **EN** HIGH to turn ON the regulator.

- **VMON** signal is monitoring the V3P3 voltage - connected to uC A/D 
  converter.  

- use test points t69 and t65 to determine V3P3 load current.

***

#### 3.2.5. VUSB Regulator:
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/VUSB_regulator.PNG)

- **VUSB** regulator use a buck-boost converter TPS6306 that allows to operate
  the USB-OTG port in host mode (requiring 5V).

- **V3P3 and VUSB** regulators use same IC type TPS6306 and share MSYNC3M 
  signal.

- drive (uC) **MSYNC3M** HIGH to turn ON the power saving mode; this mode
  needs to be available for system testing to evaluate its contribution to 
  overall performance.
  
- **default: drive (uC) MSYNC3M with a 2.4MHz clock** as described in 3.2.6. 
  to allow decreasing system noise - see 3.2.6.

- drive (uC) **EN** HIGH to turn ON the regulator.

- **VMON** signal is monitoring the V3P3 voltage - connected to uC A/D converter.  

- use test points t69 and t65 to determine VUSB load current.


#### 3.2.6. Lock the switching frequency for VCORE, V1P8, V3P3 and VUSB regulators:
- VSYS power rail is shared among root and all branches in a gateway; 
  each branch that feeding from this line may introduce noise into it; this 
  noise needs to be limited; the worst offenders are the switching mode power 
  supplies.
  
- a simple method to reduce the ripple noise is to lock the 
  switching frequency of the SMPS regulators feeding from VSYS; this can be 
  done using the uC to generate the switching frequency.
  
- the gateway design allows to control the ripple phase to avoid peak 
  overlapping:
  - each branch using the PWRCLK synchronization signal should add a 47.5ns 
    delay by using an RC circuit followed by a schmitt-trigger gate supplied 
    from VSYS to feed the PWRCLK line of the next branch.

- use the uC to generate MSYNC3M and SYNC3M from same 12MHz internal frequency:
  - divide 12MHz by 5 to generate MSYNC3M at 2.4MHz 50% duty cycle
  - divide 12MHz by 4 to generate SYNC3M at 3MHz 50% duty cycle

#### 3.2.7. Power Module Connector and POE Connectors
+++ insert schematic detail

#### 3.2.8. Power Distribution on Trunk Connector

- VSYS    connected to pins A48-50; filtered with FB4(ferrite bead 220 Ohm @1ooMHz) C53(220R) and C106 (100n)
- VSB3P3  connected to pins A47
- V3P3    connected to pin  A6
- GND     connected to pins A32,A45,B5,B7,B16,B18,B20,B25-26,B38,B41,B44,B47,B50
   - GND is multipurpose:
      - shared as return path for VSYS,VSB3P3,V3P3 and all digital signals 
      - shield/return path for high speed controlled impedance signals (USB)
	  - shield/return path for high speed clocks (SPI and SDIO clocks)

### 3.3. Microprocessor
Use [AM3356BZCZA80](http://www.ti.com/product/AM3356) by TI.
This is an ARM Cortex A-8 32bit RISC processor running at max 600MHz and 
specified over an extended industrial temperatures range of -40C to +105C. 
It includes two Programmable Real-Time Units (PRUs) 32-Bit RISC processors
capable of running at 200 MHz.

#### 3.3.1. Powering, Booting and Clocking

- uP power, boot and reset are controlled by uC.

- uP power configuration:
  - internal RTC block is disabled (use uC for RTC apps).
  - internal RTC LDO regulator is disabled.
  - VDD_CORE and VDD_MPU are supplied from VCORE rail while all other internal 
    blocks are supplied from V1P8 rail.
  - IO buffers voltages are supplied from V3P3 and V1P8 rails as follows:
    - VDDSHV1,3,5 (root circuits)   = 1.8V by V1P8
	- VDDSHV2,4,6 (branch circuits) = 3.3V by V3P3

##### Power-up:

![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/AM335x_powerup.PNG)
- take timing values in the diagram with +/-20% tolerance.
- power-down in reverse order.
- verify 6.1.2. of ![AM335x datasheet](http://www.ti.com/lit/ds/symlink/am3356.pdf) during the root validation

##### Clocking:

- On board crystal connected to XTALIN and XTALOUT pins of uP is 24MHz.

##### Booting

```
====================================================
              Signals translation table
====================================================

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
```
- SYSBOOT signals are latched on rising edge of PWRONRSTn signal:
  1. uC disable all drivers connected to SYSBOOT signals for the period 
    these lines are driven by uC:
     - assert SENn *HIGH* while using ADDR bus to address the branches using such 
       drivers (see Agathis Trunk Standard for more details).

  2. uC drives the appropriate SYSBOOT configuration before asserting PWRONRSTn 
     from *LOW* to *HIGH* and tri-states the SYSBOOT lines after.
    
  3. uC enables the drivers on branches by asserting SENn *LOW* while the 
     respective branches are scanned with the ADDR bus.

**SYSBOOT signals configuration:**

```
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
```
		
***
		
#### 3.3.2. Branch Control Interfaces:
**Control Circuits Diagram from Agathis Trunk Standard:**

![alt text](https://github.com/agathis-project/pinus-rigida/blob/master/control_circuits_diagram.png)

***

**uP Branch Control Diagram**
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/uP_branch_control.PNG)


***

**uC Branch Control Diagram**
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/uC_branch_control.PNG)


!!!! signal name migration: **to follow Agathis Trunk Standard**:
ENn shall change into GEn and TRG shall change into SEn.

- The branch control interface is **controlled by uC in Stand-By state and
  by uP in Active** State.
  
- The branch control interface buffers are supplied from **VSB3P3** rail.
  
- **INTGn, INTSn** are driven by open drain FETs on branches with 10K and 5K 
  pull-up resistance on root for respectively stand-by and active states.
  on root.
  
- **A0-2, GEn and SEn** are driven by uC and uP open drain drivers

- disable uC and uP internal pull-ups.

- limit branch current leakage into control circuits to +/-10uA **!!!!! updated Agathis Trunk Standard for this requirement and verify pull-up resistors for worst leakage logic levels !!!!**

- transistors Q4-7 implement the logic level translation between V3P3 and V1P8 
  domain used by the uP buffers and VSB3P3 domain used by the uC and the 
  branches; at the same time Q4-7 prevent back-feeding the uP buffers when 
  V3P3 or V1P8 supplying the uP are down.
  

```
                                                     uP State   uP State
Control   uC    uC       uP                   uP     before     after
Circuit   Port  Ball     Pin Name             Ball   PWRONRSTn	PWRONRSTn
=========================================================================
INTGn     PF2   C8       RMII1_REFCLK         H18    L          L
INTSn     PF1   D8       XDMA_EVENT_INTR1     D14    Z          L
A0        PK2   B2       MCASP0_ACLKR         B12    L          L
A1        PF4   D9       MCASP0_FSR           C13    L          L
A2        PH6   D6       MCASP0_AXR1          D13    L          L
GEn       PH5   C6       MCASP0_AHCLKX        A14    L          L
SEn       PF0   D7       XDMA_EVENT_INTR0     A15    Z          PD
```

***

#### 3.3.3. Branch Data Interfaces:
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/uP_trunk_data_circuits.PNG)

**all data interface buffers (uP and branches) are supplied from V3P3 rail**



##### 3.3.3.1. SPI.A,B

- SPIA and SPIB on trunk connector are respectively SPI0 and SPI1 of uP AM335x

- SPI on root is always master

- as they propagate up the trunk, the SPIA and SPIB are swapped on every 
  branch that use SPI; this ensures a balanced loading of both channels.

- if a branch use one SPI, it shall connect to SPIA

- if a branch use two SPI, it shall take SPIA and SPIB

- a branch cannot connect more than one load to SPIA respectively SPIB

- SPIx.D0 and SPIx.D1 can be configured by uP as either *miso* or *mosi*; 
  recommended allocation:

```
miso ---> SPI*.D0
mosi ---> SPI*.D1
```

- the root can boot from SPI device on first branch connected to SPIA.

```  
SPI signal allocation table
============================================
           uP            uP          uP
trunk      Pin           Signal      Ball  
circuit    Name          Name        Number  
============================================ 
SPIA.CSn   SPI0_CS0      spi0_cs0    A16  
SPIA.D0    SPI0_D0       spi0_d0     B17  
SPIA.D1    SPI0_D1       spi0_d1     B16  
SPIA.SCLK  SPI0_SCLK     spi0_sclk   A17  
--------------------------------------------
SPIB.CSn   MCASP0_AHCLKR spi0_cs0    C12  
SPIB.D0    MCASP0_FSX    spi0_d0     B13
SPIB.D1    MCASP0_AXR0   spi0_d1     D12  
SPIB.SCLK  MCASP0_ACLKX  spi0_sclk   A13  
```


##### 3.3.3.2. Q.A,B,C,D

- Q.A,B,C,D are four quads - 4 groups of 4 uP signals

- each quad is one allocation unit; unused signals in a quad allocated to a
  a branch cannot be re-allocated to another branch.
 
- the standard quads provide point to point communication: root to one branch

- quad branch routing is controlled by the Agathis Trunk Standard to maximize
  design flexibility, scalability and availabililty.

- intra quad configuration is detailed by the AM335x datasheet, manual and 
  pinmux utility tool.
  
- notable allocations for QA: 
  - uart1 (rxd,txd)
  - uart1 (rxd,txd,ctsn,rtsn)
  - dcan0 (rx,tx) 
  - dcan1 (rx,tx)
  - i2c1  (sda,scl)
  - i2c2  (sda,scl)  
  - pr1_uart0 (rxd,txd)
  - pr1_uart0 (rxd,txd,ctsn,rtsn)
  - gpio

- notable allocations for QB: 
  - uart4 (rxd,txd)
  - uart4 (rxd,txd,ctsn,rtsn)
  - i2c1  (sda,scl)
  - dcan1 (rx,tx)
  - gpio

- notable allocations for QC: 
  - uart5 (rxd,txd)
  - uart5 (rxd,txd,ctsn,rtsn)
  - gpio

- notable allocations for QD: 
  - uart3 (rxd,txd)
  - uart3 (rxd,txd,ctsn,rtsn)
  - gpio
  
- **recommended allocations for QA,B,C,D are respectively uart1,4,5,3**


##### 3.3.3.3. SDIO

- notable allocations for SDIO signals:
  - SDIO  (complies with MMC4.3, SD, SDIO 2.0 Specifications)
  - gpio1 (ARM)
  - pr1_* ((programmable real time unit subsystem))

- **recommended allocation for SDIO is SDIO**


##### 3.3.3.4. GPIO.[0..11]

- this interface supports 12 general purpose IO signals.

- in normal configurations, the GPIOs are allocated to branches on individual 
  basis, in a point to point connection.
  
- unused GPIOs are wired directly, in order, one by one, from down-trunk 
  connector to first GPIOs pins on up-trunk connector.

- uP determines the overal connectivity map from the branch descriptors stored 
  in the branch id eeprom.
  
- notable allocations for GPIO[0..11]
 - pr1_pru1_pru_r30_* (programmable real time unit subsystem)
 - pr1_pru1_pru_r31_* (programmable real time unit subsystem)
 - gpio2_* (ARM)
 - lcd_data[0..7],lcd_hsync, lcd_vsync, lcd_pclk, lcd_ac_bias_en (raster 
   controller for monochrome and color STN displays)


##### 3.3.3.5. I2C-TRUNK
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/uC_i2c.PNG)

- I2C-TRUNK root circuit is compliant with Agathis Trunk Standard. 

- I2C-TRUNK connects uP as master for: 
  - card id eeprom installed on root and branches
  - uC
  - i2c devices installed on branches

- uC and card id eeprom are supplied from VSB3P3 rail which is always ON; 
  this allows uC access to the eeprom when V3P3 is down.

- level translation and power down separation between uC and uP sides of the 
  I2C-TRUNK is implemented with Q2A,B MOSFET:
   - I2C signals on either side is pulled high by their respective pull-ups.
   - I2C signals on either side cannot exceed their respective pull-up as the
     MOSFET is turned off when the channel voltage is VSB3P3 or higher.   
  
- I2C-TRUNK root pull-up is:
  - 6.67K in active mode.
  - 20K in stand-by.

- I2C-TRUNK root maximum parasitic capacitance is 4pF (uP) + 8pF (eeprom) + 
  10pF (uC) +  25pF (PCB) + 5pF (connector) for a total of 52pF; 
  - this parasitic capacitance requires a maximum pull-up resistance of 
    352K/52 = 6.77K; this resistance is build using one 10K resistor on uP side 
	and 20K on uC side.
  - the root parasitic capacitance must be declared in root id eeprom hw 
  descriptors.
  - the PCB parasitic capacitance is a major contributor and needs to be 
  qualified during root hardware validation (measure the PCB parasitic 
  capacitance against GND plane).
  

- maximum gateway total parasitic capacitance cannot exceed 352pF, as this 
  capacitance requires 1K equivalent pull-up which is the lowest an i2c 
  fast-mode capable drive can drive without exceeding the standard minimum 
  logic level.
  
- I2C-TRUNK root leakage current is 18uA (uP) + 2uA (eeprom) + 10uA (uC) for a 
  total of 30uA; this current will cause a maximum 0.2V voltage drop on 6.77K 
  pull-up resistance, which will lead to a VIH = V3P3min - 0.2V = 3V which is 
  higher than VIHmin = 0.7 x V3P3min = 2.24V with a margin of 0.76V.
  - the root max leakage must be declared in root id eeprom hw descriptors.
  
- the number of identical i2c devices in a gateway (I2C-TRUNK) is limited to 
  two to the power of the number of device address pins connected to KNOT.0,1,2; 
  these devices shall be on branches installed in a contiguous block in the 
  gateway:
  - if A0,1,2 address bits are exposed and connected respectivelly to 
    KNOT.0,1,2 then 8 modules with identical i2c devices (one each) can be used;
    this is the particular case of the card id eeprom; other i2c with same access
    to A2,A1,A0 address bits lead to same maximized usability.
  - if only A0,1 address bits are exposed and connected respectivelly to 
    KNOT.0,1 then 4 branches with identical i2c devices (one each) can be used;
    these branches need to be installed in a contiguous block in the gateway.
  - if only A0 address bit is exposed and connected to KNOT.0, then 2 branches
    with identical i2c device (one each) can be used; these branches need to be 
    installed in a contiguous block in the gateway.
  - if no address bit is exposed and connected to KNOT, then only one such
    branch per gateway is allowed.

``` 
I2C-TRUNK signal allocation table

                                           
I2C     uC      uC     uP         uP     
TRUNK   Port    Ball   Pin Name   Ball   
=======================================
SCL     PC1     G6     I2C0_SCL   C16
SDA     PC0     H6     I2C0_SDA   C17
 
```

##### 3.3.3.6. USB.A,B,C,D

- these four USB device ports are USB2.0 and they are controlled by an USB hub 
  [USB2514B](http://www.microchip.com/wwwproducts/en/USB2514B) connected 
  up-stream to USB1 port of uP AM3356.

- the power delivery to the USB devices is controlled through USBdPWR[1..4] 
  signals that drive local USB power switches - intended to reduce power 
  consumption while not in use.

- the devices connected directly to these USB ports must be permanently 
  attached (embedded); if these ports need to exit the gateway to external 
  devices, then use a hub on that branch connected up-stream to USB.A and 
  connect the down-stream ports to the external world.


***

#### 3.3.5. LPDDR Memory:
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/uP_LPDDR.PNG)

- main RAM selection contraints:
  - supported by up AM335x
  - low self-refresh current consumption
  - large enough to run a Linux major distribution

- use MT46H128M16LFDD LPDDR (mDDR) 128M x 16 (256MB) manufactured by Micron; 

- The routing of DDR signals follow the AM335x datasheet recommendations.

***

#### 3.3.6. eMMC and SD-Card Memories:
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/emmc_sdcard.PNG)

- the **eMMC memory** is a 8GB flash MTFC8GACAANA-4M IT by Micron; it 
  integrates a MultiMediaCard and NAND Flash in a 100-Ball package:

  - connected to port mmc1 on AM3356 uP using 1.8V signaling interface.
  
  - use SYSBOOT[4:0] = 11100 to boot from eMMC.
  
  - the device is operated from V3P3 and V1P8; the power supply is turned off 
    by cutting the VSS and VSSQ lines.
	
  - assert EN-eMMC (uC port PJ1, ball# D5) HIGH to turn-on the power.
  
  - the hardware reset is not used - use power-up and software reset instead.
  
  - AM335x microprocessors support MMC4.3; later versions of the MMC standard 
    are backwards compatible; virtually any eMMC memory in 100Ball package 
	should fit the design.
	
  - transfer speed up to 48MByte/s.
  
- the **SD-Card memory** use a hinge micro-SD card socket; any uSD card that 
  meet the speed specified below should fit:
  
  - connected to port mmc0 on AM3356 uP using 3.3V signaling interface.
  
  - the device is operated from V3P3; the power supply is turned off by cutting 
    the VSS line.

  -	assert EN-SDCARD (uC port PE6, ball# E7) HIGH to turn-on the power.
	
  - see "SYSBOOT configuration" chapter for booting options.
  
  - transfer speed in boot mode: 10MHz or up to 5MByte/s 
  
  - transfer speed in normal operation: up to 24MByte/s
  

***

#### 3.3.7. Ethernet:
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/uP_mii.PNG)
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/eth_phy.PNG)

- the Ethernet feature of the root module is implemented with a 3-port Ethernet 
  switch inside AM3356 with one port connected, internally, to the 
  system side and two ports connected to the external LAN connectors through 
  two Phy transceivers KSZ8091MNX by Microchip/Micrel using MII interfaces of 
  AM3356.

- support 10/100Base-T.
  
- the two Ethernet ports support PoE+; the power is separated by the LAN 
  transformers and connected to the power module through connectors J? and J?+1. 

- to implement EEE directive the TXER, TXEN and TXD must be controlled by SW;
  the MAC of AM3356 does not support EEE.


***

#### 3.3.8. uP Clocks, Reset, JTAG and UART0 Signals:
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/uP_rst_jtag_clk_uart0.PNG)


### 3.4. Microcontroller

#### 3.4.1. Power, Reset and Clocking
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/uC_power.PNG)

#### 3.4.2. # Root ID Eeprom
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/uC_card_id_eeprom.PNG)

#### 3.4.3. VBUS Host Voltage Doubler
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/uC_vbus_voltage_doubler.PNG)


#### 3.4.4. I2C buses
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/uC_i2c.PNG)

***

#### 3.4.5. Trusted Platform Module:
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/tpm.PNG)

***

#### 3.4.6. Crypto Authentication:
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/Crypto_auth.PNG)

***



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