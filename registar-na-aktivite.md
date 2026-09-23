# Регистър на активите — GameStudio Neo OOD

## Виртуални машини и услуги
| ID | Актив | Локация / сегмент | IP адрес / мрежа | Услуга / портове | Предназначение | Статус |
|---|---|---|---|---|---|---|
| VM-FW-01 | Firewall / Gateway | София — Dev + CI/CD | `10.200.0.1` | Routing, firewall, DNS egress | Сегментация и контрол на трафика | Планиран |
| VM-CICD-01 | Jenkins + Build Server | София — Dev + CI/CD | `10.200.0.10` | Jenkins TCP 8080; SSH deploy TCP 22 | Build и CI/CD pipeline | Планиран |
| WS-DEV-VT | Developer Workstations | Велико Търново — Dev | `10.201.0.0/24` | HTTP GET към Jenkins TCP 8080 | Разработка и преглед на CI/CD | Планиран |
| VM-GAME-01 | Game Test Server | Габрово — QA + Game | `10.202.0.10` | TCP/UDP 7777; SSH TCP 22 | Тестове на билдове и game listener | Планиран |
| QA-CLIENTS | QA Test Clients | Габрово — QA + Game | `10.202.0.0/24` | TCP/UDP 7777 към Game Server | QA тестване с тест клиент | Планиран |
| PROD-DMZ | Production Servers | DMZ | отделен сегмент | според продукционната услуга | Изолирана продукционна среда | Планиран |

## Мрежови сегменти
| ID | Сегмент | Мрежа | Описание | Статус |
|---|---|---|---|---|
| NET-DEV-SOF | София Dev + CI/CD | `10.200.0.0/24` | Разработчици, Firewall/Gateway и Jenkins/Build Server | Планиран |
| NET-DEV-VT | Велико Търново Dev | `10.201.0.0/24` | Developer Workstations | Планиран |
| NET-QA-GAB | Габрово QA + Game | `10.202.0.0/24` | QA clients и Game Test Server | Планиран |
| NET-DMZ | DMZ Production | отделен сегмент | Изолирани продукционни сървъри | Планиран |
| NET-WAN | WAN / Internet | външна мрежа | Интернет и публичен DNS `8.8.8.8` | Използван |

## Начална лабораторна среда — Сесия 1
| ID | Актив | Роля | Конфигурация / адрес | Статус |
|---|---|---|---|---|
| LAB-VM-01 | `gw` | Базов gateway | Debian; 1 vCPU; 1 GB RAM; WAN DHCP/NAT; LAN1 `192.168.10.1/24`; LAN2 `192.168.20.1/24` | Реализиран |
| LAB-VM-02 | `srv` | Базов server | Debian; 2 vCPU; 4 GB RAM; LAN1 `192.168.10.2/24`; gateway `192.168.10.1` | Реализиран |
| LAB-NET-01 | LAN1 | Вътрешен сегмент | `192.168.10.0/24` | Реализиран |
| LAB-NET-02 | LAN2 | Вътрешен сегмент | `192.168.20.0/24` | Реализиран |

## Архивиране
За `gw` и `srv` е създаден VirtualBox snapshot `clean-state` след успешна първоначална конфигурация.
