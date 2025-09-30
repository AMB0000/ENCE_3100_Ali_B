# Lab 3: Latches, Flip-Flops and Registers



Student : Ali Behbehani
Instructor: Goncalo Perirera martins


# Introduction

In this lab, 
our goal is to explore how storage elements can be implemented in an FPGA. 
Although Altera FPGAs provide built-in flip-flops that are commonly used 
to store data, we will first demonstrate how storage elements can be created
without relying on these dedicated flip-flops.


## Objective

Before using the FPGA’s internal flip-flops (which we will cover in Part IV),
we will show how to construct basic memory behavior by designing circuits that
retain information through logic and feedback paths.
This step will highlight how storage can be achieved at a structural level and
deepen our understanding of how sequential behavior is realized inside programmable logic devices.




# Part 1

In this lab, we will demonstrate how to generate storage elements in an FPGA without the need for its specialized Flip-Flops.
A gated RS latch circuit is the one shown below. Although the latch can be properly implemented in a single 4-input LUT, 
the implementation prevents the observation of its internal signals, R_g and S_g, as they are not supplied as LUT outputs.
A compiler directive must be included in the code in order to maintain these internal signals in the implemented circuit.
To tell the Quartus compiler to use distinct logic elements for each of the signals R_g, S_g, Qa, and Qb, the directive /* synthesis maintain */ 
should be included in the code. Compiling the code produces the circuit with four 4-LUTS depicted.



I began by coming up with a new idea for my DE-series Board.  The Verilog file for the RS latch i made myself.

![My Screenshot](SC1.png)


Here is my code,

module part1 (Clk, R, S, Q);
    input Clk, R, S;
    output Q;

    wire R_g, S_g, Qa, Qb /*synthesis keep*/;

    and (R_g, R, Clk);
    and (S_g, S, Clk);
    nor (Qa, R_g, Qb);
    nor (Qb, S_g, Qa);

    assign Q = Qa;
endmodule





# Part 2 


The circuit provided in this lab section is a gated D latch circuit.
I began by coming up with a fresh idea. Then, in order to make sure
that the distinct logic elements used to implement signals R, S_g, R_g, Qa, and Qb were used, 
I created a Verilog file code using the /* synthesis maintain */ command.
I then verified my circuit using RTL Viewer after compiling my code.


![My Screenshot](SC2.png) 





My code, 

module part2 (Clk, D, Q);
    input Clk, D;
    output Q;

    wire R_g, S_g, Qa, Qb /*synthesis keep*/;

    nand (R_g, R, Clk);
    nand (S_g, D, Clk);
    nand (Qa, R_g, Qb);
    nand (Qb, S_g, Qa);

    assign R = ~D;
    assign Q = Qa;
endmodule


The gated D latch was then implemented on my DE-series board using a different project that I had made.
The top-level module that housed the proper input and output ports (the pins) made up the projected. 
In the top-level module, I initialize my latch.  I controlled the latch's D input using switch SW0. 
I then input the Clk using SW1.  Lastly, I linked LEDR0 to the Q output.
I assembled the projection and downloaded it to my board after adding the necessary pin assignments. 
I then toggled the D and Clk switches and looked at the Q output to make sure my circuit was working.



![My Screenshot](SC3.png)


# Part 3



# Part 4 






