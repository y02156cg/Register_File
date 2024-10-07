# Project Checkpoint 3
- Name: Chen Gong
- NetID: cg387

# Overview
Aim of this project is to construct a register file that is able to write data to registers and read data from registers. This project contains three ports in total: 1 write port, 2 read ports. In this register file, there are 32 32-bits registers. 

# Design Description
In total, there are three ports in this register file design, which are one write port that can write data to registers, and two read ports to read data from the registers. The reading and writing operations are all controlled by the signals ctrl_writeReg, ctrl_readRegA, and ctrl_readRegB. These control signals are fed into the 5 to 32 decoders, so that data from each registers can be controlled using the 5-bit signals. The decoded 32-bit values control which registers to be written to and read from. Along with the ctrl_writeReg signal, ctrl_writeEnable signal is also used to control if the data can be written to registers. 

32 32-bit registers are used for this project. The registers' writtings are also controlled by the clock signal. When it is at the positive edge of the clock signal, the enabled registers can be written data to.

Output data Q from the registers are then connected to the two read ports. They are used as the inputs to the 32-bit tristate buffer. Decoded control signals for the read ports are used as the enable signals for the tristate buffers. Each register output is connected to a tristate buffer. Then, according to the decoded control signal values, the specific registers are chosen to be read and output to be the data_readRegA/B values.

# Project File Structure
a) Main Files:
- regfile.v: This is the top-entity file in this project. It utilizes all subfiles and realize the main functions implementation.
- dffe.v: 32-bit register construction
- 5to32decoder.v: Decode the 5-bit control signals to be 32 bits, so that each register can be controlled independently by the control signals.
- tristate_32.v: Construct the 32-bit tristate buffer, so that the registers values can be chosen to read from.

b) 5 to 32 Decoders:

Decoders are actually left shifters in this project. The base output condition is 32'd1, and when the control signal changes, the 1 in the base output should shift left according to the control signal. This functionality is realized by the left shifer. Thus, the decoder is built using left shifter module from last project, which is SLL.v .
```Verilog
parameter [31:0] basevalue = 32'd1;
SLL sll_1(.data_operand(basevalue), .ctrl_shiftamt(in), .result(out));
```

c) Tristate:

The 32-bit tristate buffers are constructed using the "assign" statement. When the enable signal is available, the inputted data is output from the tristate buffer and used as the read register values. Otherwise, high impedance value will be outputted by the tristate buffer.
```Verilog
assign tri_out = enable_decoder ? reg_in : 32'bz;
```

d) dffe

In this file, the 32-bit register is constructed. D flip flop are used to construct registers. 'clr' and 'en' signals are used to control the D flip-flop. If 'clr' is set, then the flip-flop is reset to 0. When 'en' signal is set and it reaches positive edge of clock signal, data input d is outputted to output q. The 'en' signal in this project is the write enbale signal processed through the decoder and and gates. 
```Verilog
//Set value of q on positive edge of the clock or clear
   always @(posedge clk or posedge clr) begin
       //If clear is high, set q to 0
       if (clr) begin
           q <= 32'b0;
       //If enable is high, set q to the value of d
       end else if (en) begin
           q <= d;
       end
   end
```

e) regfile:

Finally, in the top register file, all these modules are used to construct to three ports.

i. Write
Decoders are first used to process the ctrl_writeReg to decide which registers to write data to. The decoded data are then inputted to the 32 and gates to be and with the ctrl_writeEnable signal to control if the write operation should be operated. Then, dffe modules are used 32 time to build the 32 registers and write the inputted data_writeReg data.
```Verilog
decoder5to32 decoder5to32_write(.in(ctrl_writeReg), .out(writeDec));
    and_32 and_32gate(.enable(ctrl_writeEnable), .writeD_in(writeDec), .and_out(writeReg_choose));

    dffe_ref dffe0(.clk(clock), .clr(ctrl_reset), .en(1'b0), .d(data_writeReg), .q(q[0]));
	 genvar i;
	 generate
		for (i=1; i<32; i=i+1) begin: register_loop
			dffe_ref dffe(.clk(clock), .clr(ctrl_reset), .en(writeReg_choose[i]), .d(data_writeReg), .q(q[i]));
		end
	endgenerate
```
ii. Read
Decoders are also used to process the ctrl_readRegA and ctrl_readRegB signals to choose which registers to read from for each ports. Then, 32 tristate buffers are used along with the decoded control signal to choose which registers values to read from. The outputs from tristate buffers are finally used as outputs data_readRegA and data_readRegB.  
```Verilog
 // Read Port A //
    decoder5to32 decoder5to32_readA(.in(ctrl_readRegA), .out(readDecA));
	 
	 genvar j;
	 generate
		for (j=0; j<32; j=j+1) begin: tristate_loop1
			 tristate_32 tristate_32_A(.reg_in(q[j]), .enable_decoder(readDecA[j]), .tri_out(data_readRegA));
		end
	endgenerate


    // Read Port B //
    decoder5to32 decoder5to32_readB(.in(ctrl_readRegB), .out(readDecB));
	 
	 genvar k;
	 generate
		for (k=0; k<32; k=k+1) begin: tristate_loop2
			 tristate_32 tristate_32_B(.reg_in(q[k]), .enable_decoder(readDecB[k]), .tri_out(data_readRegB));
		end
	endgenerate
```



