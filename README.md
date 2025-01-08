# Micro-Processo
a two player game that player 1 put a number and in 10 rounds player 2 have to guess the number correctly the catch is player2 after each guess will get a hint with leds if red or yellow led turn on its mean the number player 2 guess is less or bigger so he have 5s to cahnge the number while the white led is on and after 5s the answer of player2 would compare agian with the main answer if he guess correctly the green LED will turn on and the game will end otherwise after 10 round player 1 would win the game and all 4 LED would turn on for 5s and then the game would finish and we need to press start_game button for another match!
note: the game will run in proteus correctly.
----------------------------------------------------------------------------------------------------------------------
code explaination:
The code provided is written in **AVR assembly language** for microcontrollers in the AVR family, such as the Atmel ATmega series. It defines interrupt service routines (ISRs), timer configurations, and I/O port manipulations to control external peripherals and manage the system's logic.

Here’s an explanation of its components:

---

### **Header File and Memory Mapping**
```assembly
.INCLUDE "m64def.inc"
.ORG 0x0000
JMP Main
```
- `.INCLUDE "m64def.inc"`: Includes definitions for the AVR microcontroller (e.g., registers, flags).
- `.ORG 0x0000`: Places the `JMP Main` instruction at the reset vector, so the program starts executing at `Main` after reset.

---

### **Interrupt Vectors**
```assembly
.ORG 0x000C
JMP INT5_ISR
.ORG 0x0020
JMP TIMER0_OVF_ISR
```
- Interrupt vectors are set for **INT5** and **Timer0 Overflow**.
  - INT5_ISR handles external interrupts triggered by INT5 (pin interrupt).
  - TIMER0_OVF_ISR handles overflow of Timer0.

---

### **Main Program**
#### **Stack Pointer Initialization**
```assembly
Main:
    LDI   R16, HIGH(RAMEND)
    OUT   SPH, R16
    LDI   R16, LOW(RAMEND)
    OUT   SPL, R16
```
- Initializes the **Stack Pointer** to the top of SRAM.

#### **Registers Initialization**
```assembly
    LDI   R17, 0  ; COUNTER ROUNDS
    LDI   R18, 0  ; COUNTER 1 SEC
    LDI   R19, 0  ; COUNTER 5 SEC
    LDI   R21, 0  ; PLAYER 1,2 WINS
    LDI   R22, 0x10 ; Used in part of game logic
```
- Sets up counters and game-related registers.

#### **Timer0 Configuration**
```assembly
    LDI   R16, 6
    OUT   TCNT0, R16
    LDI   R16, 0x07
    OUT   TCCR0, R16
    LDI   R16, 0x81
    OUT   TIMSK, R16
```
- Initializes **Timer0**:
  - Sets `TCNT0` to 6 (starting value).
  - Configures `TCCR0` for a clock source and prescaler.
  - Enables Timer0 Overflow Interrupt in `TIMSK`.

#### **INT5 Configuration**
```assembly
    LDI   R16, 0x20
    OUT   EIMSK, R16
    LDI   R16, 0x08
    OUT   EICRB, R16
```
- Enables external interrupt **INT5** and configures it to trigger on a specific condition.

#### **I/O Ports Configuration**
```assembly
    LDI   R16, 0x0F
    OUT   DDRC, R16
    OUT   PORTC, R16  
    ...
    OUT   PORTD, R16
```
- Configures data direction and initial values for various ports (C, D, B, etc.).

---

### **Interrupt Service Routines (ISRs)**

#### **INT5_ISR**
```assembly
INT5_ISR:
    SEI
    OUT  PORTB, R21
    CLR  R17
    CLR  R18
    CLR  R19
    CLR  R23
    JMP  CHECK_ANSWER
```
- Handles INT5 by resetting counters, storing a value, and branching to game logic (`CHECK_ANSWER`).

#### **CHECK_ANSWER**
```assembly
CHECK_ANSWER:
    INC   R17
    OUT   PORTE, R17
    LDI   R16, 0x0E
    OUT   PORTC, R16
    ...
```
- Implements logic to compare input (`PINA` and `PIND`) and decide the next game action:
  - **AB**: Branch for incorrect answer.
  - **AE**: Branch for correct answer.
  - **AS**: Another game condition.
  - Includes timing and counter-based delays (`WAIT_JAVAB`, `WAIT_B`, etc.).

#### **TIMER0_OVF_ISR**
```assembly
TIMER0_OVF_ISR:
    LDI   R16, 6
    OUT   TCNT0, R16
    INC   R18
    INC   R19
```
- Resets Timer0 counter and increments time-related counters.

---

### **Game Logic**
#### **ETMAM_GAME**
```assembly
ETMAM_GAME:
    LDI   R16, 0x00
    OUT   PORTC, R16
    ...
    ADD  R21, R22
    RETI
```
- Finalizes the game logic:
  - Adjusts bonus logic using `OCR2`.
  - Updates output registers.
  - Signals end of game and prepares for next round.

---

### **Key Highlights**
1. **Timers and Counters**:
   - Timer0 is used to track system timing (overflow every fixed interval).
   - Registers `R17`, `R18`, and `R19` maintain time and round counters.

2. **I/O Operations**:
   - `PORTC`, `PINA`, and `PIND` are used to handle external devices.
   - Interrupt-driven events toggle LEDs or other devices.

3. **Interrupt-Driven Design**:
   - ISRs handle real-time events for external input and timing.

4. **Game Logic**:
   - Implements a reaction or game system where player input is compared, with counters managing timing and scores.

---

This program demonstrates AVR assembly for controlling hardware peripherals, real-time interrupt handling, and game-like logic processing.
