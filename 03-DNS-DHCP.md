# 🌐 03 - DNS y DHCP

## DNS

- **Servidor DNS:** SRV-DC01 (127.0.0.1)
- **Zona:** novatech.lab (integrada en Active Directory)
- **Tipo:** Búsqueda directa

## DHCP

| Configuración | Valor |
|---------------|-------|
| Ámbito | 192.168.100.50 - 192.168.100.200 |
| Máscara | 255.255.255.0 |
| Gateway | 192.168.100.10 |
| DNS | 192.168.100.10 |

## Proceso DORA

1. **Discover** - El cliente busca servidor DHCP
2. **Offer** - El servidor ofrece una IP
3. **Request** - El cliente solicita esa IP
4. **Acknowledge** - El servidor confirma la asignación

## APIPA

Si el cliente no encuentra el servidor DHCP, Windows autoasigna una IP 169.254.x.x sin conectividad real.
