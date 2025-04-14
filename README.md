# FPGA-Squadron-mini-TASK 1
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
    make clear
    make build 
    sudo make flash

# OUTPUT

https://github.com/user-attachments/assets/916b014a-0b85-4969-92d7-a43dfc03b5dd


## Task 2 
# UART Loopback 

It is designed for an FPGA (Field Programmable Gate Array) and implements a UART Loopback along with RGB LED control using an internal oscillator and counter.
It module enables serial communication via UART and controls RGB LEDs based on the signals.

led_red -	Output(1-bit)	Controls Red LED (Active High).
led_blue-	Output(1-bit)	Controls Blue LED (Active High).
led_green-	Output(1-bit)	Controls Green LED (Active High).
uarttx-	Output(1-bit)	UART TX Line (Transmits serial data).
uartrx-	Input(1-bit)	UART RX Line (Receives serial data).
hw_clk-	Input(1-bit)	FPGA system clock input.


# UART TX (Transmitter)
    module uart_tx_8n1 (
    input wire clk,        // System clock
    input wire [7:0] txbyte, // 8-bit data input
    input wire senddata,   // Transmission trigger
    output reg txdone,     // Transmission completion flag
    output wire tx         // UART TX output
    );

txbit-Stores the current bit being transmitted.
buf_tx-Shift register that holds UART data.
bits_sent-Tracks how many bits have been transmitted.

# FSM 
    parameter STATE_IDLE    = 8'd0;  // Wait for `senddata`
    parameter STATE_STARTTX = 8'd1;  // Send Start Bit (0)
    parameter STATE_TXING   = 8'd2;  // Send 8-bit Data
    parameter STATE_TXDONE  = 8'd3;  // Send Stop Bit (1)
# Internal Registers and wires
    reg[7:0] state = 8'b0; // Holds the current state of the FSM

    reg[7:0] buf_tx = 8'b0;   // Buffer to store the byte being transmitted

    reg[7:0] bits_sent = 8'b0; // keeps track for the number of bits transmitted (8 bits)

    reg txbit = 1'b1;         // Holds the TX output signal

    reg txdone = 1'b0;        // Flag to indicate transmission is complete

    assign tx = txbit; // Assign txbit register to the output tx pin

# TX logic 
    always @(posedge clk) begin
    if (senddata == 1 && state == STATE_IDLE) begin
        state <= STATE_STARTTX;
        buf_tx <= txbyte;
        txdone <= 1'b0;
    end else if (state == STATE_IDLE) begin
        txbit <= 1'b1;
        txdone <= 1'b0;
    end

    if (state == STATE_STARTTX) begin
        txbit <= 1'b0; // Start bit
        state <= STATE_TXING;
    end

    if (state == STATE_TXING && bits_sent < 8'd8) begin
        txbit <= buf_tx[0]; // Send LSB first
        buf_tx <= buf_tx >> 1;
        bits_sent <= bits_sent + 1;
    end else if (state == STATE_TXING) begin
        txbit <= 1'b1; // Stop bit
        bits_sent <= 8'b0;
        state <= STATE_TXDONE;
    end

    if (state == STATE_TXDONE) begin
        txdone <= 1'b1;
        state <= STATE_IDLE;
    end
    end
* When the senddata is High and the FSM state is in IDLE state, the state is changed to STARTTX to start the transmission
* The 8 bit input is stored in a buffer register buf_tx.
* Txdone is cleared to indicate transmission has started.
* The first bit 0 is sent as the starting bit to initiate the transmission.
* The conditional block checks whether the state is TXING and bits sent are less than 8 bits.
* If 8 bits are already sent reset the bit counter and set the txbit flag high changing the state to TXDONE
* txdone flag is set to high indicating the transmission is complete and
* state is set to STATE_IDLE


    assign uarttx = uartrx;
Any data sent on uarttx is instantly received on uartrx.
This eliminates the need for external connections during testing.

# Block diagram 
![image](https://github.com/user-attachments/assets/ce9e3369-8649-40f4-9190-2fd7df71e238)



# Testing and Output
    git clone https://github.com/Skandakm29/VsdSquadron_mini_fpga_uart_loopback.git
    cd "VsdSquadron_mini_fpga_uart_loopback"
    make build
    sudo make flash
    sudo picocom -b 9600 /dev/ttyUSB0 --echo
if any error try using 


    sudo make flash
    sudo apt install picocom
    sudo picocom -b 9600 /dev/ttyUSB0 --echo

# OUTPUT

https://github.com/user-attachments/assets/f9d19156-f0ae-422e-94af-69727f810e89

## Task 3
# UART Transmit module 
This module implements an 8N1 UART Transmitter, enabling serial data transmission using an 8-bit data frame, no parity bit, and 1 stop
bit. It generates a 9600 baud clock from a 12 MHz oscillator and provides a simple state-machine-based transmission mechanism.

* Internal Oscillator - Generates the main clock signal (clk_out).
* Frequency Counter - Uses clk_out to count the frequency and displays it,helps verify your oscillator output or system clock stability.
* UART Transmission: The UART transmitter (uart_tx_8n1) continuously sends the character 'D' using an 8N1 format (8 data bits, No parity, 1 stop bit).
* Baud Rate Generator - Converts the main clock into a baud_clk (e.g., 9600 Hz for UART communication).
* UART Receiver (RX) - Captures serial data from PC
* UART Loopback - Sends back the received data directly to UART TX (echo test).
* RGB LED Controller - Takes parallel data from UART RX.

# State Machine States 
# IDLE STATE (STATE_IDLE)

If senddata = 1 and the state is STATE_IDLE, it:

Moves to the STATE_STARTTX state.

Loads txbyte (8-bit data to transmit) into buf_tx.

Clears txdone (indicates transmission is ongoing).

Otherwise, if still in STATE_IDLE, it:

Keeps txbit high (1) because UART idles at high.

Ensures txdone remains low (0).

* Start Bit Transmission (STATE_STARTTX)
  Once in STATE_STARTTX, it:
  Sets txbit low (0) (start bit in UART communication).
  Moves to STATE_TXING to transmit data bits.
  
* Sending Data Bits (STATE_TXING)

If state == STATE_TXING and bits_sent < 8, it:
Sends the Least Significant Bit (LSB) of buf_tx.
Shifts buf_tx right (>> 1).
Increments bits_sent.

* Stop Bit Transmission (STATE_TXDONE)

After 8 data bits are transmitted, it:
Sends the stop bit (1).
Resets bits_sent to 0.
Moves to STATE_TXDONE.

* Transmission Complete (STATE_TXDONE → STATE_IDLE)

In STATE_TXDONE, it:
Sets txdone = 1 (indicates transmission complete).
Returns to STATE_IDLE.

# Block diagram 



# OUTPUT

https://github.com/user-attachments/assets/ccb3a8d5-523c-4f3c-a356-e8fa778bce03




