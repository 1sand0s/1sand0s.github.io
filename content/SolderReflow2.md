Title: Solder Reflow Oven 2.0
Date: 2021-11-24 19:11
Modified: 2021-11-24 19:11
Category: Solder Reflow, Control System
Tags: PID Control System
Author: AudiTT
Summary: This post details how to build a Solder Reflow Oven controller based on PID

Found some time to work on hobby projects for thanksgiving break and seeing that I 
had to limit myself to using IC packages with extended leads due to lack of a way to 
soldering them, I decided on building a Solder Reflow Oven 2.0 !

The first version was built around an AT90USB646, it worked fine but it used a separate
12V dc adapter to power the board. This time around I wanted to power the board and the
reflow oven from the mains and also add some TFT LCD tech to make it more rad!! This version uses
an STM32L053 mcu based on ARM Cortex M0+. Well, thats all the motivation/excuse I needed to build Solder Reflow Oven 2.0
  

<img src="/images/SolderReflow_3D.png" width="800" height="600" />

<h3> Power Supply </h3>

I did not want to deal with multiple wires coming in and out of the oven so I decided to power the board from the mains. This meant incorporating
an AC to DC (3.3V) stage. The stage consists of a rectifier implemented using a diode-bridge, before rectifying I step down the voltage from 115V to
24V so that I can use components (diodes, capacitors) rated at lower voltages which makes the entire build cheaper.  

<img src="/images/powerStage.png" width="800" height="600" />

On applying the stepped down AC to the diode bridge, we obtain a pulsating DC at the output. To smooth this pulsating DC we can use a large capacitor. Once
the capacitor fully charges to peak value of the pulsating DC, it can practically supply charge with minimal drop in its voltage till it charges again. This helps remove
ripple and produce a constant DC voltage. The output of the ripple smoothing capacitor is then input to a 10V regulator to obtain a 10V stable output.

<img src="/images/R2R_3_3V.png" width="800" height="600" />

I then use a R2R divider to obtain 3.3V from the 10V rectifier output. Although this is not the most efficient way to do it, this isn't much of a problem at low 
voltages or power. We would have needed a buck converter if the power involved was higher to improve efficiency of the conversion from 10V to 3.3V.

<h3> Crystal Layout </h3>

The crystal oscilator unit perhaps contains the highest frequency signals (24MHz) in our board. For this reason we must take care in routing and placing it in such 
a way that it doesn't couple onto nearby traces nor it get affected due to noise in nearby traces. Capacitive and inductive coupling are inevitable when traces are 
routed in close proximity to each other, however, we can decrease the extent of such coupling by following certain guidelines during route and placing. Capacitive 
coupling primarily depends on :

1. Overlap area - long sections of parallel traces increases capacitive coupling (more noise)
2. Distance to ground plane - large distance between signal and ground plane increases capacitive coupling (more noise)

<img src="/images/crystalLayout.png" width="800" height="600" />

In the above layout, the highlighted components and traces correspond to the crystal unit. By restricting other signal traces and copper pour around my crystal unit 
I can reduce the overlap area (see above points) and thereby reduce capacitive coupling. Distance to ground plane is reduced by opting for a 4-layer board but I 
think this layout should be sufficient for the frequencies I am working at and should work without any issues with a 2-layer board.

<img src="/images/crystalLayoutB.png" width="800" height="600" />
  
I also use a local ground plane for the crystal unit and tie it to the global ground at a single point. This prevents any other high frequency return currents from
flowing under the crystal unit (this is due to increase in inductance due to larger loop area that these return currents would have to take if they flowed under 
the crystal unit's local ground). This further improves the integrity of all signals involved in our crystal unit. 


<h3> Controlling the Relay </h3>

The relay I used requires about 15mA of current to turn on, the STM32L053 can deliver it but while operating at near its max rating which decreases its lifetime. 
Instead, I use an NPN bjt to drive the relay while controlling it with the mcu. 

<img src="/images/relayControl.png" width="800" height="600" />

I bias the NPN such that it operates in either cut-off or saturation. Cut-off is easy, we just make Vbe 0. To drive it into saturation we need to ensure that Vce
is driven into saturation, from the datasheet for 2SD1805 we see that the typical value for Vce,sat is 0.22V. Therefore, we need to choose Rc such that Vce is Vce,sat when Ic is 15mA. Hence, Rc = (10-Vce,sat)/Ic = (10-0.22)/0.015 = 652 Ohms. I use 653 Ohms for Rc. Since Ic is independent of beta in saturation, we can choose
 Ib independent of Ic as long as its sufficient to make Vbe > 0.7V (turn on base-emitter junction). I chose Ib to be 10mA from which I calculate Rb = (3.3V - Vbe)/Ib = (3.3 - 0.7)/0.01 = 260 Ohms. I therefore, use 260 Ohms for Rb.



<h3> Bill of Materials </h3>
| Component   | Quantity | Total Cost | Description |
|:------------|:--------:|:----------:|:------------|
| <a href="https://www.mouser.com//ProductDetail/511-STM32L053R6T6">511-STM32L053R6T6</a> | 1 | $5.32 | ARM Microcontrollers - MCU ARM Microcontrollers - MCU 16/32-BITS MICROS |
| <a href="https://www.mouser.com//ProductDetail/947-RCLAMP0502A.TCT">947-RCLAMP0502A.TCT</a> | 2 | $1.82 | ESD Suppressors / TVS Diodes ESD Suppressors / TVS Diodes Low Capacitance TVS Array, 5V |
| <a href="https://www.mouser.com//ProductDetail/863-2SD1805F-TL-E">863-2SD1805F-TL-E</a> | 3 | $2.91 | Bipolar Transistors - BJT Bipolar Transistors - BJT BIP NPN 5A 20V |
| <a href="https://www.mouser.com//ProductDetail/700-MAX31855KASA%2bT">700-MAX31855KASA+T</a> | 1 | $5.38 | Sensor Interface Sensor Interface Cold-Junction Compensated Thermocouple-to-Digital Converter |
| <a href="https://www.mouser.com//ProductDetail/750-CDBHD140L-G">750-CDBHD140L-G</a> | 2 | $3.30 | Bridge Rectifiers Bridge Rectifiers LOW VF 1A 40V Schottky Bridge |
| <a href="https://www.mouser.com//ProductDetail/621-AZ1117R-3.3TRE1">621-AZ1117R-3.3TRE1</a> | 2 | $1.10 | LDO Voltage Regulators LDO Voltage Regulators LDO BJT HiCurr |
| <a href="https://www.mouser.com//ProductDetail/649-67996-400HLF">649-67996-400HLF</a> | 50 | $1.70 | Headers & Wire Housings Headers & Wire Housings .100CC STR HEADER |
| <a href="https://www.mouser.com//ProductDetail/612-LL3301NF065QG">612-LL3301NF065QG</a> | 2 | $1.20 | Pushbutton Switches Pushbutton Switches 50mA 12VDC F065 4.3mm Gull Wing |
| <a href="https://www.mouser.com//ProductDetail/640-FFC2B28-24-G">640-FFC2B28-24-G</a> | 3 | $1.98 | FFC & FPC Connectors FFC & FPC Connectors 24W,0.5mm FFC Con,R/A,Dual Cont,B/Flp,H1.0mm,Gld,SMT,T&R |
| <a href="https://www.mouser.com//ProductDetail/710-150060VS75000">710-150060VS75000</a> | 20 | $2.80 | Standard LEDs - SMD Standard LEDs - SMD WL-SMCW SMDMono TpVw Waterclr 0603 BrtGrn |
| <a href="https://www.mouser.com//ProductDetail/81-GRM0335C1H180FA1D">81-GRM0335C1H180FA1D</a> | 5 | $0.50 | Multilayer Ceramic Capacitors MLCC - SMD/SMT Multilayer Ceramic Capacitors MLCC - SMD/SMT |
| <a href="https://www.mouser.com//ProductDetail/603-CC0402KPX58BB104">603-CC0402KPX58BB104</a> | 10 | $0.43 | Multilayer Ceramic Capacitors MLCC - SMD/SMT Multilayer Ceramic Capacitors MLCC - SMD/SMT 25V 0.1uF X5R 0402 10% |
| <a href="https://www.mouser.com//ProductDetail/963-EMK107BJ105KAHT">963-EMK107BJ105KAHT</a> | 10 | $0.37 | Multilayer Ceramic Capacitors MLCC - SMD/SMT Multilayer Ceramic Capacitors MLCC - SMD/SMT 0603 16VDC 1uF 10% X5R AEC-Q200 |
| <a href="https://www.mouser.com//ProductDetail/963-UMK325ABJ106MMHT">963-UMK325ABJ106MMHT</a> | 10 | $5.09 | Multilayer Ceramic Capacitors MLCC - SMD/SMT Multilayer Ceramic Capacitors MLCC - SMD/SMT 1210 50VDC 10uF 20% X5R AEC-Q200 |
| <a href="https://www.mouser.com//ProductDetail/660-RN731JTTD6570D25">660-RN731JTTD6570D25</a> | 2 | $0.62 | Thin Film Resistors - SMD Thin Film Resistors - SMD 0603 657 Ohms 0.5% 25PPM |
| <a href="https://www.mouser.com//ProductDetail/279-CPF-A-0603B1K0E">279-CPF-A-0603B1K0E</a> | 4 | $2.04 | Thin Film Resistors - SMD Thin Film Resistors - SMD CPF A 0603 1K0 0.1% 25PPM 5K RL |
| <a href="https://www.mouser.com//ProductDetail/815-ABM824MHZD1GT">815-ABM824MHZD1GT</a> | 2 | $1.24 | Crystals Crystals CRYSTAL 24.0000MHZ 18PF SMD |
| <a href="https://www.mouser.com//ProductDetail/810-MMZ1005S601HTD25">810-MMZ1005S601HTD25</a> | 10 | $0.83 | Ferrite Beads Ferrite Beads 600ohm 25% 440mA |
| <a href="https://www.mouser.com//ProductDetail/474-SEN-13715">474-SEN-13715</a> | 1 | $5.00 | SparkFun Accessories SparkFun Accessories Thermocouple Type-K - Stainless Steel |
| <a href="https://www.mouser.com//ProductDetail/553-F24-100-C2">553-F24-100-C2</a> | 1 | $5.16 | Power Transformers Power Transformers POWER XFMR 24 Vct @ 100 mA UL/cUL - Class 2/3, 115V SPLIT PACK PCB MOUNT / F24-100-C2 |
| <a href="https://www.mouser.com//ProductDetail/661-EMHS500GRA471MKG">661-EMHS500GRA471MKG</a> | 2 | $3.30 | Aluminum Electrolytic Capacitors - SMD Aluminum Electrolytic Capacitors - SMD 470uF 20% 50V AEC-Q200 |
| <a href="https://www.mouser.com//ProductDetail/926-LM2940IMPX10NOPB">926-LM2940IMPX10NOPB</a> | 4 | $7.96 | LDO Voltage Regulators LDO Voltage Regulators 1-A, 26-V, high-PSRR, low-dropout voltage regulator 4-SOT-223 -40 to 85 |
| <a href="https://www.mouser.com//ProductDetail/649-1012938090202ALF">649-1012938090202ALF</a> | 50 | $1.90 | Headers & Wire Housings Headers & Wire Housings ECONOSTIK HEADER SR VT SMT 1X2 |
| <a href="https://www.mouser.com//ProductDetail/651-1935161">651-1935161</a> | 2 | $0.98 | Fixed Terminal Blocks Fixed Terminal Blocks PT 1.5/2-5.0-H |
| <a href="https://www.mouser.com//ProductDetail/485-4520">485-4520</a> | 1 | $12.50 | TFT Displays & Accessories TFT Displays & Accessories Adafruit 1.3 240x240 Wide Angle IPS TFT Display |
| <a href="https://www.mouser.com//ProductDetail/992-MINI-SPDT-SW">992-MINI-SPDT-SW</a> | 2 | $3.96 | Slide Switches Slide Switches MINI SPDT SWITCH |
| <a href="https://www.mouser.com//ProductDetail/791-0201N8R0C500CT">791-0201N8R0C500CT</a> | 5 | $0.50 | Multilayer Ceramic Capacitors MLCC - SMD/SMT Multilayer Ceramic Capacitors MLCC - SMD/SMT 8.0pF +-0.25pF 50V |
| <a href="https://www.mouser.com//ProductDetail/187-CL21B104KBCNFNC">187-CL21B104KBCNFNC</a> | 10 | $0.29 | Multilayer Ceramic Capacitors MLCC - SMD/SMT Multilayer Ceramic Capacitors MLCC - SMD/SMT 100nF+/-10% 50V X7R 2012 |
| <a href="https://www.mouser.com//ProductDetail/963-TMK325B7226MM-PR">963-TMK325B7226MM-PR</a> | 3 | $2.25 | Multilayer Ceramic Capacitors MLCC - SMD/SMT Multilayer Ceramic Capacitors MLCC - SMD/SMT 1210 25VDC 22uF 20% X7R |
| <a href="https://www.mouser.com//ProductDetail/652-CRT0603BY2610ELF">652-CRT0603BY2610ELF</a> | 2 | $1.00 | Thin Film Resistors - SMD Thin Film Resistors - SMD CHIP RESISTOR Precision |
| <a href="https://www.mouser.com//ProductDetail/279-CPF0603D10KE">279-CPF0603D10KE</a> | 4 | $1.08 | Thin Film Resistors - SMD Thin Film Resistors - SMD CPF0603 10K 0.5% 25PPM |
| <a href="https://www.mouser.com//ProductDetail/667-ERA-3VEB2001V">667-ERA-3VEB2001V</a> | 3 | $1.74 | Thin Film Resistors - SMD Thin Film Resistors - SMD 0603 0.1% 2Kohm 25ppm SMD |

