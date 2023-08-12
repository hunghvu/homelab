# Configuration for homelab

## OpenWRT
These are the main focus for now. It's unlikely other files will be manually modified.

- adblock
- adblock-opkg
- attendedsysupgrade
- dhcp
- firewall
- https-dns-proxy
- https-dns-proxy-opkg
- network
- sqm
- sqm-opkg
- system

`network`, `dhcp`, and `firewall` are frequently modified. In `network`, PPPoE username and password must be filled in. **The full backup is stored locally, so this git version is only used to keep track of changes and for IaaC reference if needed.**

Virtualization reference: https://hungvu.tech/virtualize-openwrt-firewall-in-harvester-hci-cluster