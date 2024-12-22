# Configuration for homelab

## OpenWRT

### Explanation

The files by themselves are not backups. They are for change management and future reference purposes. Sensitive information such as username, ssid, password, key are redacted.

**Note: The full backup is stored locally.**

### Notes about OpenWRT access points and switches

Disable `dnsmasq`, `firewall`, and `odhcpd` because these services are only necessary in x86 firewall/router. This also disabled the GUI of these services.

To persistently disable after reboot and upgrade.

```
# In the dashboard, look at navbar: System > Startup > Local Startup
# Put your custom commands here that should be executed once
# the system init finished. By default this file does nothing.

for i in dnsmasq firewall odhcpd; do
  if /etc/init.d/"$i" enabled; then
    /etc/init.d/"$i" disable
    /etc/init.d/"$i" stop
  fi
done

exit 0


```

### Knowledge base:

- [OpenWRT guide for a dumb access point](https://openwrt.org/docs/guide-user/network/wifi/wifiextenders/bridgedap).
- [SQM guide](https://openwrt.org/docs/guide-user/network/traffic-shaping/sqm).
- [Root partition and filesystem resize](https://openwrt.org/docs/guide-user/advanced/expand_root) (currently is not usable in 23.05.4, and will brick the system).
- [OpenWRT firmware selector](https://firmware-selector.openwrt.org/).

  - Include the following packages for a portable USB image. For on-disk image, choose either AMD or Intel microcode based on the system hardware.

    - **amd64-microcode**: Microcode and security patches for AMD system
    - **intel-microcode**: Microcode and security patches for Intel system
    - **6rd**: Dependency for IPv6 config (not included in `luci-proto-ipv6)
    - **luci-proto-ipv6**: GUI for IpV6 config
    - **kmod-usb-net-rtl8152**: For Realtek USB NIC
    - **kmod-usb3**: For USB 3.x devices, such as USB NIC
    - **luci-app-sqm**: GUI and dependencies for SQM
    - **kmod-i40e**: Intel X710-DA2 driver
    - **kmod-mlx4-core**: Mellanox ConnectX-3 Pro (MCX314A-BCCT) driver
    - **kmod-igc**: Intel i22x series
    - **ethtool**: To check information of a NIC
    - **pciutils**: Mostly for `lspci`
    - **ip**: Mostly for `ip addr`, to check all attached interface, physical and virtual
    - **parted**: To resize root partition and file system
    - **losetup**: To resize root partition and file system
    - **resize2fs**: To resize root partition and file system
    - **luci-app-https-dns-proxy**: Enable DNS over HTTPS
      - Becareful when restore backup that has `https-dns-proxy` config, to a system (usually fresh installed), that does not have the package. Because an existing backup modifies DNS config in away which requires the existence of `https-dns-proxy`, so without the package, all DNS queries will fail, and hence no Internet.
    - **luci-app-adblock-fast**: Lightweight DNSBL solution that supports both IPv4, IPv6 and works in conjunction with `luci-app-https-dns-proxy`. [The document is available here](https://docs.openwrt.melmac.net/adblock-fast/)
    - **powertop**: To monitor idle state of the system, `powertop --auto-tune` is a fast way to enable all ASPM capabilities.
    - The following packages are optional dependencies to accelerate `adblock-fast`:

      - **gawk**
      - **grep**
      - **sed**
      - **coreutils-sort**

  - Remove the following packages:

    - **kmod-amazon-ena**
    - **kmod-amd-xgbe**
    - **kmod-bnx2**
    - **kmod-forcedeth**
    - **bnx2-firmware**
  
  - The following packages are optional:

    - **zram-swap**: Enable swap for the OS. However, this is mostly for embedded devices (but with trade off like reduce flash lifetime, increasing CPU usage, etc.) For x86, RAM is usually overkill so swap is not necessary
    - **kmod-lib-lz4**: LZ4 compression algorithm for swap
    - **kmod-lib-zstd**: ZSTD compression algorithm for swap

      - Customization of compression algorithm is available at `/etc/config/system`, but it seems future version of OpenWRT will expose the configuration via LuCI (source: https://github.com/openwrt/openwrt/issues/14441#issuecomment-1973885215)


 - For convenience, below is the list of packages for the firmmware selector (Generic x86/64)

```
base-files busybox ca-bundle dnsmasq dropbear e2fsprogs firewall4 fstools grub2-bios-setup kmod-button-hotplug kmod-e1000 kmod-e1000e kmod-fs-vfat kmod-igb kmod-igc kmod-ixgbe kmod-nft-offload kmod-r8169 kmod-tg3 libc libgcc libustream-mbedtls logd luci mkf2fs mtd netifd nftables odhcp6c odhcpd-ipv6only opkg partx-utils ppp ppp-mod-pppoe procd procd-seccomp procd-ujail uci uclient-fetch urandom-seed urngd 6rd luci-proto-ipv6 kmod-usb-net-rtl8152 kmod-usb3 intel-microcode amd64-microcode luci-app-sqm kmod-i40e ethtool pciutils ip parted losetup resize2fs luci-app-https-dns-proxy luci-app-adblock-fast gawk grep sed coreutils-sort zram-swap kmod-lib-lz4 kmod-lib-zstd kmod-igc
```

#### About ASPM

- Enable ASPM via `powertop` works, and it is observable via hardware interaction. For example, RTL8125BG port will get into deep idle state when a port is not connected. This requires a system restart while the port is connected to bring the NIC online. However, the power consumption is change is negligible, and is often masked by fluctuation of system activity.
- Intel 12th gen CPU (e.g., G6900, 12100) can get to C7 (HW), and C10 (OS) idle state, but Pkg (HW) stucks at C2. There were attempts such as switching PCIE devices, slots, disable SATA controller, and other BIOS tweaks, but Pkg (HW) refuses to go lower than C2. Not sure why yet.

![C state information.](/assets//firewall-c-state-information.png)

- It appears Intel I226-V is buggy with ASPM. 

  - With an Intel I226-V NIC, the system boots really slow, and when ASPM is enforced via `powertop`, the system completely freezes because the CPU utilization gradually increase up to 100%. Probably, this is an upstream bug of `igc`. There seems to be many report on this matter. Meanwhile, Realtek RTL 8125BG works perfectly.
  - Currently, there is no available NVM update for Intel I226-V (only Intel I225 series), so not sure if an NVM update can resolve the problem or not.
  - Unrelated note: Normally, `ethX` is based on whichever NIC is electrically closet to the CPU. For example, `eth0` is an integrated NIC, `eth1` is a nic at PCIE slot 1. However, the I226-V nic messes up the order, and making itself the first set of interface, from `eth0` to `eth3`, other NIC order is also changed too.

![Intel I226-V is buggy under ASPM.](/assets/buggy-i226v-aspm.png)

### SFP+ compatibility

#### Aruba 1930 JL682A

- For DAC, the switch accepts pretty much anything.

  - If the DAC is not compatible, SFP+ still operates, but the LED becomes amber color.
  - Aside from officialy listed DAC, the older code from OfficeConnect line such as J928xx(J9281/3/5:B/D) is also compatible. This means the LED is a normal green color.

- For optic modules, outside of officially listed modules, they depend. Some modules work some don't. Not sure about the rule yet.

#### TP-Link TL-SX3008F

- For DAC, the switch accept everything, no failed attempt yet.
- For optic modules, there is no officially listed modules. It appears everything works, but some older optics will not auto negotiate properly. This means the link rate may randomly switches between 1 Gbe, 10 Gbe, or is stuck at one rate (normally 1 Gbe.)

#### Intel X710-DA2

- At default:

  - For both DAC and optic modules, rather picky.
  - The compatibility differs in Windows and Linux. Some DAC and optic work in both OS, some work in one but not the other.
  - It seems generic Cisco-coded DAC are accepted normally. However, some such as J928xx DAC from HP are not usable in Windows (error in Event Viewer log), but is fine in Linux.
  - For optic modules, not sure about the rule yet because all purchases have optic modules included.

- Unlocked:

  - The NIC can be unlock using the following unlocker script: https://github.com/hunghvu/xl710-unlocker.
  - After being unlocked, it seems the NIC accepts all DAC and optic modules, as of NVM v9.52 (8/23/2024).