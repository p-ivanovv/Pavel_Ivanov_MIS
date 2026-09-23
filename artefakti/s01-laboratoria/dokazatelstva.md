# Доказателства — Сесия 1

Този файл пази доказателства, че базовата среда за GameStudio Neo OOD е конфигурирана и работи. Към него се добавят и screenshots от терминалите и VirtualBox.

## Проверка на gateway-а `gw`

```text
$ hostname
gw

$ ip -br addr
enp0s3  UP  10.0.2.15/24
enp0s8  UP  192.168.10.1/24
enp0s9  UP  192.168.20.1/24

$ ip route
default via 10.0.2.2 dev enp0s3
10.0.2.0/24 dev enp0s3
192.168.10.0/24 dev enp0s8
192.168.20.0/24 dev enp0s9

$ cat /proc/sys/net/ipv4/ip_forward
1

$ ping -c3 8.8.8.8
3 packets transmitted, 3 received, 0% packet loss
```

## Проверка на сървъра `srv`

```text
$ hostname
srv

$ ip -br addr
enp0s3  UP  192.168.10.2/24

$ ip route
default via 192.168.10.1 dev enp0s3
192.168.10.0/24 dev enp0s3

$ ping -c3 192.168.10.1
3 packets transmitted, 3 received, 0% packet loss
```

## Screenshot доказателства
Качвай снимките в `artefakti/s01-laboratoria/screenshots/` със следните имена:

| Файл | Какво трябва да се вижда |
|---|---|
| `01-gw-adresi-i-routing.png` | `gw`, IP адресите на трите интерфейса и routing table |
| `02-gw-internet.png` | Успешен `ping -c3 8.8.8.8` от `gw` |
| `03-gw-ip-forwarding.png` | `cat /proc/sys/net/ipv4/ip_forward` връща `1` |
| `04-srv-adres-i-routing.png` | `srv`, IP адресът `192.168.10.2/24` и default gateway |
| `05-srv-ping-gw.png` | Успешен `ping -c3 192.168.10.1` от `srv` |
| `06-virtualbox-topologia.png` | VirtualBox Network settings за LAN1/LAN2 и VM-ите |
| `07-snapshots.png` | Snapshot `clean-state` на `gw` и `srv` |

## Резултат
- `gw` и `srv` са с различни hostname-и и IP адреси.
- `srv` има успешен ping до `gw`.
- `gw` има успешен достъп до интернет.
- IPv4 forwarding на `gw` е включен.
- Създадени са clean-state snapshots.
