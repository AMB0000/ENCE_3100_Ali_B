# Lab 2

## Introduction to Lab

This lab investigates the design of combinational logic on the DE10-Lite FPGA using Verilog.
 We create circuits that do the following:

 Converting binary to decimal
 Operations in Binary-Coded Decimal (BCD)
 Ripple-carry adders and full adders
 Binary to BCD conversion and multi-digit BCD addition
 With inputs from slide switches (SW), all results are shown on the DE10-Lite board's HEX7-segment displays.

## Used hardware in lab

DE10-Lite FPGA Development Board
Intel MAX 10 FPGA
10 slide switches (SW9–SW0)
6 seven-segment HEX displays (HEX5–HEX0)
10 LEDs (LEDR9–0)
Quartus Prime Lite Edition



## Part 1

Objective: Display SW[15–0] values on HEX3–HEX0 (modified for DE10-Lite).
One HEX display is driven by each 4-bit group.
Digits 0–9 are displayed accurately, whereas 10–15 are not important.

Code used : 

// Example 7-segment decoder
module seg7_dec(
    input i3, i2, i1, i0,
    output [7:0] o_seg
);
    assign o_seg[0] = ~((~i3 & ~i2 & ~i1 & i0) |
                        (~i3 & i2 & ~i1 & ~i0));
    // … (remaining segments logic)
endmodule

## Part 2 

We need to design a circuit that converts a 4-bit binary number V = v3v2v1v0 into its two-digit decimal 
equivalent D = d1d0. For example, binary 1010 (decimal 10) should be displayed as "10" on the seven
segment displays. 
The circuit includes: 
A comparator that checks if V > 9, Z=1 if its is greater than 9 
Z=1 decides d1 ,if z=1 we have to display d1 as 1. 


// Example binary-to-decimal (4-bit)
bin_to_dec b2d (
    .i_v3(SW[3]), .i_v2(SW[2]), .i_v1(SW[1]), .i_v0(SW[0]),
    .o_seg0(HEX0), .o_seg1(HEX1)
);

## Part 3

We need to design a 4-bit ripple-carry adder circuit that adds two 4-bit numbers. The circuit consists of 
four full adder modules connected in a chain, with the carry output of each adder connected to the 
carry input of the next adder 
The full adder has: 
Inputs: a, b, and carry-in (ci) 
Outputs: sum (s) and carry-out (co) 
Verilog Code: 
// Full adder module 
module full_adder( 
input a, b, ci, 
output s, co 
); 
// Sum output is XOR of all three inputs 
assign s = a ^ b ^ ci; 
// Carry output is generated when at least two inputs are 1 
assign co = (a & b) | (a & ci) | (b & ci); 
endmodule 
// Top-level ripple carry adder 
module part3( 
input [8:0] SW,  // SW[7:4] = A, SW[3:0] = B, SW[8] = cin 
output [8:0] LEDR,  // Connect to SW 
output [4:0] LEDG   // LEDG[3:0] = Sum, LEDG[4] = Cout 
); 
// Connect switches to red LEDs 
assign LEDR = SW; 
wire [3:0] A, B; 
wire cin; 
wire [3:0] S; 
wire c1, c2, c3, cout; 
// Assign inputs 
assign A = SW[7:4]; 
assign B = SW[3:0]; 
assign cin = SW[8]; 
// Instantiate four full adders in a chain (ripple-carry) 
full_adder FA0(A[0], B[0], cin, S[0], c1); 
    full_adder FA1(A[1], B[1], c1, S[1], c2); 
    full_adder FA2(A[2], B[2], c2, S[2], c3); 
    full_adder FA3(A[3], B[3], c3, S[3], cout); 
     
    // Connect outputs to green LEDs 
    assign LEDG[3:0] = S; 
    assign LEDG[4] = cout; 
endmodule

## Part 4 

The objective is to add two BCD digits (A and B).
 SW[3:0] and SW[7:4] are input.
 Carry-in: SW[8].
 Output: HEX1–HEX0 shows the BCD sum S1S0.
 LEDR[9] indicates an error if input is greater than 9.


module bcd_adder(
    input  [3:0] A, B,
    input        Cin,
    output [3:0] Sum,
    output       Cout
);

    wire [4:0] raw_sum;   // 5-bit to hold normal binary addition
    reg  [4:0] adj_sum;   // corrected sum

    assign raw_sum = A + B + Cin;

    always @(*) begin
        if (raw_sum > 9)
            adj_sum = raw_sum + 6;   // BCD correction
        else
            adj_sum = raw_sum;
    end

    assign Sum  = adj_sum[3:0];
    assign Cout = adj_sum[4];

endmodule


## Part5

Objective: Add A1A0 + B1B0, two 2-digit BCD digits.
 constructed with two instances of the BCD adder from Part IV.
 Three-digit BCD sum S2S1S0 on HEX2–HEX0 is the output.

module bcd_2digit_adder (
    input  [3:0] A0, A1,   // lower and upper digits of operand A
    input  [3:0] B0, B1,   // lower and upper digits of operand B
    output [3:0] S0, S1, S2
);

    wire [3:0] sum0, sum1;
    wire c1, c2;

    // First digit addition (units place)
    bcd_adder add0 (
        .A(A0),
        .B(B0),
        .Cin(1'b0),
        .Sum(sum0),
        .Cout(c1)
    );

    // Second digit addition (tens place)
    bcd_adder add1 (
        .A(A1),
        .B(B1),
        .Cin(c1),
        .Sum(sum1),
        .Cout(c2)
    );

    // Assign results
    assign S0 = sum0;          // units digit
    assign S1 = sum1;          // tens digit
    assign S2 = {3'b000, c2};  // hundreds digit (only 0 or 1)

endmodule

## Part6

The objective is to convert a 6-bit binary number (0–63) into two-digit BCD.
 SW[5:0] is the input.
 HEX1 = tens, HEX0 = ones, is the output.

module bin2bcd_display (
    input  [5:0] SW,       // 6-bit binary input
    output [6:0] HEX0,     // ones digit display
    output [6:0] HEX1      // tens digit display
);

    reg [3:0] tens, ones;

    always @(*) begin
        tens = SW / 10;
        ones = SW % 10;
    end

    hex7seg u0 (
        .in(ones),
        .out(HEX0)
    );

    hex7seg u1 (
        .in(tens),
        .out(HEX1)
    );

endmodule

## What was learned 

The distinction between BCD and binary encoding.
Use of switches → HEX displays in a practical FPGA.
Verilog styles: structural versus behavioral.
conversion from binary to decimal on constrained hardware.








