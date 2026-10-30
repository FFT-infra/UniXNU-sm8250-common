# UniXNU-sm8250-common

Shared Qualcomm SM8250 / SM8250-AC (kona) support for UniXNU: CPU topology,
GICv3, timer rate, controller register blocks, config fragment and XNU patches.

`soc.toml` selects `ArmCpu` and `ArmDTPlatform`. Device repositories supply
memory layout, peripherals, component selection, firmware, power policy and
packaging.

Hardware descriptions use `arch/arm64/boot/dts/qcom/sm8250.dtsi` at revision
`5b0764a53235d3c77e1ed674e836e1b3fe1a4b1f` of
[torvalds/linux](https://github.com/torvalds/linux). The GIC, UFS, GPU and
QUPv3 ranges in `dts/sm8250.dtsi` come from that file; the timer rate is the
Qualcomm SM8xxx architectural counter.

| Path | Contents |
| --- | --- |
| `soc.toml` | SoC name, description, hardware source and component selection |
| `dts/sm8250.dtsi` | CPU topology, GICv3, timer, UFS, GPU and QUPv3 register ranges |
| `configs/sm8250.config` | Kernel config fragment applied before the device defconfig |
| `patches/xnu/series` | Two SoC XNU patches, in order: `native-smp-secondary.patch` (MPIDR numbering for kona's Affinity1 core ids, which raises `MAX_CPUS` to 8) then `native-smp-type-clusters.patch` (lets the topology parser take the E/P split from `unixnu,cluster-type`) |
| `kits/Sm8250Platform/` | Adreno 650 GMU cold start (`A650Gmu*`) and the CX power, GPU clock and AOP command-DB layer it needs (`A650GmuPower*`), taken from the SM8250 + A650 port at [UniXNU-pipa](https://github.com/xVanTuring/UniXNU-pipa) and host-tested here |

Build through the [UniXNU CLI](https://unixnu.com/building).
Read the [SoC support steps](https://unixnu.com/porting/soc-support),
[device-tree specification](https://unixnu.com/porting/device-tree) and
[component catalog](https://unixnu.com/reference/components).

`kits/Sm8250Platform` keeps the `A650Gmu*` source filenames and exported symbols from that port.
It follows Linux v6.6 `a6xx_gmu.c` for the a650 family, not the A740 sequence used by
`Sm8550Platform`; the two generations have different register layouts and HFI tables and are not
interchangeable. The kit needs the `AdrenoFirmware` component for `AdrenoHfi*` and takes its
register offsets from this SoC's device tree.

| File | Contents |
| --- | --- |
| `A650Gmu.{c,h}` | The GMU cold start: RSC and PDC setup, firmware load, boot registers, HFI start |
| `A650GmuRegions.{c,h}` | The region table `a650_gmu.bin` needs, with its 4-byte declared overlap clipped |
| `A650GmuPower.{c,h}` | The AOP command-DB parser, the CX power-on and the GPU clock bring-up that precede the cold start |
| `A650Gmu*Tests.c` | Host tests, one per unit; each prints a `PASS:` line |

`A650GmuPower` carries the three pieces that pipa keeps inside its 1432-line `GpuProbe.cpp` and
that cannot compile here, because that file includes the A740 driver headers:

- `A650GpuCommandDbParse` is pure parsing over a byte buffer. It returns the `gfx.lvl`, `mx.lvl`,
  `cx.lvl` level lists and the `SH0`, `MC0`, `ACV`, `CN0` BCM addresses that
  `A650GmuBuildStartup` takes, plus `gmxc.lvl` when present. It searches every slave, as pipa's
  `gpb_cmd_db_find` does; it does not assume which slave holds which name, because alioth's AOP
  database has not been read.
- `A650GpuBringup` is modes 1 to 3: the GCC reads, `gcc_gpu_cfg_ahb_clk`'s enable bit, the
  `gpu_cx_gdsc` power-on, and the two SMMU clocks. Mode 1 never writes.
- `A650GmuClocksEnable` is mode 6: the GPLL0 GPU votes, `gpu_cc_gmu_clk_src` at 200 MHz through
  GPLL0 div / 1.5, the wake/sleep fields of `gpu_cc_cx_gmu_clk`, and the five GMU branches.

The `A650GpuIo` callbacks have the same signatures as `A650GmuIo` in `A650Gmu.h`, so one board
driver fills both structs. `A650GpuIo` adds `delay_us`, which may be null.

Every register offset is in `A650GmuPower.h` with its source. Board wiring, the GPU component
selection and the boot arguments stay in the device repository; this kit only provides the
operations.
