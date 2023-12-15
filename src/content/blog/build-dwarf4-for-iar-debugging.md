---
author: Liya Du
pubDatetime: 2023-07-24T17:42:00Z
title: Build DWARF4 for IAR Workbench Debugging
postSlug: build-dwarf4-for-iar-debugging
featured: false
draft: false
tags:
  - embedded
  - gist
ogImage: "/assets/20230724/iar-error.png"
description: A quick gist about building DWARF 4 using GCC 11 and above so that you can debug the built artifact within IAR Workbench.
---

While VSCode + Arm GCC toolchain + Cortex-Debug + OpenOCD as embedded development environment is getting better everyday. And it has generally became my default choice for developing Arm based MCUs. The IAR Workbench debugging experience is still way ahead of VSCode and its extension combinations. Sometimes I have to build the `.elf` file with Arm GCC toolchain and load it within IAR Workbench.

> [DWARF 5](https://dwarfstd.org/index.html) is the latest standard support by most modern compilers such as GCC.

Now ever since early this year, I have bumped all my Arm GCC toolchain to latest which is version 12.1. Once I launched debugging my imported `.elf` file, I will got this error message says:

```
ELF/DWARF Error: Unsupported debug_info format version: 5
```

<Image src="/assets/20230724/iar-error.png" format="png" alt="IAR cannot recognize DWARF 5" />

Now if you double check the `.elf` file for the DWARF version, you will see it is version 5:

```bash
arm-none-eabi-readelf --debug-dump=info ./build/Standard/main_blinky.elf | grep -A 2 'Compilation Unit @'
```

<Image src="/assets/20230724/dwarf5.png" format="png" alt="Dump info for DWARF 5" />

The problem and solution is straightforward: [since GCC 11, it by default will built DWARF 5](https://developer.arm.com/documentation/ka004927/latest/). To force it to build DWARF 4, you need to add **`-gdwarf-4`** to your `CFLAGS` and `CXXFLAGS`.

Now if you check the built file again:

<Image src="/assets/20230724/dwarf4.png" format="png" alt="Dump info for DWARF 4" />
