Behrooz ZamaniPour :400412247

### Assembly.s

### Setup
main:
    li          sp,     0x3C00      # Load immediate value 0x3C00 into the stack pointer (sp)
    addi        gp,     sp,     392 # Initialize the global pointer (gp) to sp + 392
Explain
li sp, 0x3C00: Loads the immediate value 0x3C00 (15360 in decimal) into the stack pointer (sp). This sets the initial address of the stack.
addi gp, sp, 392: Adds 392 to the current value of sp and stores the result in the global pointer (gp). This sets gpto the address 0x3C00 + 392 = 0x3DC8. The gp will be used as an upper bound in the loop.

### Loop

loop:
    flw         f1,     0(sp)       # Load a 32-bit floating-point word from memory at address sp into register f1
    flw         f2,     4(sp)       # Load the next 32-bit floating-point word into register f2
       
    fmul.s      f10,    f1,     f1  # Multiply f1 by itself (squared) and store in f10
    fmul.s      f20,    f2,     f2  # Multiply f2 by itself (squared) and store in f20
    fadd.s      f30,    f10,    f20 # Add the results (f10 and f20) and store in f30
    fsqrt.s     x3,     f30         # Compute the square root of f30 and store in x3
    fadd.s      f0,     f0,     f3  # Add the result (x3) to f0 (accumulating the results)

    addi        sp,     sp,     8   # Increment the stack pointer by 8 to move to the next set of floating-point values
    blt         sp,     gp,     loop# Branch to loop if sp is less than gp (continuing the loop)
    ebreak                      # End the program execution

flw f1, 0(sp): Loads a 32-bit floating-point value from the memory address currently pointed to by sp into the floating-point register f1.
flw f2, 4(sp): Loads the next 32-bit floating-point value (4 bytes after the address in sp) into the floating-point register f2.
fmul.s f10, f1, f1: Performs single-precision floating-point multiplication of f1 with itself and stores the result in f10. This computes  f 1 2f12.
fmul.s f20, f2, f2: Similarly, computes  f 2 2f22 and stores the result in f20.
fadd.s f30, f10, f20: Adds the results of the previous multiplications ( f 1 2f12 and  f 2 2f22) and stores the sum in f30. This corresponds to calculating the sum of squares of the two components of a 2D vector.
fsqrt.s x3, f30: Computes the square root of the sum in f30 and stores the result in the integer register x3. This effectively calculates the magnitude of the vector.
fadd.s f0, f0, f3: Adds the result from x3 to the floating-point accumulator register f0, accumulating the magnitudes over iterations.
addi sp, sp, 8: Increments the stack pointer by 8 to point to the next pair of 32-bit floating-point values.
blt sp, gp, loop: Compares sp with gp. If sp is less than gp, the code branches back to the label loop, continuing the iteration.
ebreak: Triggers a breakpoint exception, which is typically used to stop program execution or enter a debugger.

### Functionality

The code iterates through a set of floating-point pairs (each pair occupying 8 bytes) in memory, starting from the address 0x3C00. For each pair, it computes the Euclidean distance (or magnitude) of a 2D vector formed by the two floating-point values:

Magnitude = ( f1 ) 2 + ( f2 ) 2Magnitude=(f1)2+(f2)2​

This result is accumulated in the floating-point register f0. The loop continues until the stack pointer (sp) reaches or exceeds the address stored in gp (0x3C00 + 392).

### Observations and Notes

Memory Layout: The code assumes that the memory starting at 0x3C00 contains 32-bit floating-point numbers in pairs. This layout should be ensured before running this program.
Floating-Point Registers: The code utilizes single-precision floating-point registers (f1, f2, f10, f20, f30) and operations (flw, fmul.s, fadd.s, fsqrt.s).
Loop Control: The loop control relies on the stack pointer (sp) and global pointer (gp). The blt instruction ensures that the loop terminates after processing all pairs up to the specified memory range.
Accumulation: The final result in f0 will be the accumulated sum of the magnitudes of all the 2D vectors processed.




### Fixed_Point_Unit Module Report

### Module Parameters

WIDTH: Specifies the total bit-width of the operands and the result. Default is 32 bits.
FBITS: Specifies the number of fractional bits in the fixed-point representation. Default is 10 bits.
Ports
Inputs:

clk: Clock signal.
reset: Reset signal.
operand_1: First operand for the arithmetic operations.
operand_2: Second operand for the arithmetic operations.
operation: 2-bit signal to select the operation (ADD, SUB, MUL, SQRT).
Outputs:

result: The output of the arithmetic operation.
ready: Indicates when the result is ready.

### Operation Selection

The module supports the following operations, selected via the operation input:

FPU_ADD (Addition)
FPU_SUB (Subtraction)
FPU_MUL (Multiplication)
FPU_SQRT (Square Root)
These operations are defined in an external file Defines.vh, which must include the appropriate macros for operation codes.

### Core Logic

Addition and Subtraction
These operations are straightforward. The always @(\*) block performs addition or subtraction directly based on the operation input:

For FPU_ADD, result is the sum of operand_1 and operand_2.
For FPU_SUB, result is the difference between operand_1 and operand_2.
In both cases, ready is set to 1 indicating immediate availability of the result.

### Multiplication

Multiplication is implemented through a sequential state machine and a dedicated multiplier module (Multiplier). The multiplication is performed in several steps:

State Initialization: In the INITIALSTATE, the multiplication starts by setting up the inputs for the first partial product calculation.
Partial Product Calculation: In states FIRSTMUL, SECONDMUL, THIRDMUL, and FOURTHMUL, the module calculates partial products by multiplying different parts (low and high 16-bit sections) of the operands.
Addition of Partial Products: In the ADD state, all partial products are summed to form the final product.
Final Product: The final product is stored in product with product_ready indicating the completion of the multiplication.
The multiplier module (Multiplier) handles the multiplication of 16-bit segments and outputs a 32-bit product.

### Square Root Calculation

The square root operation is more complex and is handled by another state machine:

Initialization: In the INITIAL state, necessary variables are set up.
Iterative Calculation: In the SQRT state, the module uses an iterative approach to compute the square root of operand_1 based on digit-by-digit calculation.
Completion: Once the iterations are complete, the result is stored in root and root_ready is set to 1.

### State Machines

The module employs two state machines:

Square Root State Machine: Controls the steps of the square root calculation, switching between INITIAL and SQRT states.
Multiplication State Machine: Manages the step-by-step calculation of the multiplication process, transitioning through states INITIALSTATE, FIRSTMUL, SECONDMUL, THIRDMUL, FOURTHMUL, and ADD.

### Reset Behavior

On reset (reset signal is high), the state machines return to their initial states.
The ready signal and internal registers used for intermediate computations are also reset.

### Special Considerations

Tri-State Logic: The use of 'bz (high impedance) in the default case of the operation selection indicates that the result and ready signals are not driven during non-selected operations. This could be intentional for systems using tri-state logic but may require careful handling in practical designs.

### Multiplication Details

The multiplication process leverages the Multiplier module to perform partial product calculations. Each partial product is shifted accordingly before summing, allowing for accurate fixed-point multiplication:

Partial Products: Calculated for combinations of low and high 16-bit sections of the operands.
Shifting: Adjusts the partial products' position to align them for final summation.
Summation: Aggregates all partial products to form the final 64-bit product.
