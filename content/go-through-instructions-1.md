+++
title = "Go Through Instructions 1"
description = "Go Through Instructions 1"
date = 2018-03-01
tags = ["NES"]
+++

### Goal
Go through all instructions in `sample1.nes`.

CPU instruction is
1. fetch a instruction
  1. fetch a instruction
  2. increment PC
2. parse the instruction into OperationCode, AddressingMode, IndexRegister(optional).
3. execute operation

### fetch a instruction
Fetching a instruction from `PrgRam` and address is indicated by ProgramCounter.

`src/nes/cpu/mod.rs`
```rust
pub struct Cpu {
    /// [Registers](http://wiki.nesdev.com/w/index.php/CPU_registers)
    // Program Counter
    pc: u16,

    prg_ram: PrgRam,
}

impl Cpu {
    fn fetch_instruction(&self) -> u8 {
        self.prg_ram_value()
    }

    fn prg_ram_value(&self) -> u8 {
        self.prg_ram.memory[self.pc as usize]
    }
}
```

Incrementing ProgramCounter is simple.

`src/nes/cpu/mod.rs`
```rust
impl Cpu {
    fn increment_pc(&mut self) {
        self.pc = self.pc + 1;
    }
}
```
### parse the instruction
Parsing the instruction is bit more complex. We have to know [OperationCode(OpCode)](http://wiki.nesdev.com/w/index.php/6502_instructions), [AddressingMode](http://wiki.nesdev.com/w/index.php/CPU_addressing_modes) and [IndexRegister](http://wiki.nesdev.com/w/index.php/CPU_registers).

An instruction identify a set of OpCode, AddressingMode and IndexRegister.([ref](http://wiki.nesdev.com/w/index.php/CPU_unofficial_opcodes))

These are list of OpCode, AddressingMode, and IndexRegister.

`src/nes/cpu/mod.rs`
```rust
#[derive(Debug)]
/// [OperationCode](http://wiki.nesdev.com/w/index.php/6502_instructions)
enum OpCode {
    ADC,
    SBC,
    AND,
    ORA,
    EOR,
    ASL,
    LSR,
    ROL,
    ROR,
    BCC,
    BCS,
    BEQ,
    BNE,
    BVC,
    BVS,
    BPL,
    BMI,
    BIT,
    JMP,
    JSR,
    RTS,
    BRK,
    RTI,
    CMP,
    CPX,
    CPY,
    INC,
    DEC,
    INX,
    DEX,
    INY,
    DEY,
    CLC,
    SEC,
    CLI,
    SEI,
    CLD,
    SED,
    CLV,
    LDA,
    LDX,
    LDY,
    STA,
    STX,
    STY,
    TAX,
    TXA,
    TAY,
    TYA,
    TSX,
    TXS,
    PHA,
    PLA,
    PHP,
    PLP,
    NOP,
}

#[derive(Debug)]
/// [AddressingMode](http://wiki.nesdev.com/w/index.php/CPU_addressing_modes<Paste>)
enum AddressingMode {
    Accumulator,
    Immediate,
    Absolute,
    ZeroPage,
    IndexedZeroPage,
    IndexedAbsolute,
    Implied,
    Relative,
    IndexedIndirect,
    IndirectIndexed,
    AbsoluteIndirect,
}

#[derive(Debug)]
enum IndexRegister {
    X,
    Y,
}

```

Let's start below.

```rust
// Cargo.toml
[dependencies]
error-chain = "0.11.0"

// src/nes/mod.rs
impl Nes {
  pub fn run(mut self) {
    self.cpu.run();
  }
}

// src/nes/cpu/mod.rs
use errors::*;

impl Cpu {
  pub fn new(prg_ram: PrgRam) -> Cpu {
    Cpu {
      pc: 0x8000,
      prg_ram,
    }
  }

  pub fn run(mut self) {
    loop {
      let instruction = self.fetch_instruction();
      match Self::parse_instruction(instruction) {
        Ok((op_code, addressing_mode, register)) => {
          self.increment_pc();
        }
        Err(error_message) => {
          panic!("{}, at: {:0x}, cpu_dump: {:?}", error_message, self.pc, self);
        }
      }
    }
  }

  fn fetch_instruction(&self) -> u8 {
    self.prg_ram_value()
  }

  fn parse_instruction(instruction: u8,) -> Result<(OpCode, AddressingMode, Option<IndexRegister>)> {
      let (op_code, addressing_mode, register): (OpCode,
						 AddressingMode,
						 Option<IndexRegister>) = match instruction {
	  _ => Err(format!("unknown instruction: {:0x}", instruction,))?,
      };
      Ok((op_code, addressing_mode, register))
  }

  impl fmt::Debug for Cpu {
      fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
          write!(f, "A:{:0x}, PC:{:0x}", self.pc)
      }
  }
}

```
If you run `cargo run`, you get `panicked at 'unknown instruction: 78, at: 8000, cpu_dump: A:0, X:0, Y:0, PC:8000'`.
It means instruction is 78, but you have not implemented yet. So, you have to implement all instructions.

I suggest using [cargo watch](https://github.com/passcod/cargo-watch), to implemnt instructions.
```rust
// src/nes/mod.rs
#[cfg(test)]
mod tests {
    use super::*;
    #[test]
    fn run_test() {
        let mut nes = Nes::new("sample1.nes");
        nes.run();
    }
}
```
`cargo watch -x test` watch source code change and run compile and test automatically. If you implement instruction 78, test run and you get `unknown instruction: a2, at: 8001, cpu_dump: PC:8001'` because next instruction is `a2`.


Whole source code is available [here](https://github.com/k-o-ta/nes/tree/go-through-instructions-1).
