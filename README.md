# Configuration for homelab

## OpenWRT

There are 2 packages:

- `x86` is for an edge firewall.
- `arm` is for a dumb access point.

Only the following files are in the repository:

- dhcp
- firewall
- firewall-opkg
- network
- wireless

These are frequently modified files, so they are kept track for IaaC reference purpose. Sensitive information such as username, ssid, password, key are ommited.

**Note: The full backup is stored locally, so the repository is more for change management.**

Good resource:

- [OpenWRT guide for a dumb access point.](https://openwrt.org/docs/guide-user/network/wifi/dumbap)
- [SQM guide](https://openwrt.org/docs/guide-user/network/traffic-shaping/sqm).
- [OpenWRT firmware selector](https://firmware-selector.openwrt.org/), include the following packages for a portable USB image.

  - 6rd
  - luci-proto-ipv6
  - kmod-usb-net-rtl8152
  - kmod-usb3
  - intel-microcode
  - amd64-microcode
  - luci-app-sqm
  - kmod-i40e
  - ethtool
  - pciutils