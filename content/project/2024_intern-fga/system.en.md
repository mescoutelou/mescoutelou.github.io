---
archetype: "page"
title: "System description"
weight: 1
draft: false
---

![Complete FPGA graphics system](/fig/project/2024_intern-fpga/vga-mem.en.png)

## Table of Contents
- [Table of Contents](#table-of-contents)
- [References](#references)

The goal of this project is to design a fully functional graphics system on a FPGA (Nexys A7 100T).
This system must integrate a VGA interface and use a DDR memory as a frame buffer.
In the first draft, the DDR interface is implemented using the Vivado MIG design.

## References

- [Project F: Beginning FPGA Graphics](https://projectf.io/posts/fpga-graphics/)
- [Project F: Video timings](https://projectf.io/posts/video-timings-vga-720p-1080p/)
- [Project F: Framebuffers](https://projectf.io/posts/framebuffers/)
- [MIG documentation](https://docs.amd.com/v/u/1.4-English/ug586_7Series_MIS)

