# Сесия 1: Базова лаборатория

## Кратък запис
Изградена е базова виртуална мрежова лаборатория във VirtualBox с две Debian виртуални машини: gateway `gw` и server `srv`.

## Топология

```text
                    Internet
                       |
                 VirtualBox NAT
                       |
              gw / enp0s3 (DHCP)
                 10.0.2.15/24
                       |
        +--------------+---------------+
        |                              |
LAN1: enp0s8                      LAN2: enp0s9
192.168.10.1/24                   192.168.20.1/24
        |
        |
srv / enp0s3
192.168.10.2/24
Default gateway: 192.168.10.1
```

## Конфигурация

### `gw`
- ОС: Debian
- vCPU: 1
- RAM: 1 GB
- WAN (`enp0s3`): VirtualBox NAT, DHCP
- LAN1 (`enp0s8`): `192.168.10.1/24`
- LAN2 (`enp0s9`): `192.168.20.1/24`
- Default route: през `10.0.2.2` на WAN
- IPv4 forwarding: включен (`/proc/sys/net/ipv4/ip_forward = 1`)

### `srv`
- ОС: Debian
- vCPU: 2
- RAM: 4 GB
- LAN1 (`enp0s3`): `192.168.10.2/24`
- Default gateway: `192.168.10.1`

## Доказателства / проверки
- `gw` и `srv` са с различни hostname-и и IP адреси.
- От `srv`: `ping -c3 192.168.10.1` — успешен.
- От `gw`: `ping -c3 8.8.8.8` — успешен.
- На `gw`: `cat /proc/sys/net/ipv4/ip_forward` — връща `1`.
- За двете VM е направен VirtualBox snapshot `clean-state`.
