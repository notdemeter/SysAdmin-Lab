# 🛡️ 07 - Seguridad

## Windows Defender

- Antivirus activo con protección en tiempo real
- Actualizaciones de firmas automáticas

## Firewall de Windows

| Perfil | Estado |
|--------|--------|
| Domain | Activado |
| Private | Activado |
| Public | Activado |

## Reglas de firewall personalizadas

Se creó una regla para bloquear el puerto 8080 como prueba de administración de seguridad.

## Política de contraseñas

| Configuración | Valor |
|---------------|-------|
| Longitud mínima | 7 caracteres |
| Complejidad | Activada (mayúsculas, minúsculas, números, símbolos) |
| Caducidad | 42 días |
| Bloqueo de cuenta | Configurado en Default Domain Policy |

## Auditoría

- Logs de seguridad revisados desde Event Viewer
- Eventos de inicio de sesión (SuccessAudit y FailureAudit)
- Diagnóstico con `Get-EventLog`

## Hardening básico

- Firewall activo en todos los perfiles
- Contraseñas seguras obligatorias
- Permisos NTFS aplicados con modelo AGDLP
- Carpetas personales con herencia eliminada
