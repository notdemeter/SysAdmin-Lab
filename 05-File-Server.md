# 📁 05 - File Server

## Estructura de carpetas
C:\Empresa
├── Departamentos
│ ├── Ventas
│ ├── Marketing
│ ├── IT
│ └── RRHH
└── Usuarios
├── ana.ramirez
├── carlos.garcia
├── pablo.lopez
└── admin.it

## Recursos compartidos (SMB)

| Recurso | Ruta | Permiso SMB |
|---------|------|-------------|
| Ventas | C:\Empresa\Departamentos\Ventas | Everyone Full Control |
| Marketing | C:\Empresa\Departamentos\Marketing | Everyone Full Control |
| IT | C:\Empresa\Departamentos\IT | Everyone Full Control |
| RRHH | C:\Empresa\Departamentos\RRHH | Everyone Full Control |
| Usuarios | C:\Empresa\Usuarios | Everyone Full Control |

## Permisos NTFS

| Carpeta | Grupo | Permiso |
|---------|-------|---------|
| Departamentos\Ventas | DL_Ventas_Modificar | Modify (M) |
| Departamentos\Marketing | DL_Marketing_Modificar | Modify (M) |
| Departamentos\IT | DL_IT_Modificar | Modify (M) |
| Departamentos\RRHH | DL_RRHH_Modificar | Modify (M) |
| Usuarios\* | Usuario correspondiente | Modify (M) |
| Usuarios\* | Administradores | Full Control (F) |

## Regla del más restrictivo

Cuando SMB y NTFS tienen permisos diferentes, se aplica el más restrictivo. 
Por eso SMB se deja en Full Control y la seguridad real se gestiona con NTFS.

## Carpetas personales

Cada usuario tiene una carpeta privada a la que solo él y los administradores pueden acceder.
La herencia de permisos se elimina con `icacls /inheritance:r`.
