# `HDL` Exploration

## Structure Of `Verilog`

The structure of a module is the following:

```text
module <module name> (<port list>);
<declares>
<module items>
endmodule
```

## Example

`blink.v`:

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

## Links

- <https://github.com/osresearch/fpga-class>
- <https://www.fpga4fun.com>
- <https://github.com/osresearch/up5k>
