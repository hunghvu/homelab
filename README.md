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

Good resources:

- [OpenWRT guide for a dumb access point](https://openwrt.org/docs/guide-user/network/wifi/dumbap).
- [SQM guide](https://openwrt.org/docs/guide-user/network/traffic-shaping/sqm).
- [Root partition and filesystem resize](https://openwrt.org/docs/guide-user/advanced/expand_root).
- [OpenWRT firmware selector](https://firmware-selector.openwrt.org/).

  - Include the following packages for a portable USB image. For on-disk image, remove either amd or intel microcode based on the system hardware.

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
    - **parted**: To resize root partition and file system
    - **losetup**: To resize root partition and file system
    - **resize2fs**: To resize root partition and file system

  - Remove the following packages:

    - **kmod-amazon-ena**
    - **kmod-amd-xgbe**
    - **kmod-bnx2**
    - **kmod-forcedeth**

 - For convenience, below is the list of packages for the firmmware selector (Generic x86/64)

```
base-files busybox ca-bundle dnsmasq dropbear e2fsprogs firewall4 fstools grub2-bios-setup kmod-button-hotplug kmod-e1000 kmod-e1000e kmod-fs-vfat kmod-igb kmod-igc kmod-ixgbe kmod-nft-offload kmod-r8169 kmod-tg3 libc libgcc libustream-mbedtls logd luci mkf2fs mtd netifd nftables odhcp6c odhcpd-ipv6only opkg partx-utils ppp ppp-mod-pppoe procd procd-seccomp procd-ujail uci uclient-fetch urandom-seed urngd 6rd luci-proto-ipv6 kmod-usb-net-rtl8152 kmod-usb3 intel-microcode amd64-microcode luci-app-sqm kmod-i40e ethtool pciutils ip parted losetup resize2fs
```