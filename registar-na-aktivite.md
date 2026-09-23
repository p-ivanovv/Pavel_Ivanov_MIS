# Регистър на активите

| ID | Актив | Тип | Конфигурация / адрес | Предназначение | Статус |
|---|---|---|---|---|---|
| VM-01 | `gw` | Virtual machine / Gateway | Debian; 1 vCPU; 1 GB RAM; WAN: DHCP/NAT; LAN1: `192.168.10.1/24`; LAN2: `192.168.20.1/24` | Маршрутизация между WAN, LAN1 и LAN2 | Активен |
| VM-02 | `srv` | Virtual machine / Server | Debian; 2 vCPU; 4 GB RAM; LAN1: `192.168.10.2/24`; gateway: `192.168.10.1` | Вътрешен сървър в LAN1 | Активен |
| NET-01 | WAN | VirtualBox NAT | DHCP към `gw` | Достъп на gateway-а до интернет | Активен |
| NET-02 | LAN1 | VirtualBox Internal Network | `192.168.10.0/24` | Вътрешен сегмент за `gw` и `srv` | Активен |
| NET-03 | LAN2 | VirtualBox Internal Network | `192.168.20.0/24` | Втори вътрешен сегмент към `gw` | Активен |

## Архивиране
За `gw` и `srv` е създаден VirtualBox snapshot `clean-state` след успешна първоначална конфигурация.
