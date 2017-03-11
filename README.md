    AP#:           1
    Title:         Root Module HW
    Product:       PCA-1-1
    Repo:          agathis-project
    State:         Verification
    License:       TAPR
    Author:        md-agathisproject

# Root Module HW

## 1. Abstract
The root module is the basic component of the Agathis Gateway which is built
by stacking root and branch modules, sharing a vertical interconnect.
The root has a microprocessor and an integrated 3-port Ethernet Switch
with two LAN ports and one system port; the microprocessor uses a range of low
speed serial interfaces to communicate with a plurality of external world
things over media interfaces located on branches.

## 2. Rationale:

**Current technology:**

- Majority of things use one of the basic physical layer interfaces such as:
  UART, SPI, I2C, SDIO, USB and the ubiquitous GPIO.

- Although today's microprocessors have these interfaces available for embedded
  applications, there is no physical architecture to support a modular scalable
  construction.

- Many things are located in uncontrolled environments; they need
  to deal with the scarcity of power supply, harsh electrical and mechanical
  conditions, wide temperature and humidity ranges and corrosive atmosphere.

- Current solutions to interface with things use single boards computers
  assembled in custom solutions with limited or no scalability or field
  flexibility.

- Practically, there is no open source scalable gateway - until now.

**Agathis Gateway solution:**

- **A stackable field reconfigurable gateway** that can adapt to evolving
  things and scale with their numbers and diversity.

- **A gateway that talks the dumb language of things**, at their physical level,
  like UART, SPI, I2C, GPIO.

- **A gateway that can be installed where the things are** from the sea bed to
  Mars surface, from a floor plant to a deep mining operation.

- **An open source gateway** - that supports competitiveness and fast
  development of new interfaces with the things out there.

- First module of this gateway is the root module, as covered herein.

## 3. Content

### 3.1. Design goals and overall constraints and requirements:

- operational environmental temperature range at zero internal power
  dissipation: -40C to 85C.

- operational environmental temperature range at maximum performance power
  dissipation: -40C to 75C.

- strong preference for devices with full datasheet and free online support.

- root target size excluding connectors: 85 square mm.

- trunk connector using controlled impedance technology to support speeds
  higher than 5Gbps per differential line.

- total system power max 13.5W.

- have a power (load) switch for all devices with current consumption higher
  than 0.1mA.

- optimize EMC for consistent margin against most stringent standards;
  characterize these margins during system validation.

- capable to operate from Li-Ion 1-cell battery.

- capable to operate from USB-OTG port power supply for development and
  field servicing purposes.

- support PoE operation as Powered Device and/or Power Supply Equipment.

- capable to run a mainstream Linux distribution.

- include Trusted Platform Module and Crypto-Authentication hardware solutions.

- provide hardware solution for host authentication when operating as device
  on USB-OTG port.

### 3.2. Power management

#### 3.2.1. Power design strategy:

- **the Gateway must be seen as a battery-operated system with opportunistic
  access to other power sources.** This perspective helps design a power
  efficient gateway with extended availability.

- uC and uP must be programmed for lowest system power consumption; such as:
  turn off the Christmas lights - nobody is watching anyway; the battery life
  time is more precious.

- hardware power states:

  **power-down:**
  - VSB3P3 below minimum level to safely operate the uC; the uC monitors the
    availability of the system power and always does a graceful shut-down
    of all devices before the voltage supply goes critical.

  **stand-by:**
  - VSB3P3 is at a safe operating level.
  - uC monitors its internal real time clock and trunk interface for a
  wake-up/interrupt event.
  - V3P3, V1P8 and VCORE regulators are turned off (uP and its external
  interfaces are powered down).
  - uC monitor VSYS to detect passing the **power-down** threshold and enter
  **power-down** state.
  - battery reserve is monitored and used for power management.

  **active:**
  - uP is operational and controls its data traffic with branch modules.
  - all unused interfaces are turned off.
  - battery reserve is monitored and used for power management.

***

#### 3.2.2. Tree Power Distribution:
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/TreePowerDistribution.PNG)

- The gateway can be powered through:
  1. **DC input** connector on power module.
  2. **LAN Port #1 or #2** configured as PoE Powered Device.
  3. **USB-OTG** connector on root (service only).

- Power module does the primary conversion, storage and distribution.

- **VSYS** serves as bulk power distributed to the entire gateway.

- **V1P8** and **VCORE** are local distribution rails (root only).

- **V3P3** supplies the **data interface circuits** on root and branches.

- **VSB3P3** supplies the **control interface circuits** on root and branches.

- **USB-OTG** port may either:
  - **sink power** when configured as USB Device.
  - **source power** when configured as USB Host.

- The specifications for the trunk power rails are governed by the [AP-7
  Agathis Trunk Standard](https://github.com/agathis-project/pinus-rigida).

***

#### 3.2.3. VCORE and V1P8 Regulator:
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/VCORE_and_V1P8_regulators.PNG)

- **V1P8** rail supplies uP, LPDDR, eMMC, ETH PHY.

- **VCORE** rail supplies uP (V_CORE and V_MPU of AM3356 wired together).

- **V1P8 and VCORE** regulators feed from VSYS through FB5, C48, C54 filters.

- **VCORE and V1P8** regulators are implemented with FAN5355 buck converter
  fabricated by Fairchild/On Semi; this device can be controlled over I2C;
  two different models with distinct I2C addresses are used.

- **V1P8 and VCORE** regulators are synchronized with SYNC3M clock running at
  nominal 3MHz and phased at 180deg to reduce the switching noise injected
  into the VSYS rail.

- **VCORE** regulator output can be switched between two voltage levels,
   programmable over I2C, using **VSEL** signal, which must be asserted *low*
   at power-up:

  - this feature can be used to change the uP operating power.

  - default output voltage for **VSEL = "low" is 1.05V** and can
    be used as such for the initial power-up of the uP.

  - default output voltage for **VSEL = "high" is 1.2V and MUST
    BE ADJUSTED** before use; if there is no need to switch the output
    voltage during normal operation, then the second voltage must be set to
    same value as first (safe operation).

- **V1P8 and VCORE** regulators are enabled by the **EN-VCORE and EN-V1P8**
  signals driven by the uC.

- **V1P8 and VCORE** regulators offer a light load operation mode configurable
  over I2C; this mode should be made available by the uC firmware to test its
  relevance for the system performance.

- **V1P8 and VCORE** regulators have the output voltage programmable over I2C
  interface; this feature must be supported by the uC firmware for the system
  performance tests and to increase power efficiency; 1% change on operating
  voltage leads to about 2% change in power consumption; there are applications
  for which these percentages count.

- **PWRCLK** signal is forwarded to the trunk interface to synchronize SMPS
  regulators on branches; this feature supports system noise reduction.

- **CORE-MON** is a remote sense signal from the uP die to the VCORE regulator.

- use testpoints T60 and T58 to measure VCORE load current.

- use testpoints T64 and T67 to measure V1P8 load current.

***

#### 3.2.4. V3P3 Regulator:
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/V3P3_regulator.PNG)

- **V3P3** regulator use a buck-boost converter TPS6306 that allows operation
  from VSYS below 3.3V.

- **V3P3 and VUSB** regulators use same IC type TPS6306 and share MSYNC3M
  synchronization signal.

- **V3P3 regulator** is turned ON per AM3356 power-up/power-down
  sequence requirements.

- drive (uC) **MSYNC3M** by default with a clock signal of 2.4MHz
  (2.2MHz to 2.6MHz) as described in 3.2.6. to allow decreasing system noise.

- drive (uC) **MSYNC3M** HIGH to turn ON the power saving mode; this mode
  needs to be available for evaluation.

- drive (uC) **EN** HIGH to turn ON the regulator.

- **VMON** signal wired to uC A/D converter monitors the V3P3 voltage.

- use test points T69 and T65 to measure V3P3 load current.

***

#### 3.2.5. VUSB Regulator:
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/VUSB_regulator.PNG)

- **VUSB** regulator use a buck-boost converter TPS6306 that allows to operate
  the USB-OTG port in host mode from VSYS.

- **V3P3 and VUSB** regulators use same IC type TPS6306 and share MSYNC3M
  signal.

- drive (uC) **MSYNC3M** by default with a clock signal of 2.4MHz
  (2.2MHz to 2.6MHz) as described in 3.2.6. to allow decreasing system noise.

- drive (uC) **MSYNC3M** HIGH to turn ON the power saving mode; this mode
  needs to be available for evaluation.

- drive (uC) **EN** HIGH to turn ON the regulator.

- **VMON** signal wired to uC A/D converter monitors the VUSB voltage.

- use test points T69 and T65 to measure VUSB load current.

#### 3.2.6. Switching frequency lock for VCORE, V1P8, V3P3 and VUSB regulators:

- VSYS power rail is shared among root and all branches in a gateway;
  each branch feeding from this line may introduce noise into it; this
  noise needs to be controlled; the worst offenders are the switching mode
  power supplies.

- a simple method to reduce the ripple noise is to lock the
  switching frequency of the SMPS regulators feeding from VSYS; this can be
  done using the uC to generate the switching frequency.

- the gateway design allows to control the phase of the ripple to avoid peak
  overlapping:
  - each branch using the PWRCLK synchronization signal should add a 47.5ns
    delay by using an RC circuit followed by a schmitt-trigger gate supplied
    from VSYS to feed the PWRCLK line for the next branch.

- use the uC to generate MSYNC3M and SYNC3M from same 12MHz internal frequency:
  - divide 12MHz by 5 to generate MSYNC3M at 2.4MHz 50% duty cycle.
  - divide 12MHz by 4 to generate SYNC3M at 3MHz 50% duty cycle.

#### 3.2.7. Power Module Power and PoE Connectors

- connector J9 delivers:
  - VSYS power rail into root module.
  - VSB3P3 standby voltage rail into power module.
  - I2C-MCU bus to control the power module.
  - PINTn interrupt signal from power module.
  - VPIN power from USB-OTG port to feed the gateway in service mode.

- connectors J7 and J8 deliver the power to/from PoE PSE (Power Supply
Equipment) or PD (Powered Device) circuits on power module.

#### 3.2.8. Power Distribution on Trunk Connector

- VSYS    distributed over A48-50; filtered with FB4 (ferrite bead 220R@100MHz)
C53 (10u) and C106 (100n).

- VSB3P3  distributed over A47.

- V3P3    distributed over A6.

- GND     distributed over A32,A45,B5,B7,B16,B18,B20,B25-26,B38,B41,B44,B47,B50.

GND is multipurpose:

- shared as return path for VSYS,VSB3P3,V3P3 and all digital signals.

- shield/return path for high speed controlled impedance signals (USB).

- shield/return path for high speed clocks (SPI and SDIO clocks).

### 3.3. Microprocessor:

Use [AM3356BZCZA80](http://www.ti.com/product/AM3356) by TI.
This is an ARM Cortex A-8 32bit RISC processor running at max 600MHz and
specified over an extended industrial temperature range of -40C to +105C.
It includes two Programmable Real-Time Units (PRUs) 32-Bit RISC processors
capable of running at 200 MHz.

#### 3.3.1. uP Clock, Resets, JTAG and UART0 Signals:
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/uP_rst_jtag_clk_uart0.PNG)

- **XTEST-CPU** harness is connected to the expansion connector.

- **CPU-RST** harness is connected to uC.

- EMU0,1, JTAG and UART0 signals are wired only to the extension connector
  to support uP bring-up and development.

- Crystal Y5 (24MHz) and oscillator support components R101,R108,C116,C137 must
  be optimized through lab experiments.

#### 3.3.2. Power-up sequence and Booting:

- uP power, boot and reset are controlled by uC.

- uP power configuration:
  - internal RTC block is disabled (use uC for RTC apps).
  - internal RTC LDO regulator is disabled.
  - VDD_CORE and VDD_MPU are supplied from VCORE rail while all other internal.
    blocks are supplied from V1P8 rail.
  - IO buffers voltages are supplied from V3P3 or V1P8 rails as following:
    - VDDSHV1,3,5 (root circuits) = 1.8V by V1P8.
    - VDDSHV2,4,6 (branch circuits) = 3.3V by V3P3.

##### 3.3.2.1. Power-up:
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/AM335x_powerup.PNG)

- apply +/-20% tolerance to the timing values in the diagram.

- power-up as depicted and power-down in reverse order.

- verify chapter 6.1.2. of [AM335x datasheet](http://www.ti.com/lit/ds/symlink/am3356.pdf)
  during the root validation.

##### 3.3.2.2. Booting:

```
====================================================
       Booting Signals Translation Table
====================================================

uP Config Pin   uP Pin Name   Trunk Signal   uC Port
=============   ===========   ============   =======
SYSBOOT.15      LCD_DATA15    QC.1           PK7
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
  1. uC disables all drivers connected to SYSBOOT signals for the period
    these lines are driven by uC:
     - assert SENn *high* while using ADDR bus to address the branches using
     such drivers (see Agathis Trunk Standard for more details).
  2. uC drives the appropriate SYSBOOT configuration before asserting PWRONRSTn
     from *low* to *high* and tri-states the SYSBOOT lines after.
  3. uC enables the drivers on branches by asserting SENn *low* while the
     respective branches are scanned with the ADDR bus matching the KNOT bus.

**SYSBOOT signals configuration:**

```
SYSBOOT.15,14       fixed           01 == 24MHz (crystal frequency)
SYSBOOT.13,12       fixed           00 == mandatory by specs
SYSBOOT.11,10,9,8   don't care      XXXX == not controlled by uC
SYSBOOT.7,6         fixed:          00 == select MII for EMAC1
SYSBOOT.5           fixed:          0 == CLK1 OUT disabled
SYSBOOT.4,3,2,1,0   configurable:   00001 == UART0,_,MMC0,SPI0
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

#### 3.3.3. Branch Control Interfaces:
**Agathis Trunk Standard Control Circuits Diagram:**

![alt text](https://github.com/agathis-project/pinus-rigida/blob/master/control_circuits_diagram.png)

***

**uP Branch Control Diagram:**
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/uP_branch_control.PNG)

***

**uC Branch Control Diagram:**
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/uC_branch_control.PNG)

```
!!!! signal name migration: **to follow Agathis Trunk Standard**:
ENn shall change into GEn and TRIG shall change into SEn.
```

- uC is master on branch control interface in **Stand-By** state.

- uP is master on branch control interface in **Active** state.

- the branch control interface buffers are supplied from **VSB3P3**.

- **INTGn, INTSn** signals are driven by open drain FETs on branches with 10K
  pull-up in stand-by state and 5K pull-up in active state; the pull-ups are
  on root.

- **ADDR.0,1,2, GEn and SEn** are driven by uC and uP open drain drivers with
  10K pull-ups on root.

**!! need to rename schematic nets A0,1,2 into ADDR0,1,2 to avoid confusion
  with A0,1,2 I2C addressing bits !!**

- after coming out or reset, the uC and uP shall first:
  - disable any default internal pull-ups on ADDR.0,1,2, GEn and SEn.
  - drive *low* ADDR0,1,2.
  - tristate GEn and SEn drivers to let the pull-up to drive high.

- there are branches that can be up during stand-by state - this should be
  accounted for when designing the transition between active state (uP master)
  and stand-by state (uC master).

- limit branch current leakage into control circuits to +/-10uA.

**!! need to update Agathis Trunk Standard for this requirement and verify
pull-up resistors for worst leakage logic levels!!**

- transistors Q4-7 implement the logic level translation between V3P3 or V1P8
  domain used by the uP buffers and VSB3P3 domain used by the uC and the
  branches; also, Q4-7 prevent back-feeding the uP buffers when
  uP is powered off.

```
Branch Control Signal Assignment Table
=======================================
                                                     uP State   uP State
Control   uC    uC       uP                   uP     before     after
Circuit   Port  Ball     Pin Name             Ball   PWRONRSTn  PWRONRSTn
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

#### 3.3.4. Branch Data Interfaces:
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/uP_trunk_data_circuits.PNG)

**All data interface buffers (located on uP or branches) are supplied from
V3P3 rail**

##### 3.3.4.1. Serial Peripheral Interfaces SPIA and SPIB

- SPIA and SPIB signals on root trunk connector are assigned respectively to
  SPI0 and SPI1 of uP AM335x.

- the root is master on SPIA and SPIB.

- as they propagate up the trunk, the SPIA and SPIB are swapped on every
  branch that use SPI; this wiring balances the total gateway capacitive
  loading and bandwidth.

- if a branch needs one SPI, it shall take the SPIA on down-trunk connector.

- if a branch needs two SPIs, it shall take both SPIA and SPIB.

- the number of SPI devices is limited to two per branch.

- SPIx.D0 and SPIx.D1 can be configured by uP as either *miso* or *mosi*;
  recommended allocation:

```
miso ---> SPI*.D0
mosi ---> SPI*.D1
```

- the root is capable to boot from SPI device on first branch connected to SPIA.

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

##### 3.3.4.2. QA,B,C,D

- QA,B,C,D are four "quads" - groups of four uP signals.

- a quad provides a point to point communication: root to one branch.

- each quad is one allocation unit; unused signals in a quad allocated to a
  a branch cannot be re-allocated to another branch.

- quad branch routing is controlled by the Agathis Trunk Standard to maximize
  gateway flexibility, scalability and availability.

- intra quad configuration is detailed by the AM335x datasheet, manual and
  pinmux utility tool.

- notable allocations for root QA:
  - uart1 (rxd,txd)
  - uart1 (rxd,txd,ctsn,rtsn)
  - dcan0 (rx,tx)
  - dcan1 (rx,tx)
  - i2c1  (sda,scl)
  - i2c2  (sda,scl)
  - pr1_uart0 (rxd,txd)
  - pr1_uart0 (rxd,txd,ctsn,rtsn)
  - gpio

- notable allocations for root QB:
  - uart4 (rxd,txd)
  - uart4 (rxd,txd,ctsn,rtsn)
  - i2c1  (sda,scl)
  - dcan1 (rx,tx)
  - gpio

- notable allocations for root QC:
  - uart5 (rxd,txd)
  - uart5 (rxd,txd,ctsn,rtsn)
  - gpio

- notable allocations for root QD:
  - uart3 (rxd,txd)
  - uart3 (rxd,txd,ctsn,rtsn)
  - gpio

- **THE recommended allocations for QA,B,C,D are respectively uart1,4,5,3**
  - this allocation maximizes flexibility by allowing up to 4 UART branches to
  be stacked together in any order and preserve access to at least one UART.

- uP determines the overall connectivity map from the branch descriptors stored
  in each branch id EEPROM.

##### 3.3.4.3. SDIO

- notable allocations for SDIO signals:
  - SDIO  (complies with MMC4.3, SD, SDIO 2.0 Specifications)
  - gpio1 (ARM)
  - pr1_* (programmable real time unit subsystem)

##### 3.3.4.4. GPIO.[0..11]

- this interface supports 12 general purpose IO signals.

- in normal configurations, the GPIOs are allocated to branches on individual
  basis, in a point to point connection.

- unused GPIOs are wired directly, in order, one by one, from down-trunk
  connector to first GPIOs pins on up-trunk connector.

- notable allocations for GPIO[0..11]:
  - pr1_pru1_pru_r30_* (programmable real time unit subsystem).
  - pr1_pru1_pru_r31_* (programmable real time unit subsystem).
  - gpio2_* (ARM).
  - lcd_data[0..7],lcd_hsync, lcd_vsync, lcd_pclk, lcd_ac_bias_en (raster
    controller for monochrome and color STN displays).

- uP determines the overall connectivity map from the branch descriptors stored
  in each branch id EEPROM.

##### 3.3.4.5. I2C-TRUNK
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/uC_I2C.PNG)

- I2C-TRUNK root circuit is compliant with Agathis Trunk Standard.

- I2C-TRUNK has uP as master for:
  - card id EEPROM installed on root and branches
  - uC
  - I2C devices installed on branches

- uC and card id EEPROM are supplied from VSB3P3 rail which is always ON;
  this allows uC access to the EEPROM in stand-by state.

- level translation and power down separation between VSB3P3 and V3P3 sides of
  the I2C-TRUNK are implemented with Q2A,B n-MOSFET:
   - I2C signals on either side are pulled high by their respective pull-ups.
   - I2C signals voltage on either side cannot exceed their respective pull-up
     voltage as the n-MOSFET is turned off when the channel voltage is VSB3P3
   or higher.

- **I2C-TRUNK root pull-up** is:
  - 6.67K in active mode.
  - 20K in stand-by.

- **I2C-TRUNK root maximum parasitic capacitance is 52pF**; this is made of 4pF
  (uP) + 8pF (EEPROM) + 10pF (uC) +  25pF (PCB) + 5pF (connector);
  - this parasitic capacitance requires a maximum pull-up resistance of
    352K/52 = 6.77K; this resistance is build using one 10K resistor on uP side
  and 20K on uC side.
  - the root parasitic capacitance must be declared in root id EEPROM hw
    descriptors.
  - the PCB parasitic capacitance is a major contributor and needs to be
    qualified during root hardware validation (measure the PCB parasitic
    capacitance against GND plane).

- **I2C-TRUNK gateway maximum parasitic capacitance is 352pF**; this value
  requires 1K equivalent pull-up, which is the lowest an I2C fast-mode capable
  drive can drive without exceeding the standard minimum logic level.

- **I2C-TRUNK root maximum leakage current is 30uA**; this is made of 18uA (uP)
  + 2uA (EEPROM) + 10uA (uC); this current will cause a maximum 0.2V voltage
  drop on 6.77K pull-up resistance, which will lead to a VIH = V3P3min - 0.2V
  = 3V which is higher than VIHmin = 0.7 x V3P3min = 2.24V with a margin of
  0.76V.
  - the root max leakage must be declared in root id EEPROM hw descriptors.

- **I2C-TRUNK gateway maximum number of identical I2C devices** is limited to
  two to the power of the number of device address pins connected to
  KNOT.0,1,2; these devices shall be on branches installed in a contiguous
  block in the gateway:

  - if A0,1,2 address bits are exposed and connected respectively to
    KNOT.0,1,2 then 8 modules with identical I2C devices (one each) can be
    used; this is the particular case of the card id EEPROM; other I2C with
    same access to A2,A1,A0 address bits lead to same maximized usability.

  - if only A0,1 address bits are exposed and connected respectivelly to
    KNOT.0,1 then 4 branches with identical I2C devices (one each) can be used;
    these branches need to be installed in a contiguous block in the gateway.

  - if only A0 address bit is exposed and connected to KNOT.0, then 2 branches
    with identical I2C device (one each) can be used; these branches need to be
    installed in a contiguous block in the gateway.

  - if no address bit is exposed and connected to KNOT, then only one such
    branch per gateway is allowed.

```
I2C-TRUNK Signal Allocation Table
==================================

I2C     uC      uC     uP         uP
TRUNK   Port    Ball   Pin Name   Ball
=======================================
SCL     PC1     G6     I2C0_SCL   C16
SDA     PC0     H6     I2C0_SDA   C17
```

##### 3.3.4.6. USBA,B,C,D

- these four USB device ports are type USB2.0 and they are controlled by an
  USB hub [USB2514B](http://www.microchip.com/wwwproducts/en/USB2514B)
  connected up-stream to port USB1 of uP AM3356.

***

**USB Hub Schematic:**
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/usb_hub.PNG)

```
USB Hub Configuration:
======================
CFG_SEL[0] = 1 (SCL pu)
CFG_SEL[1] = 0 (HS_IND pd)

The hub is configured over I2C as an SMBus slave device:

- Strap options disabled
- All registers configured over SMBus
```

- **HUBCTRL** signal harness is connected to uC
  - **EN** signal control the power switch
  - **RSTn** resets the hub
  - **SUI** indicates the state of the hub

- the power delivery to the USB devices is controlled through USBdPWR[1..4]
  signals that drive local USB power switches - to cut power consumption when
  the port is not in use.

- only permanently attached devices (embedded) can be wired to USBA,B,C,D

- external USB devices shall connect to the down-stream ports of an USB hub
  wired with its up-stream port to one of USBA,B,C,D; this hub shall be
  installed on the branch that connects to the external USB devices.

***

#### 3.3.5. LPDDR Memory:
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/uP_LPDDR.PNG)

- main selection constraints for RAM device:
  - supported by up AM335x
  - lowest self-refresh current consumption
  - large enough to run a Linux major distribution

- use MT46H128M16LFDD LPDDR (mDDR) 128M x 16 (256MB) manufactured by Micron.

- follow the AM335x datasheet recommendations for routing the DDR signals.

***

#### 3.3.6. eMMC and SD-Card Memories:
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/emmc_sdcard.PNG)

- the **eMMC memory** is a 8GB flash MTFC8GACAANA-4M IT by Micron; it
  integrates a MultiMediaCard and NAND Flash in a 100-Ball package:
  - connected to port MMC1 on AM3356 uP using 1.8V signaling interface.
  - the device is operated from V3P3 and V1P8; the power supply is turned off
    by cutting the VSS and VSSQ lines using Q10 dual n-MOSFET.
  - assert EN-eMMC (uC port PJ1, ball# D5) HIGH to turn-on the power.
  - the hardware reset is not used - use power-up and software reset instead.
  - AM335x microprocessors support MMC4.3; later versions of the MMC standard
    are backward compatible; virtually any eMMC memory in 100Ball package
    should fit the design.
  - maximum transfer speed: 48MByte/s.

- the **SD-Card memory** use a hinge micro-SD card socket; any uSD card should
  fit:
  - connected to port mmc0 on AM3356 uP and use 3.3V signaling interface.
  - the device is operated from V3P3; the power supply is turned off by cutting
    the VSS line using Q13 dual MOSFET.
  - assert *high* EN-SDCARD (uC port PE6, ball# E7)  to turn-on the power.
  - see "SYSBOOT configuration" chapter for booting options.
  - maximum transfer speed in boot mode: 5MByte/s
  - maximum transfer speed in normal operation: 24MByte/s

***

#### 3.3.7. Ethernet:
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/uP_mii.PNG)

***

![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/eth_phy.PNG)

***

![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/eth_mag.PNG)

***

- the Ethernet feature of the root module is implemented with the 3-port
  Ethernet switch of AM3356, with one port connected, internally, to the
  system side and two ports connected to the LAN connectors over two PHY
  transceivers [KSZ8091MNX](http://ww1.microchip.com/downloads/en/DeviceDoc/KSZ8091MNX-RNB.pdf)
  using the MII interfaces supporting 10/100Base-T.

- the two Ethernet ports support PoE+; the DC power is separated by the LAN
  transformers and connected to the power module through connectors J7 and J8.

- the PHY transceivers KSZ8091MNX provide several power reduction features:
  - power-saving mode
  - energy-detect power-down mode
  - power-down mode
  - slow-oscillator mode
  - Energy Efficient Ethernet (EEE)
    - to implement EEE directive for the *MAC to PHY* direction, the TXER, TXEN
    and TXD[3:0] must be controlled by the SW; the AM3356's MAC does not
    support EEE.
    - to implement EEE directive for the *PHY to MAC* direction, the RXDV, RXER
    and RXD[3:0] must be monitored by the SW; the AM3356's MAC does not
    support EEE.
  - Wake-On-LAN

- the PHY transceivers can be turned-off completely by uC asserting *low*
  the EN signal.

- the uC monitors the nINT line that can be programmed to transmit a variety of
  PHY events.

```
KSZ8091MNX Straping Options:

PHYADD[2,1] = 00
PHYADD[0]   = 0 for ethernet port 1 (depopulate pu)
PHYADD[0]   = 1 for ethernet port 2
CONFIG[2:0] = 000 (MII)
PME_EN      = 1 (enable PME for Wake-on-LAN
ISO         = 0 (Isolate disable)
DUPLEX      = 1 (Full-duplex)
NWAYEN      = 1 (autoneg enable)
B-CAST-OFF  = 1 (address 0 is set as unique)
NAND_Tree#  = 1 (disable NAND Tree diagnostic)

- ethernet 1 MDIO address = 0
- ethernet 2 MDIO address = 1
- broadcasting MDIO address = disabled
- MII, full duplex, autoneg enable
- disabled ISOLATE function
- disabled PME output for Wake-on-LAN (overridden by SW)
```

- the two PHY transceivers are managed by using a standard MDIO interface.

***

### 3.4. Microcontroller (uC)

#### 3.4.1. Power, Reset and Clocking
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/uC_VSB3P3_reg.PNG)

- the uC is supplied with VSB3P from a low quiescent current(typ 1uA)voltage
  regulator RT9073.

- uC use crystal Y4 to generate 32.768kHz internal clock.

- uC reset is embedded; there are no external features.

#### 3.4.2. Root ID EEPROM
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/uC_card_id_EEPROM.PNG)

- the root id EEPROM is MC24C32, which implements two memories at two distinct
  I2C addresses:
  - 32 Kbyte memory at 0xAE
  - 32  byte id     at 0xBE - lockable, intended to hold a factory programmable,
    never to be changed, serial number.

#### 3.4.3. VBUS Host Voltage Doubler
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/uC_vbus_voltage_doubler.PNG)

- the voltage doubler allows the uP to operate its USB1 port without a 5V
  power supply.

- a clock in 1kHz-100kHz range generated by the uC on **DBLR-CLK** line should
  produce a DC voltage about 5V needed by the sensing circuit in the AM3356;
  the circuit needs to be optimized during validation.

#### 3.4.4. I2C-MCU bus

- this is the I2C bus used by uC to control a number of devices as listed
  below.

- I2C-MCU operates at 400kHz and is I2C Fast Mode compliant.

- this bus is connected as well to the power module where other devices may be
  connected; power module design is responsible to avoid any I2C addressing
  issues and maintain compliance with I2C Fast Mode.

```
I2C-MCU addresses map (8 bit with LSB as the w/r bit)
=====================================================

USB2514B            : 0x58
ATEC508A            : 0xC0
AT97SC3205T         : 0x28
FAN5355UC03X (V1P8) : 0x90
FAN5355UC03X (VCORE): 0x94
```

***

#### 3.4.5. Trusted Platform Module:
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/tpm.PNG)

***

- the TPM solution can be implemented with one of the following devices:
  - [AT97SC3205T](http://www.microchip.com/wwwproducts/en/AT97sc3205t) (by Microchip/Atmel) - installed by default.
  - [ST19NP18](http://www.st.com/content/ccc/resource/technical/document/data_brief/7e/15/02/2a/e9/bd/4b/eb/DM00039181.pdf/files/DM00039181.pdf/jcr:content/translations/en.DM00039181.pdf) (by STMicro)
  - [SLB9645](http://www.infineon.com/dgdl/Infineon-TPM+SLB+9645-DS-v02_14-EN.pdf?fileId=5546d4625185e0e201518b83d0c63d7c) (by Infineon)

The schematic details the resistor stuffing for the above three options.

- the TPM device is connected to uC over I2C-MCU and a number of up to 5
  configurable IOs.

#### 3.4.6. Crypto Authentication:
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/Crypto_auth.PNG)

- a crypto authentication solution is implemented using one of the following
  [options](http://ww1.microchip.com/downloads/en/DeviceDoc/Atmel-8756-ATSHA204A-ATAES132A-ATECC108A-ATECC508A-Flyer-E.pdf):
  - ATSHA204A
  - ATAES132A
  - ATECC508A (default installation)

#### 3.4.7. TRIGIO

- this is a trigger line that can be used by the gateway for real time
applications where a synchronization is desired among some or all of the
modules.

- TRIGIO on root is wired on latest release to uC only; there is a plan to wire
it as well to uP in next hw release.

***

### 3.5. USB Switch
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/usb_switch.PNG)

- when the root needs to be the device for an USB host connected to USB-OTG
  port, a hw switch(U22) connects the host to the uC USB port to run the
  authentication; after passing the test, the host is switched to
  the uP USB (USB0 of AM3356).

- harness **USB0** is wired to uP.

- harness **USB-MCU** is wired to uC.

- harness **USB-SW-CTRL** is wired to uC.

- harness **USB-CONN** is wired to USB-OTG connector.

***

### 3.6. Trunk Connector:
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/Trunk_connector.PNG)

***

### 3.7. Test Extension Connector:
![alt text](https://github.com/agathis-project/salix-arctica/blob/master/AP-1/Test_extension_card_connector.PNG)

### 3.8. Schematic

[The schematic](https://github.com/agathis-project/salix-arctica/blob/master/v1/SCH-1-1-1.pdf)
for this design was captured in Altium and is available as pdf and project
file package in v1 folder of this repo.

### 3.9. Layout

The layout for this design was done in Altium.
See [v1](https://github.com/agathis-project/salix-arctica/blob/master/v1) folder.

### 3.10. Mechanicals

- tbd

### 3.11. Prototype

- tbd

### 3.12. Validation

- tbd

### 3.13. Integration

#### 3.13.1. EMC Compliance Test Plan

#### 3.13.2. Safety Test Plan

#### 3.13.3. Hazardous Materials Control Plan

## 4. References

- tbd

## 5. License

This design is licensed under the terms of the TAPR.
The terms of the license are available in the LICENSE.TXT file included in the
repository.

## 6. Attachments

See [AP-1](https://github.com/agathis-project/salix-arctica/blob/master/AP-1) folder
