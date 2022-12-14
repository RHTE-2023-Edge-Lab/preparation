# Access Point

## Model: TL-WR802N v4

Specifications: https://www.tp-link.com/fr/home-networking/wifi-router/tl-wr802n/

## OpenWRT installation

Install dnsmasq.

```sh
sudo dnf install dnsmasq
sudo mkdir -p /var/lib/tftpboot
```

/etc/dnsmasq.conf

```ini
#bind-interfaces
#interface=lo
port=0
enable-tftp
tftp-root=/var/lib/tftpboot
tftp-secure
```

Place the OpenWRT firmware under tftp root.

```sh
systemctl restart dnsmasq
sudo curl -Lo /var/lib/tftpboot/tp_recovery.bin https://downloads.openwrt.org/releases/22.03.2/targets/ramips/mt76x8/openwrt-22.03.2-ramips-mt76x8-tplink_tl-wr802n-v4-squashfs-tftp-recovery.bin
sudo chown -R dnsmasq:dnsmasq /var/lib/tftpboot
sudo restorecon -RF /var/lib/tftpboot
```

Configure static IP address on your ethernet interface.

```sh
sudo nmcli con mod enp0s31f6 ipv4.method manual ipv4.address 192.168.0.66/24
sudo firewall-cmd --add-port 69/udp --zone public
```

Follow dnsmasq logs:

```sh
journalctl -fu dnsmasq.service
```

* Connect PC with the LAN port, press the reset button, power up the router and keep button pressed for around 10 seconds, until device starts downloading the file. Pumpkin on Mac by default will prompt for approval for the file transfer.
* Router will download file from server, write it to flash and reboot. The reboot process takes a couple of minutes. The light on the device will cycle through various patterns as it updates.
* When the light on the device settles on solid it should be fully rebooted.

In the dnsmasq logs, you should see:

```
dnsmasq-tftp[359677]: sent /var/lib/tftpboot/tp_recovery.bin to 192.168.0.2
```

Configure your ethernet interface with dhcp

```sh
sudo nmcli con mod enp0s31f6 ipv4.method auto -ipv4.address 192.168.0.66/24
```

Web interface is at 192.168.1.1, login root, no password.
