# 🛠️ Scripts de Utilidad para Administración de Sistemas

Colección de scripts PowerShell desarrollados para la administración y automatización 
de tareas en entornos Windows Server con Active Directory.

---

## Script 1: Backup de GPOs
Realiza una copia de seguridad de todas las GPOs del dominio en una carpeta con la fecha actual.
Útil antes de hacer cambios importantes en políticas de grupo.

```powershell
# Define la ruta con la fecha de hoy
$fecha = Get-Date -Format "yyyyMMdd"
$ruta = "C:\Backups\GPOs\$fecha"

# Crea la carpeta si no existe
New-Item -Path $ruta -ItemType Directory -Force

# Hace backup de todas las GPOs
Backup-GPO -All -Path $ruta

Write-Host "✅ GPOs backed up to $ruta"

```
## Script 2: Auditoría de usuarios inactivos (90 días)
Detecta usuarios que no han iniciado sesión en los últimos 90 días.
El resultado se exporta a un CSV para su revisión. 

```powershell
# Buscar usuarios con LastLogonDate superior a 90 días
$inactivos = Get-ADUser -Filter * -Properties LastLogonDate | 
    Where-Object {$_.LastLogonDate -lt (Get-Date).AddDays(-90)}

# Exportar a CSV
$inactivos | Select-Object Name, SamAccountName, LastLogonDate | 
    Export-Csv "C:\Informes\inactivos.csv" -NoTypeInformation

Write-Host "✅ $( $inactivos.Count ) usuarios inactivos encontrados"

``` 
## Script 3: Eventos del sistema (últimos 30 días)
Muestra un resumen de los eventos del sistema de los últimos 30 días.
Útil para auditorías de seguridad y diagnóstico de problemas.

```powershell
# Obtener eventos del sistema de los últimos 30 días
$logs = Get-EventLog -LogName System -After (Get-Date).AddDays(-30)

Write-Host "✅ $( $logs.Count ) eventos del sistema en los últimos 30 días"

```

## Script 4: Creación masiva de usuarios desde CSV
Lee un archivo CSV con los datos de nuevos empleados y los crea automáticamente en Active Directory.
El CSV debe tener las columnas: Nombre, SamAccountName, Departamento.

```powershell

# Importar el CSV con los nuevos empleados
$usuarios = Import-Csv "C:\Temp\nuevos.csv"

# Recorrer cada línea y crear el usuario
ForEach ($u in $usuarios) {
    # Crear usuario en la OU de su departamento
    New-ADUser -Name $u.Nombre `
               -SamAccountName $u.SamAccountName `
               -Path "OU=$($u.Departamento),OU=OU_Usuarios,DC=novatech,DC=lab" `
               -AccountPassword (ConvertTo-SecureString "Temp1234" -AsPlainText -Force) `
               -Enabled $true
    
    # Añadir al grupo Global correspondiente
    Add-ADGroupMember -Identity "GG_$($u.Departamento)" -Members $u.SamAccountName
    
    Write-Host "✅ $($u.Nombre) creado en $($u.Departamento)"
}

```

## Script 5: Deshabilitar usuarios inactivos
Busca usuarios que lleven más de 90 días sin iniciar sesión y los deshabilita automáticamente.
Importante: solo deshabilita usuarios que estén actualmente activos.

```powershell

# Buscar usuarios inactivos que aún están habilitados
$inactivos = Get-ADUser -Filter * -Properties LastLogonDate | 
    Where-Object {$_.LastLogonDate -lt (Get-Date).AddDays(-90) -and $_.Enabled -eq $true}

# Deshabilitar cada uno
ForEach ($u in $inactivos) {
    Disable-ADAccount -Identity $u.SamAccountName
    Write-Host "❌ Deshabilitado: $($u.Name)"
}

Write-Host "✅ $( $inactivos.Count ) usuarios deshabilitados"

```

## Script 6: Informe de permisos NTFS
Muestra los permisos NTFS de todas las carpetas departamentales.
Útil para auditorías de seguridad y verificación de AGDLP.

```powershell

# Lista de carpetas departamentales
$carpetas = @("Ventas", "Marketing", "IT", "RRHH")

# Mostrar permisos de cada carpeta
ForEach ($carpeta in $carpetas) {
    Write-Host "`n=== Permisos de $carpeta ===" 
    icacls "C:\Empresa\Departamentos\$carpeta"
}

```

## Script 7: Verificación de conectividad
Comprueba si los servidores principales responden a ping.
Se puede ampliar para verificar puertos específicos (SMB, RDP, etc.).

```powershell

# Lista de servidores a comprobar
$servidores = @("SRV-DC01", "SRV-FS01")

# Hacer ping a cada servidor
ForEach ($srv in $servidores) {
    if (Test-Connection -ComputerName $srv -Count 1 -Quiet) {
        Write-Host "✅ $srv responde a ping"
    } else {
        Write-Host "❌ $srv NO responde a ping"
    }
}

```

## Script 8: Onboarding completo con carpeta personal
Función completa para el proceso de onboarding de un nuevo empleado:
crea el usuario en AD, lo añade a su grupo Global, crea su carpeta personal
y asigna los permisos NTFS correctos.

```powershell

function New-OnboardUser {
    param (
        $Nombre,           # Nombre completo del empleado
        $SamAccountName,   # Nombre de usuario (login)
        $Departamento      # Departamento (Ventas, Marketing, IT, RRHH)
    )
    
    try {
        # 1. Crear usuario en AD
        New-ADUser -Name $Nombre `
                   -SamAccountName $SamAccountName `
                   -Path "OU=$Departamento,OU=OU_Usuarios,DC=novatech,DC=lab" `
                   -AccountPassword (ConvertTo-SecureString "Hola1234" -AsPlainText -Force) `
                   -Enabled $true
        
        # 2. Añadir al grupo Global del departamento
        Add-ADGroupMember -Identity "GG_$Departamento" -Members $SamAccountName
        
        # 3. Crear carpeta personal
        $ruta = "C:\Empresa\Usuarios\$SamAccountName"
        New-Item -Path $ruta -ItemType Directory -Force
        
        # 4. Quitar herencia de permisos
        icacls $ruta /inheritance:r
        
        # 5. Asignar permisos: Modify para el usuario
        icacls $ruta /grant "NOVATECH\$SamAccountName`:(M)"
        
        # 6. Asignar permisos: Full Control para Administradores
        icacls $ruta /grant "Administradores:(F)"
        
        Write-Host "✅ $Nombre creado en $Departamento" -ForegroundColor Green
        
    } catch {
        Write-Host "❌ Error al crear $Nombre : $_" -ForegroundColor Red
    }
}

# Ejemplo de uso:
# New-OnboardUser -Nombre "Ana Gil" -SamAccountName "ana.gil" -Departamento "Ventas"

```

## 
