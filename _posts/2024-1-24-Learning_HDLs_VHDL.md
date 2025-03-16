---
title: Learning HDL/HVLs | VHDL
date: 2024-1-24
categories:
  - HDL
  - Programming
tags:
  - HDL
  - VHDL
  - Programming
  - WIP
author: Ryan
---
# Introduction
After taking ELEC271 - Digital Systems in the F23 Semester of my undergraduate. I remembered how much I loved digital systems. To preface, in high school I had an engineering class that went over the basics of logic gates, truth tables, and binary/octal/hexadecimal conversion. I found it to be the most fun unit we would have that year. Now that this it is the W24 semester, I've decided to teach myself Verilog/SystemVerilog, along with reinforcing my knowledge of VHDL from ELEC271 to ideally work for a ASIC/FPGA this summer (2024). This is part 1 of the blog post series with this post containing VHDL.

## Resources
I will link resources that I am using below for those who also want to learn:
- Verilog and the ASIC Design Flow, Udemy course linked [here](https://www.udemy.com/course/verilog-hdl-vlsi-hardware-design-comprehensive-masterclass/).
- For practice of Verilog, I used HDLBits linked [here](https://hdlbits.01xz.net/wiki/Main_Page).
- For project ideas and more Verilog/VHDL, fpga4fun site linked [here](https://www.fpga4fun.com/SiteInformation.html).
- For learning VHDL I used some videos from VHDLwhiz linked [here](https://VHDLwhiz.com);
- My GitHub repository Erebus linked [here](https://github.com/Supercobra27/Erebus);

## Before we begin
It is important to understand that all code written with HDLs is run simultaneously. This is why it is called a hardware description language as it runs in RTL or in Register Transfer Logic. There are statements in each language to combat this to allow code to be run sequentially. An example of sequential code is an if statement written in C.

# VHDL
VHDL is a programming language I was introduced to in ELEC271 and from my friend last year when he was doing practice labs for some reason. This was my initial spark to hardware and ASIC design and VHDL is a less employed, but still powerful language for communicating with and controlling FPGAs. Every VHDL file usually starts with an `entity` declaration.

```vhdl
entity example is
port(

);
end example;

```

Inside of the port brackets one will usually put the inputs and outputs from outside sources into an FPGA, whether that be a button, a switch, a sensor or anything else. This is also where you describe the outputs. I'll fill in the `entity` declaration with some example ports.

```vhdl
entity example is
port(
  button : in std_logic;
  output : out std_logic
);
end entity;

```

Above I have added ports for a simple button (1 bit) input and a 1 bit output. `std_logic` comes from the standard library provided by IEEE. This library is much more powerful than the basic `BIT` type provided by VHDL and can be declared using this syntax at the top of every VHDL file.

```vhdl
library ieee;
use ieee.std_logic_1164.all;
```

Now that we have designed the inputs from outside sources, it is time to design the behavior that we want implemented on the FPGA. This goes inside of the `architecture` block. Here is an example of one below:

```vhdl
architecture behavior of example is
	begin
		output <= button and '1';
end architecture;
```

In this case, we have an asynchronous (not synced with clock) occurrence where whenever the button input is triggered, its input passes through an `AND` gate along with a constant signal of `1` which is then evaluated to be put at the output. In real applications, usually circuits are synced up to a clock signal to allow for predictable changes in the the timing analysis of circuits. In order for this to happen, we have to make operations synchronous where they are synced to a clock edge. This an be done using a `process` statement along with an if statement inside which contains the condition of `(clk = '1' and clk'EVENT)`. This only evaluates to true on a positive clock edge. Our changed synchronous code would look like this:

```vhdl
library ieee;
use ieee.std_logic_1164.all;

entity example is
port(
	button : in std_logic;
	output : out std_logic;
	clk : in std_logic -- our new clock signal to allow synchronous code
);
end entity;

architecture behavior of example is
	begin
	process(clk)
		begin
		if(clk'EVENT and clk = '1') then
		output <= button and '1';
		end if;
	end process;
end architecture;
```

Now the code will only execute when the button is pressed on a positive clock edge. **need to talk about sensitivity list**

# VHDL Project
Now that we have a basic understanding of VHDL, let's do an example project where we implement a simple 4 bit shift register which includes asynchronous clear and synchronous loading capabilities.

```vhdl
library ieee;
use ieee.std_logic_1164.all;

entity 4b_register is
port(
	clear, load, clock : in std_logic;
	output : out std_logic;
	data : in std_logic_vector(3 downto 0)
);
end entity;

architecture behavior of 4b_register is
	begin
	process(load, clear)
		begin
			if(clear = '1') then --async clear
				data <= "0000";
			end if;
			
			if (clock'EVENT and clock = '1') then --sync load
			data(0) <= data(1);
			data(1) <= data(2);
			data(2) <= data(3);
			data(3) <= load;
			end if;
	end process;
end architecture;
```

# Summary
I know that I can go way more in-depth in this post, as it will continue to be updated as my VHDL knowledge gets stronger, or I find something cool that I didn't know that I could do in it. This was a good practice of explaining VHDL to help reinforce my own concepts.
