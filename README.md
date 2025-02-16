# INSTRUCTIONS-DESIGN-A-4-STAGE-PIPELINED-PROCESSOR-WITH-BASIC-INSTRUCTIONS-LIKE-ADD-SUB-AND-LOAD.
# COMPANY: CODTECH IT SOLUTIONS 
# NAME: NEHA NARAYAN RAJEGORE
# INTERN ID:CT08JTK
# DOMAIN:VLSI (Very Large Scale Integration)
# BATCH DURATION:JANUARY30TH,2025 TO MARCH 1ST,2025 
# MANTOR NAME:NEELA SANTHOSH KUMAR 
# DESCRIPTION OF TASK:  Here's a basic 4-stage pipelined processor design in Verilog with instructions for ADD, SUB, and LOAD. The four pipeline stages are
Fetch (IF): Fetch the instruction from memory.
Decode (ID): Decode the instruction and read registers.
Execute (EX): Perform arithmetic or address calculation.
Writeback (WB): Write results back to registers.
# Processor Design:
module PipelinedProcessor (
    input wire clk, reset
);

    // Instruction Format (Simple 8-bit): [Opcode (2) | Rs (2) | Rt (2) | Rd (2)]
    reg [7:0] instruction_mem [0:15];  // Instruction Memory
    reg [7:0] regfile [0:3];           // 4 General Purpose Registers
    reg [7:0] data_mem [0:15];         // Data Memory

    reg [7:0] IF_ID_instr;             // IF/ID Pipeline Register
    reg [7:0] ID_EX_instr, ID_EX_A, ID_EX_B;
    reg [7:0] EX_WB_result;
    reg [1:0] EX_WB_rd;
    
    // Instruction Fetch (IF)
    integer pc = 0;
    always @(posedge clk or posedge reset) begin
        if (reset) pc <= 0;
        else begin
            IF_ID_instr <= instruction_mem[pc]; // Fetch instruction
            pc <= pc + 1;
        end
    end

    // Instruction Decode (ID)
    always @(posedge clk) begin
        ID_EX_instr <= IF_ID_instr;
        ID_EX_A <= regfile[IF_ID_instr[5:4]]; // Rs
        ID_EX_B <= regfile[IF_ID_instr[3:2]]; // Rt
    end

    // Execute (EX)
    reg [7:0] ALU_result;
    always @(posedge clk) begin
        case (ID_EX_instr[7:6]) // Opcode
            2'b00: ALU_result <= ID_EX_A + ID_EX_B; // ADD
            2'b01: ALU_result <= ID_EX_A - ID_EX_B; // SUB
            2'b10: ALU_result <= data_mem[ID_EX_A]; // LOAD
        endcase
        EX_WB_rd <= ID_EX_instr[1:0]; // Destination Register
        EX_WB_result <= ALU_result;
    end

    // Write Back (WB)
    always @(posedge clk) begin
        regfile[EX_WB_rd] <= EX_WB_result;
    end

endmodule

# Testbench with Simulation

module Testbench;
    reg clk, reset;
    PipelinedProcessor uut (.clk(clk), .reset(reset));

    // Clock Generation
    always #5 clk = ~clk;

    initial begin
        clk = 0; reset = 1;
        #10 reset = 0;

        // Load instructions (ADD R0, R1, R2), (SUB R3, R2, R1), (LOAD R1, 0x02)
        uut.instruction_mem[0] = 8'b00001010; // ADD R2 = R0 + R1
        uut.instruction_mem[1] = 8'b01011001; // SUB R1 = R3 - R2
        uut.instruction_mem[2] = 8'b10010000; // LOAD R0 = MEM[R1]
        
        uut.regfile[0] = 8; // R0 = 8
        uut.regfile[1] = 2; // R1 = 2
        uut.regfile[3] = 10; // R3 = 10

        uut.data_mem[2] = 20; // MEM[2] = 20

        #50 $stop;
    end
endmodule

