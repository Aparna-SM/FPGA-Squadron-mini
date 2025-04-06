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
 * 28-bit register (frequency_counter_i)
 *  Increments on every positive edge
 *  Bit 5 is routed to testwire for monitoring
 *  Provides a way to verify oscillator operation and timing

**3. RGB LED**
 * RGBLEDEN = 1'b1 : Enables LED
 * RGB0PWM = 1'b0 : Red LED minimum brightness
 * RGB1PWM = 1'b0 : Green LED minimum brightness
 * RGB2PWM = 1'b1 : Blue LED maximum brightness
 * All LEDs are set to minimum current
 * RGB0 - led_red
 * RGB1 - led_green
 * RGB2 - led_blue

# PIN ASSIGNMENTS
 * led_red - LED red output - PIN.no 39
 * led_green - LED green output - PIN.no 40
 * led_blue - LED blue output - PIN.no 41
 * hw_clk - External hardware clock - PIN.no 20
 * testwire - For debugging - PIN.no 17

# Build and Flash the code

