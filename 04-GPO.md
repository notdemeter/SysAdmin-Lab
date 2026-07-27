# 📋 04 - Group Policy Objects (GPO)

## GPOs configuradas

| GPO | Enlazada a | Configuración |
|-----|------------|---------------|
| GPO_Ventas_UnidadZ | OU=Ventas | Drive Map Z: → \\SRV-DC01\Ventas |
| GPO_Marketing_UnidadZ | OU=Marketing | Drive Map Z: → \\SRV-DC01\Marketing |
| GPO_RRHH_UnidadZ | OU=RRHH | Drive Map Z: → \\SRV-DC01\RRHH |
| GPO_CarpetaPersonal | OU=OU_Usuarios | Drive Map P: → \\SRV-DC01\Usuarios\%UserName% |

## Tipo de configuración

Todas las GPOs de mapeo de unidades usan **User Configuration**, ya que la unidad debe seguir al usuario independientemente del equipo donde inicie sesión.

## Herencia

- Las GPOs se aplican de arriba hacia abajo: Dominio → OU_Usuarios → OU_Departamento
- Las GPOs más cercanas al objeto tienen prioridad

## Comandos de diagnóstico

```powershell
gpupdate /force     # Forzar actualización de GPOs
gpresult /r         # Ver GPOs aplicadas
gpresult /h         # Informe HTML
