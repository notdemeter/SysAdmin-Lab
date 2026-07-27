# 📦 09 - WSUS y PKI

## WSUS (Windows Server Update Services)

- **Servicio:** Instalado y configurado en SRV-DC01
- **Productos:** Windows Server 2022
- **Clasificaciones:** Actualizaciones críticas, de seguridad, Service Packs
- **Idiomas:** Español, Inglés
- **Sincronización:** Programada diaria
- **Consola:** wsus.msc

## PKI (Active Directory Certificate Services)

- **CA Name:** novatech-CA
- **Tipo:** Enterprise Root CA
- **Servicio:** CertSvc en ejecución
- **Consola:** certsrv.msc

La CA empresarial permite emitir certificados de confianza a todos los equipos del dominio para HTTPS, autenticación y cifrado de comunicaciones internas.
