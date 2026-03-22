# 10-Bit Interpolated DDS Function Generator (Basys 3)

This project implements a high-precision Direct Digital Synthesizer (DDS), also known as a Numerically Controlled Oscillator (NCO), on a Xilinx Artix-7 FPGA (Basys 3). It generates a high-fidelity sine wave by combining a memory-efficient Lookup Table (LUT) with Real-Time Linear Interpolation and hardware symmetry logic.

## :rocket: Key Features

* **32-Bit Phase Accumulator**: Provides ultra-fine frequency resolution ($\approx 0.0233\text{ Hz}$ at 100MHz clock).

* **Dual-Port BRAM Architecture**: Uses Xilinx Block RAM primitives to fetch adjacent samples ($y_n$ and $y_{n+1}$) simultaneously in a single clock cycle.

* **Linear Interpolation**: Implements the fixed-point math formula $y = \frac{y_1(16-f) + y_2(f)}{16}$ using 4-bit fractional precision. This effectively increases the LUT resolution from 512 to 8,192 virtual samples.

* **Quarter-Wave Symmetry**: Stores only 0°–90° of the sine wave (512 samples) to save memory, reconstructs the full 360° cycle using hardware mirroring and sign-bit inversion.

* **XADC Integration**: Real-time frequency control (approx. 0–20kHz) via a 10k potentiometer connected to the Basys 3 JXADC header.

* **10-Bit R-2R Output**: 10-bit digital output optimized for external R-2R resistor ladders or DACs.

## :hammer_and_wrench: Technical Specifications

| Parameter | Specification |
|---|---|
| FPGA Target | Artix-7 (XC7A35T-1CPG236C) |
| Clock Frequency | 100MHZ |
| BRAM Utilization | 1 x RAMB18 |
| DSP | 1x DSP48E1 slice |
| Phase Resolution | 32-bit (Frequency Tuning Word) |
| Phase Precision | 9-bit integer address + 4-bit fraction |
| Amplitude Precision | 10 bit(0-1023) |

## :triangular_ruler: Signal Processing Pipeline

The design utilizes a 3-stage pipeline to ensure timing closure at 100MHz:

1) **Stage 1 (Accumulation)**: Updates the 32-bit phase. Calculates addr1 and   addr2 based on symmetry logic. Captures the MSB (sign) and fractional bits.
2) **Stage 2 (BRAM Read)**: Synchronous dual-port read from the BRAM. Control signals are delayed to stay aligned with the memory data.
3) **Stage 3 (Interpolation & Symmetry)**: Executes the weighted average multiplication, applies 180°–360° sign inversion, and adds the +512 DC offset.

## :file_folder: Project Structure

1) **dds.vhd**: Core DDS engine with BRAM inference and dual-address generation.
2) **top.vhd**: Top-level module including XADC instantiation, frequency scaling, and the interpolation process.
