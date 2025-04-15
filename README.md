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
![image](https://github.com/user-attachments/assets/a406b90e-ee7e-4684-8a95-1e74b9ca078e)



# OUTPUT

https://github.com/user-attachments/assets/ccb3a8d5-523c-4f3c-a356-e8fa778bce03

## Task 4
# Architecture Summary 
 The core components of this architecture are:

Sensor Data Processing

Baud Clock Creation

UART Data Transfer Mechanism

State Machine for Transmission Control
# Workflow
 Sensor readings are captured at specific time intervals. The data_valid signal flags when new data is ready to be sent. A 32-bit buffer temporarily stores sensor values before transmission.A baud rate generator ensures a consistent 9600 baud frequency. A counter mechanism regulates accurate timing for each bit.

 * START BIT: Transmission begins with a low (0) start bit.
 * DATA BITS: Sends 8-bit segments of the 32-bit sensor data sequentially.
 * STOP BIT: Ends with a high (1) stop bit to complete the data frame.
tx_done indicates the end of a transmission cycle.

ready ensures the module can handle back-to-back sensor data without loss.

tx_out: Carries the UART-encoded serial data to external devices.

# Blockdiagram 
![image](https://github.com/user-attachments/assets/61f39bd9-84fd-4651-afd9-397b5799c704)





# Output 

https://github.com/user-attachments/assets/44201ac3-9aca-43dd-91ca-6701c201627f

## PROJECT 
# Real time data acquistion and Tx system
This document details the complete architecture of a real-time system for acquiring sensor data and transmitting it, implemented on an FPGA. The system focuses on measuring distance using an HC-SR04 ultrasonic sensor. The FPGA processes the sensor signals and subsequently communicates the calculated distance to a computer via a UART interface. This report encompasses system specifications, circuit diagrams, commented Verilog code for each module, testing methodologies, and a video demonstration of the system in operation.


The described system comprises several interconnected functional blocks:

* A 12 MHz internal oscillator integrated within the FPGA serves as the primary clock signal for the entire system.
* A dedicated timing unit is responsible for generating periodic trigger signals for the distance measurement, with selectable intervals such as 50 ms or 250 ms.
* An ultrasonic sensor module, implemented in Verilog (hc_sr04.v), produces a 10 µs trigger pulse and determines the distance based on the time it takes for the echo to return.
* A UART transmitter module (uart_tx_8n1.v) manages the serial communication, transmitting the computed distance in ASCII character format at a data rate of 9600 bits per second.
* Optional RGB LEDs can provide a visual indication of the system's operational state. The top-level Verilog design integrates all the individual modules. It stores the obtained distance value, transforms it into its ASCII representation, and facilitates its transmission to a personal computer through a USB-to-Serial converter.

# Block diagram 


# Circuit diagram


# Uart Tx 
    // 8N1 UART Module, transmit only

    module uart_tx_8n1 (
    clk,        // input clock
    txbyte,     // outgoing byte
    senddata,   // trigger tx
    txdone,     // outgoing byte sent
    tx,         // tx wire
    );

    /* Inputs */
    input clk;
    input[7:0] txbyte;
    input senddata;

    /* Outputs */
    output txdone;
    output tx;

    /* Parameters */
    parameter STATE_IDLE=8'd0;
    parameter STATE_STARTTX=8'd1;
    parameter STATE_TXING=8'd2;
    parameter STATE_TXDONE=8'd3;

    /* State variables */
    reg[7:0] state=8'b0;
    reg[7:0] buf_tx=8'b0;
    reg[7:0] bits_sent=8'b0;
    reg txbit=1'b1;
    reg txdone=1'b0;

    /* Wiring */
    assign tx=txbit;

    /* always */
    always @ (posedge clk) begin
    // start sending?
    if (senddata == 1 && state == STATE_IDLE) begin
        state <= STATE_STARTTX;
        buf_tx <= txbyte;
        txdone <= 1'b0;
    end else if (state == STATE_IDLE) begin
        // idle at high
        txbit <= 1'b1;
        txdone <= 1'b0;
    end

    // send start bit (low)
    if (state == STATE_STARTTX) begin
        txbit <= 1'b0;
        state <= STATE_TXING;
    end
    // clock data out
    if (state == STATE_TXING && bits_sent < 8'd8) begin
        txbit <= buf_tx[0];
        buf_tx <= buf_tx>>1;
        bits_sent = bits_sent + 1;
    end else if (state == STATE_TXING) begin
        // send stop bit (high)
        txbit <= 1'b1;
        bits_sent <= 8'b0;
        state <= STATE_TXDONE;
    end

    // tx done
    if (state == STATE_TXDONE) begin
        txdone <= 1'b1;
        state <= STATE_IDLE;
    end

    end

    endmodule

# Ultra sonic Sensor integration 
    module hc_sr04 #( parameter ten_us = 10'd120  // ~120 cycles for ~10µs at 12MHz)
    (
    input             clk,         // ~12 MHz clock
    input             measure,     // start a measurement when in IDLE
    output reg [1:0]  state,       // optional debug: current state
    output            ready,       // high in IDLE (between measurements)
    input             echo,        // ECHO pin from HC-SR04
    output            trig,        // TRIG pin to HC-SR04
    output reg [23:0] distanceRAW, // raw cycle count while echo=1
    output reg [15:0] distance_cm  // computed distance in cm
    );

  
    // State definitions
 
    localparam IDLE      = 2'b00,
             TRIGGER   = 2'b01,
             WAIT      = 2'b11,
             COUNTECHO = 2'b10;

    // 'ready' is high in IDLE
    assign ready = (state == IDLE);

    // 10-bit counter for ~10µs TRIGGER
    reg [9:0] counter;
    wire trigcountDONE = (counter == ten_us);

    // Initialize registers (for simulation & synthesis without reset)
    initial begin
    state       = IDLE;
    distanceRAW = 24'd0;
    distance_cm = 16'd0;
    counter     = 10'd0;
    end

    // 1) State Machine
  
    always @(posedge clk) begin
    case (state)
      IDLE: begin
        // Wait for measure pulse
        if (measure && ready)
          state <= TRIGGER;
      end

    TRIGGER: begin
    // ~10µs pulse, then WAIT
    if (trigcountDONE)
      state <= WAIT;
    end

    WAIT: begin
    // Wait for echo rising edge
    if (echo)
      state <= COUNTECHO;
    end

    COUNTECHO: begin
    // Once echo goes low => measurement done
    if (!echo)
      state <= IDLE;
    end
 
    default: state <= IDLE;
    endcase
    end

  
     // 2) TRIG output is high in TRIGGER
  
     assign trig = (state == TRIGGER);

  
    // 3) Generate ~10µs trigger pulse
 
    always @(posedge clk) begin
    if (state == IDLE) begin
      counter <= 10'd0;
    end
    else if (state == TRIGGER) begin
      counter <= counter + 1'b1;
    end 
    
    end

 
    // 4) distanceRAW increments while ECHO=1

    always @(posedge clk) begin
    if (state == WAIT) begin
      // Reset before new measurement
      distanceRAW <= 24'd0;
    end
    else if (state == COUNTECHO) begin
      // Add 1 each clock cycle while echo=1
      distanceRAW <= distanceRAW + 1'b1;
    end
    end

 
    // 5) Convert distanceRAW to centimeters
 
    // distance_cm = (distanceRAW * 34300) / (2 * 12000000)
    always @(posedge clk) begin
    distance_cm <= (distanceRAW * 34300) / (2 * 12000000);
    end

    endmodule

    //===================================================================
    // 2) Refresher for ~50ms or ~250ms pulses
    //===================================================================
    module refresher250ms(
    input  clk,  // 12MHz
    input  en,
    output measure
    );
    // For ~50ms at 12MHz: 12,000,000 * 0.05 = 600,000
    // For ~250ms at 12MHz: 12,000,000 * 0.25 = 3,000,000
    reg [18:0] counter;

    // measure = 1 if counter == 1 => single‐cycle pulse
    assign measure = (counter == 22'd1);

    initial begin
    counter = 22'd0;
    end

    always @(posedge clk) begin
    if (~en || (counter == 22'd600000))  
    // change to 3_000_000 if you want 250ms
    counter <= 22'd0;
    else
    counter <= counter + 1;
    end
    endmodule

# Testbench code
    `include "uart_trx.v"
    `include "ultra_sonic_sensor.v"


    module top (
      // outputs
      output wire led_red,    // Red
      output wire led_blue,   // Blue
      output wire led_green,  // Green
      output wire uarttx,     // UART Transmission pin
      input  wire uartrx,     // UART Reception pin
      input  wire hw_clk,
      input  wire echo,       // External echo signal from sensor
      output wire trig        // Trigger output for sensor
    );

  
    wire int_osc;
    SB_HFOSC #(.CLKHF_DIV("0b10")) u_SB_HFOSC (
    .CLKHFPU(1'b1),
    .CLKHFEN(1'b1),
    .CLKHF(int_osc)
    );

  
    reg  clk_9600 = 0;
    reg  [31:0] cntr_9600 = 32'b0;
    parameter period_9600 = 625; // half‐period for 12 MHz -> 9600 baud

    always @(posedge int_osc) begin
    cntr_9600 <= cntr_9600 + 1'b1;
    if (cntr_9600 == period_9600) begin
      clk_9600  <= ~clk_9600;
      cntr_9600 <= 32'b0;
    end
    end

  
    SB_RGBA_DRV #(
    .RGB0_CURRENT("0b000001"),
    .RGB1_CURRENT("0b000001"),
    .RGB2_CURRENT("0b000001")
    ) RGB_DRIVER (
    .RGBLEDEN(1'b1),
    .RGB0PWM(uartrx),
    .RGB1PWM(uartrx),
    .RGB2PWM(uartrx),
    .CURREN(1'b1),
    .RGB0(led_green),
    .RGB1(led_blue),
    .RGB2(led_red)
    );

 
    wire [23:0] distanceRAW;       
    wire [15:0] distance_cm;       
    wire        sensor_ready;
    wire        measure;

    hc_sr04 u_sensor (
    .clk        (int_osc),
    .trig       (trig),
    .echo       (echo),
    .ready      (sensor_ready),
    .distanceRAW(distanceRAW),
    .distance_cm(distance_cm),  // must exist in your sensor module
    .measure    (measure)
    );

  
    refresher250ms trigger_timer (
    .clk (int_osc),
    .en  (1'b1),  // always enabled
    .measure (measure)
    );

 
    reg [3:0] state;
    localparam IDLE    = 4'd0,
             DIGIT_4 = 4'd1,
             DIGIT_3 = 4'd2,
             DIGIT_2 = 4'd3,
             DIGIT_1 = 4'd4,
             DIGIT_0 = 4'd5,
             SEND_CR = 4'd6,
             SEND_LF = 4'd7,
             DONE    = 4'd8;

    reg [31:0] distance_reg; // latch distance_cm for division
    reg [7:0]  tx_data;
    reg        send_data;

 
    always @(posedge clk_9600) begin
    // By default, don't load a new character
    send_data <= 1'b0;

    case (state)
 
    IDLE: begin
    if (sensor_ready) begin
      distance_reg <= distance_cm; // store the 16-bit measurement
      state <= DIGIT_4;           // go print all digits
    end
    end

  
    DIGIT_4: begin
    tx_data  <= ((distance_reg / 10000) % 10) + 8'h30;
    send_data <= 1'b1;
    state    <= DIGIT_3;
    end
    DIGIT_3: begin
    tx_data  <= ((distance_reg / 1000) % 10) + 8'h30;
    send_data <= 1'b1;
    state    <= DIGIT_2;
    end
    DIGIT_2: begin
    tx_data  <= ((distance_reg / 100) % 10) + 8'h30;
    send_data <= 1'b1;
    state    <= DIGIT_1;
    end
    DIGIT_1: begin
    tx_data  <= ((distance_reg / 10) % 10) + 8'h30;
    send_data <= 1'b1;
    state    <= DIGIT_0;
    end
    DIGIT_0: begin
    tx_data  <= (distance_reg % 10) + 8'h30;
    send_data <= 1'b1;
    state    <= SEND_CR;
    end

  
    SEND_CR: begin
    tx_data   <= 8'h0D; // '\r'
    send_data <= 1'b1;
    state     <= SEND_LF;
    end
    SEND_LF: begin
    tx_data   <= 8'h0A; // '\n'
    send_data <= 1'b1;
    state     <= DONE;
    end

  
    DONE: begin
    state <= IDLE;
    end

    default: state <= IDLE;
    endcase
    end

  
    uart_tx_8n1 sensor_uart (
    .clk      (clk_9600),
    .txbyte   (tx_data),
    .senddata (send_data),
    .tx       (uarttx)
    );

    endmodule

# OUTPUT

https://github.com/user-attachments/assets/38786079-7456-45ee-bc37-dc5c43562988





