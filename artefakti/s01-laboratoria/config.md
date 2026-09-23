# Конфигурация — Сесия 1

Този файл описва **реално изпълнената** базова VirtualBox конфигурация. Тя е начална среда за бъдещата инфраструктура на GameStudio Neo OOD.

## Виртуални машини
| VM | Hostname | vCPU | RAM | ОС | Потребител |
|---|---|---:|---:|---|---|
| `gw` | `gw` | 1 | 1 GB | Debian | `vboxuser` (sudo) |
| `srv` | `srv` | 2 | 4 GB | Debian | `vboxuser` (sudo) |

## VirtualBox мрежови адаптери

### `gw`
| Адаптер | VirtualBox режим | Име на мрежата | Linux интерфейс | IPv4 адрес |
|---|---|---|---|---|
| Adapter 1 | NAT | — | `enp0s3` | DHCP, `10.0.2.15/24` |
| Adapter 2 | Internal Network | `LAN1` | `enp0s8` | `192.168.10.1/24` |
| Adapter 3 | Internal Network | `LAN2` | `enp0s9` | `192.168.20.1/24` |

### `srv`
| Адаптер | VirtualBox режим | Име на мрежата | Linux интерфейс | IPv4 адрес |
|---|---|---|---|---|
| Adapter 1 | Internal Network | `LAN1` | `enp0s3` | `192.168.10.2/24` |

## NetworkManager профили

### `gw`
```bash
sudo nmcli connection add type ethernet ifname enp0s3 con-name gw-wan ipv4.method auto ipv6.method ignore
sudo nmcli connection add type ethernet ifname enp0s8 con-name gw-lan1 ipv4.method manual ipv4.addresses 192.168.10.1/24 ipv6.method ignore
sudo nmcli connection add type ethernet ifname enp0s9 con-name gw-lan2 ipv4.method manual ipv4.addresses 192.168.20.1/24 ipv6.method ignore
sudo nmcli connection up gw-wan
sudo nmcli connection up gw-lan1
sudo nmcli connection up gw-lan2
```

### `srv`
```bash
sudo nmcli connection add type ethernet ifname enp0s3 con-name srv-lan1 ipv4.method manual ipv4.addresses 192.168.10.2/24 ipv4.gateway 192.168.10.1 ipv4.dns "1.1.1.1,8.8.8.8" ipv6.method ignore
sudo nmcli connection up srv-lan1
```

## IPv4 forwarding на `gw`
```bash
echo 'net.ipv4.ip_forward = 1' | sudo tee /etc/sysctl.d/99-ip-forward.conf
cat /proc/sys/net/ipv4/ip_forward
```

Очакван резултат:

```text
1
```

## Snapshot
След успешните проверки е създаден VirtualBox snapshot `clean-state` и за двете виртуални машини.
