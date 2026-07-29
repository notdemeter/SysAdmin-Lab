# 🎫 Casos de Service Desk Resueltos

## Caso 1: Usuario sin Internet (IP 169.254.x.x - APIPA)

**Síntoma:** Usuario reporta que no tiene Internet. Su compañero de al lado sí tiene.

**Diagnóstico:**
- `ipconfig` → IP 169.254.100.5 (APIPA)
- Capa: DHCP

**Causa:** El equipo no ha podido contactar con el servidor DHCP. Windows se autoasignó una IP APIPA sin conectividad real.

**Solución:**
1. Comprobar cable de red físico
2. `ipconfig /release && ipconfig /renew`
3. Comprobar servicio DHCP en el servidor
4. Verificar ámbito DHCP (¿tiene IPs libres?)

**Resultado:** El servicio DHCP estaba detenido. Se reinició y el equipo obtuvo IP correcta.

---

## Caso 2: Usuario sin Internet (IP correcta pero no navega)

**Síntoma:** Usuario reporta que no puede navegar. Tiene IP 192.168.1.50.

**Diagnóstico:**
- `ipconfig` → IP correcta
- Capa: Red / DNS

**Causa:** DHCP funciona. El problema está en puerta de enlace, conexión externa o DNS.

**Solución:**
1. `ping` a puerta de enlace → si falla, problema de red local
2. `ping 8.8.8.8` → si falla, problema de conexión externa (router/ISP)
3. `ping google.com` → si falla pero 8.8.8.8 funciona, problema de DNS
4. `nslookup` para confirmar diagnóstico DNS

**Resultado:** El servidor DNS no respondía. Se reinició el servicio DNS y se recuperó la navegación.

---

## Caso 3: Unidad Z: no visible (GPO NO aparece en gpresult)

**Síntoma:** Usuario de Ventas no ve la unidad Z:. Ejecuto `gpresult /r` y la GPO NO aparece aplicada.

**Diagnóstico:**
- `gpresult /r` → GPO no listada
- Capa: GPO (aplicación)

**Causa:** La GPO no se está aplicando. Posibles motivos: OU incorrecta, GPO no enlazada, Security Filtering mal configurado, Block Inheritance.

**Solución:**
1. Verificar que el usuario está en la OU correcta
2. Verificar que la GPO está enlazada a esa OU
3. Revisar Security Filtering
4. Comprobar Block Inheritance
5. `gpupdate /force` y volver a comprobar con `gpresult /r`

**Resultado:** El usuario había sido movido accidentalmente a otra OU. Se recolocó en la OU correcta y la GPO se aplicó.

---

## Caso 4: Unidad Z: no visible (GPO SÍ aparece en gpresult)

**Síntoma:** Usuario de Marketing no ve la unidad Z:. `gpresult /r` muestra la GPO aplicada.

**Diagnóstico:**
- `gpresult /r` → GPO aplicada correctamente
- Capa: SMB / NTFS

**Causa:** La GPO funciona. El problema está en el destino: el recurso compartido no es accesible o los permisos son incorrectos.

**Solución:**
1. `ping SRV-DC01` → comprobar conectividad con el servidor
2. `net view \\SRV-DC01` → comprobar que el recurso compartido existe
3. Acceder manualmente a `\\SRV-DC01\Marketing`
4. Si "Acceso denegado" → permisos SMB
5. Si entra pero no ve archivos → permisos NTFS
6. Revisar AGDLP (¿usuario en el grupo correcto?)

**Resultado:** El grupo Domain Local había perdido los permisos NTFS tras una modificación. Se reasignaron con `icacls`.

---

## Caso 5: Acceso denegado a carpeta compartida (Error 5)

**Síntoma:** Usuario intenta abrir `\\SRV-DC01\RRHH` y recibe "Acceso denegado".

**Diagnóstico:**
- Capa: SMB

**Causa:** El usuario no tiene permisos para entrar al recurso compartido.

**Solución:**
1. `Get-SmbShare` en el servidor → verificar que el recurso existe
2. Revisar permisos de compartición SMB
3. Comprobar que el usuario pertenece al grupo Global correcto
4. Verificar que el grupo Global está en el Domain Local correspondiente

**Resultado:** El usuario había sido eliminado accidentalmente del grupo Global. Se volvió a añadir y recuperó el acceso.

---

## Caso 6: Recurso no encontrado (Error 67)

**Síntoma:** Usuario intenta acceder a `\\SRV-DC01\IT` y recibe "No se encuentra el recurso de red" (Error 67).

**Diagnóstico:**
- Capa: SMB

**Causa:** El recurso compartido no existe o el nombre está mal escrito.

**Solución:**
1. `Get-SmbShare` en el servidor → verificar que el recurso existe
2. Si no existe, crearlo con `New-SmbShare`
3. Verificar que el nombre de la carpeta y la compartición coinciden
4. Comprobar conectividad con `ping SRV-DC01`

**Resultado:** La carpeta compartida fue eliminada durante una migración. Se recreó y el usuario pudo acceder.

---

## Caso 7: Usuario no puede iniciar sesión (relación de confianza)

**Síntoma:** Usuario intenta iniciar sesión en el dominio y recibe "No se puede establecer una relación de confianza entre esta estación de trabajo y el dominio principal".

**Diagnóstico:**
- Capa: Active Directory

**Causa:** El equipo se ha desincronizado del dominio. La contraseña del equipo en AD no coincide con la local.

**Solución:**
1. Iniciar sesión con administrador local
2. `Reset-ComputerMachinePassword -Server SRV-DC01.novatech.lab`
3. Si no funciona: salir del dominio y volver a unir
4. Verificar que la hora del equipo coincide con la del DC (máx. 5 min de diferencia)

**Resultado:** Se reseteó la contraseña del equipo con PowerShell y el usuario pudo iniciar sesión.

---

## Caso 8: Usuario no puede crear archivos en carpeta compartida

**Síntoma:** Usuario puede abrir `\\SRV-DC01\Ventas` y ver archivos, pero al intentar guardar o crear algo nuevo recibe "Acceso denegado".

**Diagnóstico:**
- Capa: NTFS

**Causa:** SMB funciona (puede entrar). El problema es que los permisos NTFS son de solo lectura.

**Solución:**
1. `icacls` en la carpeta → ver permisos NTFS actuales
2. Comprobar que el Domain Local tiene permisos (M) Modify
3. Revisar AGDLP completo: Usuario → GG → DL → Permisos NTFS
4. Corregir permisos: `icacls C:\Empresa\Departamentos\Ventas /grant "DL_Ventas_Modificar:(M)"`

**Resultado:** El permiso NTFS estaba configurado como solo lectura. Se cambió a Modify y el usuario pudo crear archivos.

---

## Caso 9: Contraseña caducada - usuario bloqueado

**Síntoma:** Usuario llama porque no puede iniciar sesión. Recibe "La contraseña ha caducado" o "Cuenta bloqueada".

**Diagnóstico:**
- Capa: Active Directory

**Causa:** La contraseña ha superado los 42 días de caducidad o la cuenta se bloqueó por múltiples intentos fallidos.

**Solución:**
1. Verificar estado de la cuenta: `Get-ADUser -Identity "usuario" -Properties LockedOut, PasswordExpired`
2. Si está bloqueada: `Unlock-ADAccount -Identity "usuario"`
3. Si la contraseña ha caducado: `Set-ADAccountPassword -Identity "usuario" -Reset -NewPassword (ConvertTo-SecureString "NuevaTemp123" -AsPlainText -Force)`
4. Forzar cambio de contraseña en el próximo inicio de sesión: `Set-ADUser -Identity "usuario" -ChangePasswordAtLogon $true`

**Resultado:** La cuenta estaba bloqueada por 5 intentos fallidos. Se desbloqueó, se reseteó la contraseña y el usuario pudo acceder.

---

## Caso 10: GPO se aplica pero el fondo de pantalla no cambia

**Síntoma:** La GPO de fondo de pantalla aparece en `gpresult /r` como aplicada, pero el usuario sigue viendo el fondo por defecto de Windows.

**Diagnóstico:**
- Capa: GPO (destino)

**Causa:** La GPO está configurada correctamente pero la imagen no es accesible o el formato no es compatible.

**Solución:**
1. Verificar que la ruta de la imagen en la GPO es correcta
2. Comprobar que la imagen está en una carpeta compartida accesible (no en local)
3. Verificar permisos NTFS sobre la carpeta que contiene la imagen
4. Comprobar que la imagen es .jpg o .bmp (formatos soportados)
5. `gpupdate /force` y reiniciar el equipo

**Resultado:** La imagen estaba en una ruta local del servidor no compartida. Se movió a `\\SRV-DC01\Fondos` con permisos de lectura y se actualizó la GPO. El fondo se aplicó correctamente.
