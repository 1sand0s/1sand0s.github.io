Title: Building an S-Band Radar 
Date: 2021-11-6 19:11
Modified: 2021-11-6 19:11
Category: Radars
Tags: FMCW
Author: AudiTT
Summary: This post details how to build an  S-Band Ground Penetrating Radar

This post details my second and perhaps a more in-depth attempt at building an S-Band Ground Penetrating Radar. So lets jump right in ! 

<h3> What is the S-Band and why choose it ? </h3>

Electromagnetic waves are capable of existing with various frequencies and this entire range of frequency is called a spectrum. EM waves have been harnesed to serve
several different purposes, from Communication to Remote Sensing and down to heating our foods ! We certainly cannot have someone microwaving their broccoli jeopardising
the radio communication with a space shuttle for instance. Hence, to maximize the potential of EM wave utility, we need to 

1. Allocate certaing range of frequencies called bands for specific operations
2. Share the band by following the max transmit power statutes  

The establishment of such laws and regulations gives the governing body a legal basis to prosecute any malfeasance while simultaneously preventing interference amongst users.
The laws and regulations concerning the allocation and usage of the EM spectrum in the United States is set by the FCC. The S-Band is one such range of frequencies covering
2-4GHz. Operations such as Satellite Communciations, WiFi and microwave ovens etc utilize this band. In this project, we restrict ourselves to the ISM band 2.4 - 2.4835GHz
portion of the S-Band.

In summary:

1. We operate in the ISM band of the S-Band 2.4-2.4835GHz
2. Mass production of commercial devices for Wifi applications utilizing this band helps reduce the cost of our build
3. Does not require licensing from FCC as long as we abide by the rules in &sect; 15.249 of Title 47 of Code of Federal Regulations    

<h3> Functional Diagram </h3>

<img alt="thumbnail" height="450px" src="/blockDiagram.png">

The functional diagram of our build is presented above. All images used are representational and attributions to the respective authors can be found at the bottom of this page.
At the highest level of abstraction, our GPR is made up of the PC, GPR controller board and the transmit and receive antennas. 

Functions of the PC

1. Upload firmware to the GPR controller MCU
2. Obtain data from GPR controller MCU for analysis
3. Supply power to the GPR controller

Functions of the GPR Controller Board

1. Digitize the range information and transfer it to the PC
2. Generate an FMCW sweep/chirp of desired frequency range and resolution

Functions of the Antenna

1. Transmit the FMCW signal from the GPR controller board
2. Receive the reflected signal and transfer it to the GPR controller board

<h3> GPR Controller Board </h3>


The major constraint in choosing the parts for the board is packaging of the IC. Since I don't have a solder reflow oven, I cannot solder packages without leads.

<h4> USB to UART/SPI </h4>
MCP2210
<h4> DSP Microcontroller </h4>
<h4> Fractional N PLL </h4>
<h4> VCO </h4>
<h4> Loop Filter </h4>
<h4> Reference Oscillator </h4>
<h4> Power Divider </h4>
<h4> LNA </h4>
<h4> Mixer </h4>



<a href="https://github.com/1sand0s/Lc3B-Assembler">Github Link to the LC3b Assembler</a>









