`HDL`: Hardware Description Language
====================================

That is, languages that describes the circuits (wires, registers, combinational
logic, clocks, etc.). In other words, specification of hardware. The simulator
or synthesizer can take in the spec and simulate actual hardware behavior. We
do not need to program `wire` and `registers` with this and that properties
before being able to use them. Those are provided to us as primitives and
we describe `A` is wire and properties of wire would be assigned to `A`.

`Verilog` is the older language from the 1980s. It describes digital circuits
using modules, wires, registers, and always blocks. It is widely used for
`FPGA` and `ASIC` work.

The structure of a module is the following:

```text
module <module name> (<port list>);
<declares>
<module items>
endmodule
```

Example of `and` gate:

```verilog
module and_gate(
    input a,
    input b,
    output y
);
assign y = a & b;
endmodule
```

A better example:

```vh
// modules are like functions but they create hardware.
module top(
  output led_r // Arguments can be input, output or inout PCF file defines pin mappings
);
  wire clk; // wire is like a local variable, (except it doesn’t hold state)
  SB_HFOSC osc(1,1,clk); // Creates a “black box” module HFOSC = High-Frequency oscillator, Output 48 MHz into the clk wire
  reg [25:0] counter; // reg declares a local variable that holds state between clocks. (Can be a single bit or a vector)
                      // 26 bit register. 2^26 == 67,108,864

  always @(posedge clk)
    counter <= counter + 1; // Increments counter on positive edges of clk At 48 MHz, counter overflows every 1.4 s

  assign led_r = counter[25]; // Continuous assignment to the led_r output wire (not clocked)
                              // Turns on the LED when top bit is 0 (50% dutycycle, negative logic)
endmodule
```

Here, we use the primitives (input, output) and assign them to variables. The
output variable has it's property (i.e. `&` operation over `a` and `b`).

Other options include `SystemVerilog` & `VHDL`.

`SystemVerilog` is a modern extension of `Verilog` and `VHDL` is a stricter
and verbose language used by defense and aerospace industry primarily.

Some Fundamental Concepts
-------------------------

**Transistors**

Transistors are basically a controlled switch. When the gate voltage is high,
the transistor conducts. When gate voltage is low, it does not. Modern
CPUs use CMOS transistors. There are two types of MOSFET used in CMOS.

- `NMOS`: good at pulling the signal down to 0.
- `PMOS`: good at pulling the signal up to 1.

In CMOS, `PMOS` transistors are placed in parallel at the top and `NMOS`
transistors are placed in series at the bottom.

watch:

- [MOSFET Explained -How MOSFET Works](https://www.youtube.com/watch?v=AwRJsze_9m4)
- [CMOS Transistors](https://www.youtube.com/watch?v=K4IXY3f-Smw)

> The earliest microprocessors starting in 1970 were all MOS microprocessors,
> fabricated entirely from PMOS logic or fabricated entirely from nMOS logic.
> In the 1970s, MOS microprocessors were often contrasted with CMOS
> microprocessors and bipolar bit-slice processors.

- [Building logic gates from MOSFET transistors](https://www.youtube.com/watch?v=1rZyGL1K5QI)
- [Transistor Logic Gates - NAND, AND, OR, NOR](https://www.youtube.com/watch?v=OWlD7gL9gS0)
- [Implementation of Boolean Expressions using CMOS - NMOS, PMOS, Transistors, Examples, Explained](https://www.youtube.com/watch?v=iWcQN8duQpY)

**Logic Gates**

- [How Transistors Run Code](https://www.youtube.com/watch?v=HjneAhCy2N4)

A logic gate is a device that performs a Boolean function.

> Boolean function means takes in arbitrary number of binary input and produces
> a single binary output.

The primary way of building logic gates uses diodes or transistors acting
as electronic switches. Most logic gates are made from MOSFETs.

> They can also be constructed using vacuum tubes, electromagnetic relays with
> relay logic, fluidic logic, pneumatic logic, optics, molecules, acoustics,
> or even mechanical or thermal elements.

Logic gates can be cascaded which allows for construction of a physical model
of all Boolean logic. Logic circuits includes devices such as multiplexers,
registers, arithmetic logic units (ALUs), and computer memory, all the way up
through complete microprocessors, which may contain more than 100 million
logic gates.

> There are seven basic logic gates: NOT, OR, NOR, AND, NAND, XOR, XNOR.

1. Combinational Logic

  Type of digital logic that is implemented by Boolean circuits, where the
  output is a pure function of the present input only. Examples: adders,
  muxes, gates.
  ```verilog
  assign y = a ^ b
  ```

  see: [half-adder implementation](./src/half-adder.v)

2. Sequential Logic

  Sequential logic is a type of logic circuit whose output depends on the
  present value of its input signals and on the sequence of past inputs, the
  input history. Or in simpler words the output depends on state stored in
  registers.
  > Registers update on clock edges.
  ```verilog
  always @(posedge clk) begin
    q <= d;
  end
  ```
  This describes a flip-flop.
  > `<=` is non-blocking assignment used for registers.

  see: [counter implementation](./src/count.v)

3. Modules

  hardware is built hierarchically:
  ```verilog
  module adder(
    input [31:0] a,
    input [31:0] b,
    output [31:0] y
  );
  assign y = a + b;
  endmodule
  ```
  Then another module can instantiate it:
  ```verilog
  adder myadder(
    .a(x),
    .b(y),
    .y(z)
  );
  ```
  This is just connecting circuits together.

Some Basic Syntax
-----------------

**Signals**

```verilog
wire a; reg b;
```

**Bit vectors**

```verilog
wire [7:0] data;
```

**Clocked block**

```verilog
always @(poseedge clk)
```

**Combinational blocks**

```verilog
always @(*)
```

Simulation and Testing
----------------------

We write a `testbench` that feeds signals to our design and checks outputs.
Example:

```verilog
module tb;

reg clk;
reg a;
reg b;
wire y;

and_gate dut(
  .a(a),
  .b(v),
  .y(y)
);

initial begin
  a = 0;
  b = 0;

  #10 a = 1;
  #10 b = 1;
end

endmodule
```

This can be run with tools like `icarus verilog`, `verilator`. It then can be
viewed with `GTKWave`

`uart` Verification/Simulation
----------------------------

```sh
iverilog -g2012 src/uart.sv testbench/uart.v -o sim.out
vvp sim.out
gtkwave uart.vcd
```

## Links

- <https://github.com/osresearch/fpga-class>
- <https://www.fpga4fun.com>
- <https://github.com/osresearch/up5k>
