\# Kali Linux → Windows: SSH Remote Administration Lab



\## 📌 Descripción



Laboratorio práctico de administración remota en el que \*\*Kali Linux actúa como cliente SSH\*\* y \*\*Windows como servidor OpenSSH\*\*, utilizando autenticación mediante claves públicas y privadas.



El objetivo del laboratorio fue implementar una conexión SSH segura, configurar el servicio OpenSSH en Windows, permitir el tráfico SSH mediante Windows Firewall y realizar pruebas de conectividad y diagnóstico desde Kali Linux.



\---



\## 🎯 Objetivos



\* Configurar un servidor OpenSSH en Windows.

\* Generar un par de claves SSH Ed25519 en Kali Linux.

\* Configurar autenticación mediante clave pública.

\* Mantener la clave privada exclusivamente en Kali Linux.

\* Configurar Windows Firewall para permitir TCP/22.

\* Comprobar la conectividad entre ambos sistemas.

\* Diagnosticar problemas de red y del servicio SSH.

\* Establecer una sesión remota desde Kali Linux hacia Windows PowerShell.

\* Practicar procedimientos de troubleshooting habituales en entornos IT.



\---



\## 🖥️ Entorno del laboratorio



| Componente     | Configuración             |

| -------------- | ------------------------- |

| Cliente SSH    | Kali Linux                |

| Servidor SSH   | Windows                   |

| Protocolo      | SSH                       |

| Puerto         | TCP/22                    |

| Autenticación  | Ed25519 public-key        |

| Shell remoto   | PowerShell                |

| Virtualización | VirtualBox                |

| Firewall       | Windows Defender Firewall |



\### Arquitectura



```text

┌──────────────────────┐

│     Kali Linux       │

│                      │

│  SSH Client          │

│  Private Key         │

│  id\_ed25519          │

└──────────┬───────────┘

&#x20;          │

&#x20;          │ SSH / TCP 22

&#x20;          │

&#x20;          ▼

┌──────────────────────┐

│       Windows        │

│                      │

│  OpenSSH Server      │

│  sshd                │

│  authorized\_keys     │

│  Windows Firewall    │

│  PowerShell          │

└──────────────────────┘

```



\---



\## 🔐 1. Generación de claves SSH



El par de claves se generó en Kali Linux utilizando Ed25519:



```bash

ssh-keygen -t ed25519 -C "kali-to-windows"

```



Se generaron dos archivos:



```text

........................…

```



\### Seguridad



La clave privada:



```text

……………………………………..

```



permanece exclusivamente en Kali Linux.



La clave pública:



```text

………………………………...

```



se instala en Windows.



\*\*La clave privada nunca debe copiarse al servidor.\*\*



\---



\## 🪟 2. Configuración de OpenSSH Server en Windows



Se comprobó el estado del servicio:



```powershell

Get-Service sshd

```



Cuando fue necesario, se instaló OpenSSH Server y se inició el servicio:



```powershell

Add-WindowsCapability -Online -Name OpenSSH.Server\~\~\~\~0.0.1.0



Start-Service sshd



Set-Service -Name sshd -StartupType Automatic

```



Finalmente se verificó:



```powershell

Get-Service sshd

```



\---



\## 🔑 3. Configuración de autenticación mediante clave pública



En Windows se creó el directorio de claves:



```powershell

mkdir $HOME\\.ssh -Force

```



Y se configuró:



```text

authorized\_keys

```



La clave pública generada en Kali fue añadida a este archivo.



De esta manera, Windows puede verificar la identidad del cliente SSH sin necesidad de transmitir la clave privada.



\---



\## 🔥 4. Configuración de Windows Firewall



Se comprobó la existencia de reglas relacionadas con SSH:



```powershell

Get-NetFirewallRule -Name \*ssh\*

```



Cuando fue necesario, se creó una regla para permitir conexiones TCP al puerto 22:



```powershell

New-NetFirewallRule `

&#x20;   -Name sshd `

&#x20;   -DisplayName "OpenSSH Server (sshd)" `

&#x20;   -Enabled True `

&#x20;   -Direction Inbound `

&#x20;   -Protocol TCP `

&#x20;   -Action Allow `

&#x20;   -LocalPort 22

```



\---



\## 🌐 5. Verificación del puerto SSH



Se comprobó que Windows estuviera escuchando en TCP/22:



```powershell

Get-NetTCPConnection -LocalPort 22 -State Listen

```



También se utilizó:



```powershell

netstat -ano | findstr :22

```



Un resultado similar a:



```text

TCP    0.0.0.0:22    0.0.0.0:0    LISTENING

```



indica que el servidor SSH está escuchando conexiones.



\---



\## 🔎 6. Pruebas de conectividad desde Kali



Desde Kali se verificó primero la conectividad IP:



```bash

ping <WINDOWS\_IP>

```



Posteriormente se comprobó la disponibilidad del puerto SSH:



```bash

nc -zv <WINDOWS\_IP> 22

```



Estas pruebas permiten diferenciar problemas de conectividad de red de problemas específicos del servicio SSH.



\---



\## 🔗 7. Conexión SSH



Finalmente se estableció la conexión desde Kali:



```bash

ssh <WINDOWS\_USER>@<WINDOWS\_IP>

```



También se puede especificar explícitamente la clave privada:



```bash

ssh -i \~/.ssh/id\_ed25519 <WINDOWS\_USER>@<WINDOWS\_IP>

```



Una vez establecida la conexión se verificó el entorno remoto mediante:



```powershell

hostname

```



y:



```powershell

$PSVersionTable

```



Esto permitió confirmar que la sesión remota estaba ejecutándose sobre Windows PowerShell.



\---



\## 🛠️ Troubleshooting



Durante el laboratorio se realizaron diferentes comprobaciones para diagnosticar problemas de conectividad, servicios y acceso remoto.



\### SSH no disponible en Windows



Se comprobó si el comando y el servicio OpenSSH estaban disponibles:



```powershell

Get-Service sshd

```



La ausencia del servicio indicó que OpenSSH Server necesitaba ser instalado/configurado.



\### Problemas de conectividad



Se realizaron pruebas progresivas:



```text

Kali

&#x20; │

&#x20; ├── ping → ¿Existe conectividad IP?

&#x20; │

&#x20; ├── nc → ¿Está abierto TCP/22?

&#x20; │

&#x20; └── ssh → ¿Funciona la autenticación?

```



Este enfoque permite aislar el problema por capas.



\### Verificación del listener



En Windows:



```powershell

Get-NetTCPConnection -LocalPort 22 -State Listen

```



Si no existe un listener en TCP/22, el problema puede estar relacionado con el servicio `sshd` o su configuración.



\### Firewall



Se comprobó la configuración de Windows Firewall para determinar si el tráfico entrante hacia TCP/22 estaba permitido.



\---



\## 🧠 Conocimientos adquiridos



Este laboratorio permitió practicar:



\* Administración de sistemas Linux.

\* Administración básica de Windows.

\* PowerShell.

\* SSH.

\* OpenSSH Server.

\* Autenticación mediante claves públicas.

\* Criptografía de clave pública.

\* TCP/IP.

\* Puertos y servicios.

\* Windows Firewall.

\* Diagnóstico de conectividad.

\* Troubleshooting.

\* Administración remota.

\* Virtualización mediante VirtualBox.



\---



\## 🚀 Próximas mejoras



Como continuación del laboratorio se podrían implementar:



\* Autenticación SSH con passphrase.

\* Gestión de múltiples usuarios.

\* Restricción del acceso SSH mediante firewall.

\* Configuración avanzada de `sshd\_config`.

\* Deshabilitación de autenticación por contraseña.

\* Registro y análisis de eventos SSH.

\* Automatización mediante PowerShell.

\* Automatización mediante Bash.

\* Integración con Ansible.

\* Monitorización del servicio SSH.



\---



\## 📚 Finalidad



Este proyecto forma parte de un laboratorio personal orientado al desarrollo de competencias prácticas en:



\*\*IT Support · Linux · Windows · System Administration · Networking · Cybersecurity\*\*



> ⚠️ Este repositorio documenta un entorno de laboratorio. No contiene claves privadas, contraseñas, tokens ni credenciales reales.



