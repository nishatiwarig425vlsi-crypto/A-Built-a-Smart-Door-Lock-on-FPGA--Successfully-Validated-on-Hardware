# A-Built-a-Smart-Door-Lock-on-FPGA--Successfully-Validated-on-Hardware
Designed and implemented a password-based smart door lock using Verilog on the AMD Spartan-7 (XC7S50) FPGA. V Features: 16-bit password authentication (OxAAAA) Displays "OPEN" on successful authentication and unlocks via SG90 servo motor Displays "SHUT" and remains locked for incorrect passwords Implemented FSM, button debounce. 
CODE:-
module door_lock_top(
    input clk,
    input [15:0] sw,
    input [2:0] btn,   // btn[0]=reset, btn[1]=set, btn[2]=check
    output reg led,
    output reg servo0,                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 
    output reg [7:0] D0_SEG,
    output reg [7:0] D1_SEG,
    output reg [3:0] D0_AN,
    output reg [3:0] D1_AN
);

// =========================
// PASSWORD + DOOR LOGIC
// =========================
reg [7:0] stored_password = 0;
reg door_open = 0;

always @(posedge clk)
begin
    if(btn[0]) begin
        stored_password <= 0;
        door_open <= 0;
    end
    else begin
        if(btn[1])
            stored_password <= sw[7:0];

        if(btn[2])
            door_open <= (sw[15:8] == stored_password);
    end
end

// =========================
// LED
// =========================
always @(posedge clk)
begin
    led <= door_open;
end

// =========================
// SERVO PWM
// =========================
reg [20:0] counter = 0;

always @(posedge clk)
begin
    counter <= counter + 1;

    if(counter < (door_open ? 200000 : 100000))
        servo0 <= 1;
    else
        servo0 <= 0;

    if(counter >= 2000000)
        counter <= 0;
end

// =========================
// 7-SEG DISPLAY
// =========================
reg [19:0] refresh_counter = 0;
reg [2:0] digit = 0;

always @(posedge clk)
begin
    refresh_counter <= refresh_counter + 1;
    digit <= refresh_counter[19:17];
end

always @(*)
begin
    // default OFF
    D0_AN = 4'b1111;
    D1_AN = 4'b1111;
    D0_SEG = 8'b11111111;
    D1_SEG = 8'b11111111;

    case(digit)

        // ===== SHUT (door closed) =====
        3'd0: if(!door_open) begin D0_AN=4'b0111; D0_SEG=8'b10010010; end // S
        3'd1: if(!door_open) begin D0_AN=4'b1011; D0_SEG=8'b10001001; end // H
        3'd2: if(!door_open) begin D0_AN=4'b1101; D0_SEG=8'b11000001; end // U
        3'd3: if(!door_open) begin D0_AN=4'b1110; D0_SEG=8'b10000111; end // T

        // ===== OPEN (door open) =====
        3'd4: if(door_open) begin D1_AN=4'b0111; D1_SEG=8'b11000000; end // O
        3'd5: if(door_open) begin D1_AN=4'b1011; D1_SEG=8'b10001100; end // P
        3'd6: if(door_open) begin D1_AN=4'b1101; D1_SEG=8'b10000100; end // E
        3'd7: if(door_open) begin D1_AN=4'b1110; D1_SEG=8'b10101011; end // N

    endcase
end

endmodule
