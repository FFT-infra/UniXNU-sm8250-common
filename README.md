# UniXNU-sm8250-common

Shared SM8250 platform support for UniXNU: SoC resources, DTSI, config fragment and patches.

This repository is selected by a device through `board.soc_source = "sm8250"` and a
`kind = "soc"` source in its `feeds.toml`.

| Path | Contents |
| --- | --- |
| `soc.toml` | SoC name, description and component selection |
| `dts/sm8250.dtsi` | CPU topology, GICv3, timer, UFS and peripheral register ranges |
| `configs/sm8250.config` | Kernel config fragment applied before the device defconfig |
| `patches/xnu/series` | SoC XNU patches in application order |
| `kits/` | Shared platform operations |
