---
title: SimpleRISC Processor
date: 2025-04-24
categories:
tags: 
author: Ryan
---

# Introduction
In [ELEC 374 (Digital Systems Engineering)](https://smithengineering.queensu.ca/ece/undergraduate/courses/elec-374), the main course project was to design a full CPU, based on a simplified version of the simpleRISC ISA. This article details my implementation for this project.

## ISA
The ISA has multiple instruction formats which include:
- R-Format for register instructions
- I-Format for instructions with relevant immediate values (Load/Store)
- B-Format for branch instructions
- J-Format for jump, IO, and other special instructions
- M-Format for miscellaneous instructions

## Implementation
I like to design all my Verilog as separate logical modules. I started with designing registers. I chose to go for a register file approach which would prove beneficial later on in the project. 

```Verilog
module regFile (
	input clk, clr, rIn, rOut, BAout, // Control Signals
	input wire [31:0]data_in, // pass data into a register
	input wire [3:0]addr_in, // Select specific register
	output wire [31:0]data_out // put data from a register onto the bus
);

wire [15:0]selected;
wire [15:0]enable;
wire [15:0]sendout;

d4_16 regSelect (
	.code(addr_in),
	.selected(selected)
	);

// initialize r0 separately
wire [31:0] r0_data; // r0 is always zero
wire sendout_r0 = selected[0] & BAout; // Only output control

reg32 ri_0 (
	.clk(clk),
	.clr(clr & selected[0]),
	.en(rIn & selected[0]),
	.d(data_in),
	.q(r0_data)
	
);

wire ba_wire = {32{~BAout}};
tri32 trii_r0 (
    	.in(r0_data & ba_wire),
    	.out(data_out),
    	.ctrl(sendout_r0)
);

genvar i;
generate // Generate GP Registers r0 -- r15
    for (i = 1; i < 16; i = i + 1) begin: gen_loop

	wire [31:0] data_intermediate;
	assign enable[i] = rIn & selected[i];
	assign sendout[i] = (rOut | BAout) & selected[i];

        reg32 ri (
            .clk(clk), 
            .clr(clr & selected[i]), 
            .en(enable[i]), 
            .d(data_in),
            .q(data_intermediate)
        );
	tri32 trii (
  		.in(data_intermediate),
		.out(data_out),
		.ctrl(sendout[i])
	);
    end
endgenerate
endmodule
```

I was very happy with the clean generate statements and this was used as the basis for the datapath along with relevant selection logic that would be driven by the control unit later on.

Following specification, the datapath was designed with a single bus in mind. At each cycle a new stage would be processed and would use the bus to work accordingly. Albeit easy to implement, the performance took a big hit due to each instruction taking a minimum of 5-7 cycles.

The logic was just a 32-5 encoder along with a 32-1 multiplexer where each input was 32 bits wide. This was encoded in Verilog using a case statement.

The ALU was driven as well with custom multiplication logic using Booth's bit-pair recoding and the divider was implemented using the non-restoring algorithm.