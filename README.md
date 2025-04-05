# FPGA-Squadron-mini
    module top (
    output wire led_red,   // Red
    output wire led_blue,  // Blue
    output wire led_green, // Green
    input wire hw_clk,     // Hardware Oscillator, not the internal oscillator
    output wire testwire);
The code explains that the 

Input is hw_clk, Outputs are led_red,led_blue,led_green and testwire

# Internal components

**1. Internal Oscilliator (SB_HFOSC)**
 * generates a stable internal clock signal
 * Uses CLKHF_DIV = "0b10" for clock division
 * CLKHFPU = 1'b1 : Enables power-up
 * CLKHFEN = 1'b1 : Enables oscillator
 * CLKHF : Output connected to internal oscillator signal
**2. Frequency Counter Logic**
   

