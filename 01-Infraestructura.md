# 🖥️ 01 - Infraestructura

## Servidor: SRV-DC01

| Característica | Valor |
|----------------|-------|
| Sistema | Windows Server 2022 Standard (GUI) |
| RAM | 4 GB |
| Disco | 50 GB |
| Hostname | SRV-DC01 |
| IP Red Interna | 192.168.100.10 |
| IP NAT | 10.0.2.100 |
| DNS | 127.0.0.1 |

## Cliente: Win11-Lab

| Característica | Valor |
|----------------|-------|
| Sistema | Windows 11 Pro |
| RAM | 4 GB |
| Disco | 50 GB |
| IP Red Interna | 192.168.100.20 |
| DNS | 192.168.100.10 |

## Red

- Red interna: `LAN-NOVATECH` en VirtualBox
- SRV-DC01 tiene un segundo adaptador NAT para acceso a Internet
- Win11-Lab solo tiene acceso a la red interna

## Virtualización

- Plataforma: VirtualBox
- Snapshots tomados en cada fase completada

