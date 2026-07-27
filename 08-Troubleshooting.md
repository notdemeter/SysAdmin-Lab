# 🔧 08 - Troubleshooting

## Metodología

1. **Escuchar** el síntoma exacto
2. **Identificar la capa** (red, DHCP, DNS, AD, GPO, SMB, NTFS)
3. **Diagnosticar** con comandos
4. **Resolver**
5. **Comprobar**
6. **Documentar**

## Regla de oro

> NUNCA tocar nada sin haber diagnosticado primero.

## Caso 1: Usuario sin Internet (IP 169.254.x.x)

**Capa:** DHCP
**Diagnóstico:** APIPA - el cliente no encuentra el servidor DHCP
**Solución:** Comprobar cable físico → `ipconfig /release && ipconfig /renew` → verificar servicio DHCP → revisar ámbito

## Caso 2: Usuario no ve la unidad Z: (GPO NO aparece en gpresult)

**Capa:** GPO
**Diagnóstico:** La GPO no se está aplicando
**Solución:** Verificar OU → enlace de GPO → Security Filtering → Block Inheritance → `gpupdate /force`

## Caso 3: Usuario no ve la unidad Z: (GPO SÍ aparece en gpresult)

**Capa:** SMB / NTFS
**Diagnóstico:** La GPO funciona, problema en el destino
**Solución:** `ping` al servidor → `net view` → permisos SMB → permisos NTFS → AGDLP

## Caso 4: Usuario accede pero no puede crear archivos

**Capa:** NTFS
**Solución:** `icacls` para verificar permisos → comprobar DL → comprobar AGDLP

## Caso 5: Usuario no puede acceder (Acceso denegado)

**Capa:** SMB
**Solución:** `Get-SmbShare` → permisos de compartición → grupo correcto

## Priorización de incidencias

1. Servidor caído (afecta a todos)
2. Red caída (afecta a muchos)
3. Recurso compartido no accesible (departamento)
4. Usuario sin Internet (individual)
5. Unidad de red no visible (individual)
6. Problema estético (sin impacto)
