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

  - **6rd**: Dependency for IPv6 config (not included in `luci-proto-ipv6)
  - **luci-proto-ipv6**: GUI for IpV6 config
  - **kmod-usb-net-rtl8152**: For Realtek USB NIC
  - **kmod-usb3**: For USB 3.x devices, such as USB NIC
  - **intel-microcode**: Micro code and security patches for Intel system
  - **amd64-microcode**: Micro code and security patches for AMD system
  - **luci-app-sqm**: GUI and dependencies for SQM
  - **kmod-i40e**: Intel X710-DA2 driver
  - **ethtool**: To check information of a NIC
  - **pciutils**: Mostly for `lspci`
  - **ip**: Mostly for `ip addr`, to check all attached interface, physical and virtual