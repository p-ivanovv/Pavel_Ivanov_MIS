# Конфигурация — Сесия 1

## VirtualBox

### `gw`
| Адаптер | Режим | Име на мрежата | Linux интерфейс | Адрес |
|---|---|---|---|---|
| Adapter 1 | NAT | — | `enp0s3` | DHCP (`10.0.2.15/24`) |
| Adapter 2 | Internal Network | `LAN1` | `enp0s8` | `192.168.10.1/24` |
| Adapter 3 | Internal Network | `LAN2` | `enp0s9` | `192.168.20.1/24` |

### `srv`
| Адаптер | Режим | Име на мрежата | Linux интерфейс | Адрес |
|---|---|---|---|---|
| Adapter 1 | Internal Network | `LAN1` | `enp0s3` | `192.168.10.2/24` |

## NetworkManager профили

### `gw`
```bash
nmcli connection add type ethernet ifname enp0s3 con-name gw-wan ipv4.method auto ipv6.method ignore
nmcli connection add type ethernet ifname enp0s8 con-name gw-lan1 ipv4.method manual ipv4.addresses 192.168.10.1/24 ipv6.method ignore
nmcli connection add type ethernet ifname enp0s9 con-name gw-lan2 ipv4.method manual ipv4.addresses 192.168.20.1/24 ipv6.method ignore
```

### `srv`
```bash
nmcli connection add type ethernet ifname enp0s3 con-name srv-lan1 ipv4.method manual ipv4.addresses 192.168.10.2/24 ipv4.gateway 192.168.10.1 ipv4.dns "1.1.1.1,8.8.8.8" ipv6.method ignore
```

## IPv4 forwarding на `gw`
```bash
echo 'net.ipv4.ip_forward = 1' | sudo tee /etc/sysctl.d/99-ip-forward.conf
cat /proc/sys/net/ipv4/ip_forward
```
