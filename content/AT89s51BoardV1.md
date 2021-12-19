Title: Make your own 8051(AT89S51) Board
Date: 2019-12-4 19:11
Modified: 2019-12-4 19:11
Category: Custom MCU Board
Tags: AT89S51, 8051 Board
Author: AudiTT
Summary: Short version of my first blog post

I had a week off for thanksgiving and thought it was worth the time to start designing some basic general puprose MCU boards
to build upon for more complex projects later on.

The AT89S51 based off of Intel's famous MCS-51 instruction set is an 8-bit CISC microcontroller. Its fairly simple and so I
though of beginning my blog by detailing how to design a general purpose board centred on arguably its most popular variant 
AT89S51 (there are other variants supplied by Zilog, Philips etc).

Its important that if you choose to use an 8051 from Atmel (Microchip) in your design that you go for the **AT89S51** rather
than **AT89C51** since the **S** variant offers in-system programming (ISP) and hence allowing SPI and other protocols to be used 
to program the MCU without needing a dedicated programmer. 

The Kicad schematic for the board is explained below

![Photo]({attach}PowerSupply.PNG)

The power supply for the MCU is made up of a voltage regulator **MC78L05** which is capable of outputting 5V DC from an input
of 5V to 15V DC. I have added an additional 330uF bulk decoupling capacitor in case the MCU draws too much current during which
the low frequency ripple in the power supply line will be reduced by the bulk decoupling capacitor. Presently however, I haven't
found the need yet to include it but just in case.



![Photo]({attach}Crystal.PNG)

The oscilltor is the usual **Pierce-type oscillator** employing a standard 24Mhz crystal from Abracon (through hole). The Abracon crystal
oscillator has a load capacitance of 18pF and hence the caps($C_p$) employed to match this impedance is given by the formula
$C_p = 2 * C_L - 2 * C_{stray}$

I have ignored $C_{stray}$ the stray capacitance since I have placed the crystal very close to the XTAL pins of the MCU.


![Photo]({attach}TestLED.PNG)

As is customary, the Hello World equivalent of a hardware project is the LED blink test. D1 is the Power status LED whereas D2 is the test LED
and is connected to the fifth pin of Port 1. Before programming, the RST should be pulled high for 2 machine cycles to reset the MCU. To perform
this reset quickly, I have added a pushbutton SPST between RST and VCC.


![Photo]({attach}MCU.PNG)

The only interesting thing mentioning in MCU schematic is that the bypass capacitor 0.1uF has to be placed as close as possible to the VCC supply
pin of the MCU to ensure that the shortest path is available for the high frequency currents(ripples) to flow back to the source.
Essentially it decreases the parasitic inductance by reducing the loop area.


The Unsoldered and the Soldered boards : Kudos to OSHPark 


<img src="/images/UnSoldered.jpg" width="700" height="700" />    <img src="/images/Soldered.jpg" width="700" height="700" />


Lets take a quick look at the blink test before heading onto the programming intracacies.

<iframe src="http://www.youtube.com/embed/fNMvQsAE2kg"
   width="600" height="400" frameborder="0" allowfullscreen>
</iframe>


The usage of AT89S51 has emphasized before makes programming the board extremely simple. We are going to use SPI (Serial Peripheral Interface)
protocol. I had an Arduino Mega lying around and decided to use that as the ISP (In-system Programmer).

These are the steps to be followed to program the 8051 board.

1. Upload the ArduinoISP code to the arduino you intend to use as the ISP

2. Connect MISO of Arduino to MISO of the 8051 board

3. Connect MOSI of Arduino to MOSI of the 8051 board

4. Connect SCK of Arduino to SCK of the 8051 board

5. The maximum baud rate allowed is equal to $\frac{8051 Crystal Frequency}{16}$. I usually use 19200

6. To simplify the avrdude upload process I have written a powershell script(runs only on Windows but can easily be ported).

7. Download the powershell script from my github repository and add the folder called Bootloader to your environment path variable

8. Edit the bootloader.ps1 file present inside the Bootloader folder such that $AVRDUDE_HOME points to the avr_dude.exe and $CONF_FILE points to 
   the config file for 8051 (F40R96CIUSLFZFP.conf). The config file is also present in the Bootloader directory.

9. Proceed to the directory where your HEX file is present and upload it using powershell by executing the command 

                                              bootloader.ps1 COM3 19200 your_hex_file.hex

10. Detailed procedure can be found by executing the command

                                              bootloader.ps1 -h

11. Once the code has been successfully uploaded to the 8051, short the EA/VPP pin and the VCC pin to ensure that the code memory is read from internal
    ROM.



<a href="https://github.com/1sand0s/AT89S51-8051-General-Purpose-Board">Github Link to the Powershell Scripts and Kicad Project</a>



