Laboratorio de ciberseguridad enfocado en la implementación de un entorno SIEM con **Wazuh v4.9.0**, ingesta de telemetría avanzada de Windows mediante **Sysmon** y respuesta automatizada ante amenazas (*Active Response*).

##  Arquitectura del Entorno

* **Manager / Indexer / Dashboard**: Wazuh 4.9.0 desplegado en contenedores Docker sobre **Kali Linux**.
* **Agente Monitoreado**: **Windows Server 2022** con **Sysmon** (Event ID 1 habilitado).
* **Red**: Red Interna aislada en VirtualBox (`10.0.2.0/24`).

##  Implementación y Configuración

### 1. Conectividad y Telemetría
* Asignación de IP estática en Kali (`10.0.2.15/24`) e interconexión con Windows Server (`10.0.2.6/24`).
* Apertura y binding de puertos en el stack de Docker (`1514` para comunicación con agentes y `443` para el Dashboard).
* Instalación y vinculación del agente de Wazuh en Windows Server leyendo canales de eventos nativos y `Microsoft-Windows-Sysmon/Operational`.

  

### 2. Detección de Amenazas (SIEM)
* Simulación de ataque mediante comandos PowerShell codificados en Base64 (`-EncodedCommand`).
* Captura de eventos mediante Sysmon (Event ID 1 - Process Creation) e ingesta en tiempo real.
* Mapeo de reglas nativas de Wazuh (`rule.id: 92057`) alineadas al marco **MITRE ATT&CK** (*T1059.001 - PowerShell*).

<img width="1172" height="640" alt="878b715d-b15f-46af-af23-f2b98152b7dd" src="https://github.com/user-attachments/assets/67ab670e-e383-42c6-88e3-8bc9c406050e" />

> **Nota técnica sobre la simulación:**
> Al probar la ejecución codificada con `powershell.exe -EncodedCommand`, la cadena enviada en Base64 debe estar codificada estrictamente en formato **UTF-16LE (Unicode)**. Los intentos generados directamente en entornos Linux/Kali bajo formato **UTF-8** producen errores de interpretación sintáctica en el intérprete de PowerShell. Para garantizar la correcta ejecución del payload se utilizó la conversión explícita:
> ```powershell
> $cmd = [Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes('Write-Output "[Hacking Lab Test]"'))
> powershell.exe -EncodedCommand $cmd
> ```


### 3. Respuesta Activa (SOAR)
* Configuración del motor de **Active Response** en el Manager (`ossec.conf`) para ejecutar scripts automáticos de mitigación (`netsh`) ante eventos de severidad alta.
* Activación del módulo de ejecución remota (`wazuh-execd`) en la máquina víctima.
* Validación del ciclo completo: **Detección -> Disparo de Regla -> Ejecución Automática en Agente**.
 
<img width="967" height="572" alt="Captura de pantalla 2026-10-12 124325" src="https://github.com/user-attachments/assets/5a5c6433-6ce2-4a24-9ade-121c50be7581" />


## Verificación

El flujo fue validado confirmando el registro de ejecución en el agente:
`C:\Program Files (x86)\ossec-agent\active-response\active-responses.log`
