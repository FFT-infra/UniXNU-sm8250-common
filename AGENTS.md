# SoC repository

This repository owns SM8250 resources shared by every board on the SoC: CPU
topology, GICv3 ranges, timer rate, controller register blocks, config fragment
and XNU patches.

Board wiring, memory map, partition geometry and component selection belong in
the device repository.

Read the UniXNU [device tree](https://unixnu.com/porting/device-tree) and
[component catalog](https://unixnu.com/reference/components) guides before
changing this repository. Keep `soc.toml` and `dts/sm8250.dtsi` in agreement.
