+++
title = "load iNES"
description = "Load iNES file to NES"
date = 2018-02-28
tags = ["NES"]
+++

## First of all
NES is Nintendo Entertainment System a.k.a. 'ファミコン'.
My goal is developing NES simulator using Rust language.
Reference guide is [here](http://wiki.nesdev.com/w/index.php/NES_reference_guide).

NES consists of some parts. For example CPU, PPU, APU and so on. I start from loading iNES file,  which is a format of NES binary programs. You can check iNES format [here](http://wiki.nesdev.com/w/index.php/INES).

## Two parts of iNES
iNES have two parts. PRG_ROM and CHR_ROM. PRG_ROM is for game logic and CHR_ROM is for graphic. When iNES is loaded, PRG_RAM load PRG_ROM data and V_RAM load CHR_ROM.

## PRG_RAM and V_RAM
PRG_RAM and V_RAM has 2KB memory.

## init project
~~~
% cargo init --bin nes
% cd nes
~~~

## nes module
This is the today's directory structure.
~~~
% tree
.
├── lib.rs
├── main.rs
└── nes
    ├── cpu
    │   └── mod.rs
    ├── mod.rs
    └── ppu
        └── mod.rs
~~~

### CPU
`src/nes/cpu/mod.rs`
```rust
/// [CPU](http://wiki.nesdev.com/w/index.php/CPU)
pub struct Cpu {
    prg_ram: PrgRam,
}

impl Cpu {
    pub fn new(prg_ram: PrgRam) -> Cpu {
        Cpu { prg_ram }
    }
}

/// [CPU memory](http://wiki.nesdev.com/w/index.php/CPU_memory_map)
/// RAM is 2KB.
pub struct PrgRam {
    memory: Box<[u8; 0xFFFF]>,
    v_ram: Arc<Mutex<VRam>>,
}

impl PrgRam {
    pub fn new(memory: Box<[u8; 0xFFFF]>, v_ram: Arc<Mutex<VRam>>) -> PrgRam {
        PrgRam { memory, v_ram }
    }
}
```
PrgRam doesn't have to has VRam at this time, but We need it on a further section.

### PPU
`src/nes/ppu/mod.rs`
```rust
/// [PPU](http://wiki.nesdev.com/w/index.php/PPU) is short for Picture Processing Unit
pub struct Ppu {
    v_ram: Arc<Mutex<VRam>>,
}

impl Ppu {
    pub fn new(v_ram: Arc<Mutex<VRam>>) -> Ppu {
        Ppu { v_ram }
    }
}

/// [PPU memory](http://wiki.nesdev.com/w/index.php/PPU_memory_map)
/// RAM is 2KB.
pub struct VRam(Box<[u8; 0xFFFF]>);
impl VRam {
    pub fn new(memory: Box<[u8; 0xFFFF]>) -> VRam {
        VRam(memory)
    }
}
```
### NES
`src/nes/mod.rs`
``` rust
mod cpu;
mod ppu;

use std::fs::File;
use std::path::Path;
use std::io::{BufReader, Read};
use std::sync::{Arc, Mutex};
use nes::cpu::{Cpu, PrgRam};
use nes::ppu::{Ppu, VRam};

pub struct Nes {
    cpu: Cpu,
    ppu: Ppu,
}
impl Nes {
    const I_NES_HEADER_SIZE: u16 = 0x0010;
    const COUNT_OF_PRG_ROM_UNITS_INDEX: u16 = 4;
    const COUNT_OF_CHR_ROM_UNITS_INDEX: u16 = 5;
    const SIZE_OF_PRG_ROM_UNIT: u16 = 0x4000;
    const SIZE_OF_CHR_ROM_UNIT: u16 = 0x2000;

    const RAM_SIZE: usize = 0x10000;
    const PRG_ROM_LOWER_IDX: usize = 0x8000;
    const PRG_ROM_UPPER_IDX: usize = 0xC000;

    pub fn new(casette_name: &str) -> Nes {
        let path_string = format!("cassette/{}", String::from(casette_name));
        let path = Path::new(&path_string);

        let (prg_ram, v_ram) = Self::load_ram(path);
        let cpu = Cpu::new(prg_ram);
        let ppu = Ppu::new(v_ram.clone());

        Nes { cpu, ppu }
    }

    /// load [iNES](http://wiki.nesdev.com/w/index.php/INES)
    fn load_ram(path: &Path) -> (PrgRam, Arc<Mutex<VRam>>) {
        let file = match File::open(path) {
            Ok(file) => file,
            Err(why) => panic!("{}: path is {:?}", why, path),
        };
        let mut buffer = BufReader::new(file);
        let mut i_nes_data: Vec<u8> = Vec::new();
        let _ = buffer.read_to_end(&mut i_nes_data).unwrap();

        let prg_rom_start = Self::I_NES_HEADER_SIZE;
        let prg_rom_end = prg_rom_start +
            i_nes_data[Self::COUNT_OF_PRG_ROM_UNITS_INDEX as usize] as u16 * Self::SIZE_OF_PRG_ROM_UNIT - 1;

        let chr_rom_banks_num = i_nes_data[Self::COUNT_OF_CHR_ROM_UNITS_INDEX as usize];
        let chr_rom_start = prg_rom_end + 1;
        let chr_rom_end = chr_rom_start + chr_rom_banks_num as u16 * Self::SIZE_OF_CHR_ROM_UNIT - 1;

        // set prg_ram_memory
        let mut prg_rom: Vec<u8> = Vec::new();
        prg_rom.extend_from_slice(
            &i_nes_data[(prg_rom_start as usize)..(prg_rom_end as usize + 1)],
        );

        let mut prg_ram_memory: Box<[u8; Self::RAM_SIZE]> = Box::new([0; Self::RAM_SIZE]);
        prg_ram_memory[Self::PRG_ROM_LOWER_IDX..Self::PRG_ROM_UPPER_IDX]
            .clone_from_slice(&prg_rom[..(Self::SIZE_OF_PRG_ROM_UNIT as usize)]);

        match i_nes_data[Self::COUNT_OF_PRG_ROM_UNITS_INDEX as usize] {
            1 => {
                prg_ram_memory[Self::PRG_ROM_UPPER_IDX..].clone_from_slice(
                    &prg_rom[..(Self::SIZE_OF_PRG_ROM_UNIT as usize)],
                );

            }
            2...255 => {
                prg_ram_memory[Self::PRG_ROM_UPPER_IDX..].clone_from_slice(
                    &prg_rom[(Self::SIZE_OF_PRG_ROM_UNIT as usize)..(Self::SIZE_OF_PRG_ROM_UNIT as usize * 2)],
                );
            }
            _ => {}
        }

        // set v_ram_memory
        let mut v_ram_memory: Box<[u8; Self::RAM_SIZE]> = Box::new([0; Self::RAM_SIZE]);
        v_ram_memory[..(chr_rom_end as usize + 1 - chr_rom_start as usize)].clone_from_slice(
            &i_nes_data[(chr_rom_start as usize)..(chr_rom_end as usize + 1)],
        );

        let v_ram = Arc::new(Mutex::new(VRam::new(v_ram_memory)));
        let prg_ram = PrgRam::new(prg_ram_memory, v_ram.clone());

        (prg_ram, v_ram)
    }
}

```
`load_ram` method is today's main topic.
1. Parse an iNES format file.
2. Check the header.
  - 4th Byte in the header means count of PRG ROM *units*.
      - A PRG ROM unit is 16KB.
  - 5th Byte in the header means count of CHR ROM *units*.
      - A CHR ROM unit is 8KB.
3. Load PRG ROM data to PrgRam.
4. Load CHR ROM data to VRam.

## dump
Dump PrgRam and VRam to check they store data.
```rust
// src/nes/cpu/mod.rs
impl Cpu {
    #[allow(dead_code)]
    pub fn dump(&self) {
        self.prg_ram.dump();
    }
}

impl PrgRam {
    pub fn dump(&self) {
        let dump_file = "prg_ram.dump";
        let mut f = BufWriter::new(File::create(dump_file).unwrap());
        for v in self.memory.iter() {
            f.write(&[*v]).unwrap();
        }
    }
}


// src/nes/ppu/mod.rs
impl Ppu {
    #[allow(dead_code)]
    pub fn dump(&self) {
        self.v_ram.lock().unwrap().dump();
    }
}

impl VRam {
    fn dump(&self) {
        let dump_file = "v_ram.dump";
        let mut f = BufWriter::new(File::create(dump_file).unwrap());
        for v in self.0.iter() {
            f.write(&[*v]).unwrap();
        }
    }
}


// src/nes/mod.rs
    pub fn new(casette_name: &str) -> Nes {
       // load iNES

        #[cfg(feature = "dump")] cpu.dump();
        ppu.dump();

        Nes { cpu, ppu }
    }


// src/main.rs
extern crate nes;

use nes::nes::Nes;

fn main() {
    let _nes = Nes::new("sample1.nes");
}


// Cargo.toml
[features]
dump = []
```
Sample iNES file is [here](http://hp.vector.co.jp/authors/VA042397/nes/sample.html)(Download from a first link).

Unzip the file and put `sample1.nes` at `cassette/`.

After `cargo run --features dump`, you can check `prg_ram.dump` and `v_ram.dump` using hex editor such as [0xED](http://www.suavetech.com/0xed/) and [hexedit](https://sourceforge.net/projects/hexedit/).


Whole source code is available [here](https://github.com/k-o-ta/nes/tree/load-iNES-file).
