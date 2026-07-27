# ⚡ 06 - PowerShell

## Función: New-OnboardUser

Crea un usuario en AD, lo mete en su grupo Global, crea su carpeta personal y asigna permisos NTFS.

```powershell
function New-OnboardUser {
    param ($Nombre, $SamAccountName, $Departamento)
    try {
        New-ADUser -Name $Nombre -SamAccountName $SamAccountName `
                   -Path "OU=$Departamento,OU=OU_Usuarios,DC=novatech,DC=lab" `
                   -AccountPassword (ConvertTo-SecureString "Hola1234" -AsPlainText -Force) `
                   -Enabled $true
        Add-ADGroupMember -Identity "GG_$Departamento" -Members $SamAccountName
        $ruta = "C:\Empresa\Usuarios\$SamAccountName"
        New-Item -Path $ruta -ItemType Directory -Force
        icacls $ruta /inheritance:r
        icacls $ruta /grant "NOVATECH\$SamAccountName`:(M)"
        icacls $ruta /grant "Administradores:(F)"
        Write-Host "✅ $Nombre creado en $Departamento" -ForegroundColor Green
    } catch {
        Write-Host "❌ Error: $_" -ForegroundColor Red
    }
}
---------------------------------------------------------------------------------------------------------
Función: Remove-OffboardUser
Deshabilita la cuenta del empleado y la mueve a la OU Bajas.

```powershell
function Remove-OffboardUser {
    param ($SamAccountName)
    try {
        Disable-ADAccount -Identity $SamAccountName
        $usuario = Get-ADUser -Identity $SamAccountName
        Move-ADObject -Identity $usuario.DistinguishedName `
                      -TargetPath "OU=Bajas,OU=OU_Usuarios,DC=novatech,DC=lab"
        Write-Host "✅ $SamAccountName deshabilitado y movido a Bajas" -ForegroundColor Yellow
    } catch {
        Write-Host "❌ Error con $SamAccountName" -ForegroundColor Red
    }
}
---------------------------------------------------------------------------------------------------------
Función: Get-UserReport
Exporta todos los usuarios de AD a un archivo CSV con la fecha del día.

powershell
function Get-UserReport {
    if (!(Test-Path "C:\Informes")) {
        New-Item -Path "C:\Informes" -ItemType Directory -Force
    }
    $fecha = Get-Date -Format "yyyyMMdd"
    $ruta = "C:\Informes\usuarios_$fecha.csv"
    Get-ADUser -Filter * -Properties Created | 
        Select-Object Name, SamAccountName, Enabled, Created |
        Export-Csv $ruta -NoTypeInformation
    Write-Host "✅ Informe guardado: $ruta" -ForegroundColor Green
}
---------------------------------------------------------------------------------------------------------
Menú interactivo
powershell
do {
    Write-Host "`n===== MENU SYSADMIN ====="
    Write-Host "1. Onboarding"
    Write-Host "2. Offboarding"
    Write-Host "3. Informe"
    Write-Host "4. Salir"
    $op = Read-Host "Opcion"
    switch ($op) {
        1 { $n=Read-Host "Nombre"; $s=Read-Host "Sam"; $d=Read-Host "Depto"
            New-OnboardUser -Nombre $n -SamAccountName $s -Departamento $d }
        2 { $s=Read-Host "SamAccountName"; Remove-OffboardUser -SamAccountName $s }
        3 { Get-UserReport }
        4 { Write-Host "Adios" }
    }
} while ($op -ne 4)
