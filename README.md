
**Guía de Hardening para Ubuntu Server**

**Basada en los estándares de PCI DSS y Protección de Datos**

**1\. Actualización y Mantenimiento del Sistema**

Mantener el sistema operativo y el software actualizado es fundamental para la seguridad.

**Actualizar el sistema**

```bash
sudo apt update

sudo apt upgrade -y

sudo apt dist-upgrade -y

sudo apt autoremove -y
```

**Configurar actualizaciones automáticas**

Editar el archivo de configuración:

```bash
sudo nano /etc/apt/apt.conf.d/20auto-upgrades
```

Añadir las siguientes líneas:

```plaintext
APT::Periodic::Update-Package-Lists "1";

APT::Periodic::Download-Upgradeable-Packages "1";

APT::Periodic::AutocleanInterval "7";

APT::Periodic::Unattended-Upgrade "1";
```

**2\. Aviso de Entrada (Banner de Aviso)**

PCI DSS requiere que se muestre un aviso antes del inicio de sesión.

Editar el archivo de banner:

```bash
sudo nano /etc/issue.net
```

Añadir el siguiente mensaje:

```plaintext
************************************************************************
*                               ADVERTENCIA                            *
* Este sistema es un sistema privado. El acceso no autorizado está     *
* prohibido y será perseguido por la ley. Toda actividad está          *
* monitoreada y registrada.                                            *
************************************************************************

```

Editar el archivo de configuración SSH para mostrar el banner:

```bash
sudo nano /etc/ssh/sshd_config
```
Asegurarse de que la línea siguiente está presente y no comentada:

```plaintext
Banner /etc/issue.net
```

Reiniciar el servicio SSH:

```bash
sudo systemctl restart ssh
```

**3\. Deshabilitar Servicios y Puertos No Necesarios**

Identificar servicios en ejecución:

```bash
sudo systemctl list-units --type=service --state=running
```

Deshabilitar servicios innecesarios:

```bash
sudo systemctl disable nombre-del-servicio

sudo systemctl stop nombre-del-servicio
```
Configurar el firewall UFW para bloquear puertos no necesarios:

```bash
sudo ufw default deny incoming

sudo ufw default allow outgoing
```
Permitir solo los puertos necesarios:

```bash
sudo ufw allow ssh

sudo ufw allow http

sudo ufw allow https
```

Habilitar el firewall:

```bash
sudo ufw enable
```

**4\. Configuración de SSH Segura**

Editar el archivo de configuración SSH:

```bash
sudo nano /etc/ssh/sshd_config
```

Realizar las siguientes modificaciones:

```plaintext
PermitRootLogin no

PasswordAuthentication no

AllowUsers usuario1 usuario2
```

Reiniciar el servicio SSH:

```bash
sudo systemctl restart ssh
```

**5\. Auditoría y Monitoreo**

Instalar y configurar auditd para la auditoría del sistema.

**Instalar auditd**

```bash
sudo apt install auditd

sudo systemctl enable auditd

sudo systemctl start auditd
```

**Configurar reglas de auditoría**

Editar el archivo de reglas:

```bash
sudo nano /etc/audit/rules.d/audit.rules
```
Añadir las siguientes reglas:

```plaintext
\-w /etc/passwd -p wa -k passwd_changes

\-w /etc/group -p wa -k group_changes

\-w /etc/shadow -p wa -k shadow_changes

\-w /var/log/ -p wa -k log_changes
```

Reiniciar auditd:

```bash
sudo systemctl restart auditd
```
**6\. Fortalecimiento de Contraseñas y Cuentas de Usuario**

Modificar la política de contraseñas:

```bash
sudo nano /etc/security/pwquality.conf
```
Ajustar las siguientes configuraciones:

```plaintext
minlen = 12

dcredit = -1

ucredit = -1

ocredit = -1

lcredit = -1
```
Asegurar cuentas de usuario:

```bash
sudo nano /etc/login.defs
``` 
Modificar las siguientes líneas:

```plaintex
PASS_MAX_DAYS 90

PASS_MIN_DAYS 10

PASS_WARN_AGE 7
```
**7\. Habilitar la Autenticación de Dos Factores (2FA)**

Instalar Google Authenticator:

```bash
sudo apt install libpam-google-authenticator
```
Configurar Google Authenticator para el usuario:

```bash
google-authenticator
```
Configurar PAM para utilizar Google Authenticator:

```bash
sudo nano /etc/pam.d/sshd
```
Añadir la siguiente línea al principio:

```plaintext
auth required pam_google_authenticator.so
```

Editar el archivo de configuración SSH:

```bash
sudo nano /etc/ssh/sshd_config
```
Asegurarse de que la siguiente línea está presente y no comentada:

```plaintext
ChallengeResponseAuthentication yes
```
Reiniciar el servicio SSH:

```bash
sudo systemctl restart ssh
```
**8\. Configuración del Registro de Eventos**

Configurar rsyslog para el registro de eventos.

Editar el archivo de configuración de rsyslog:

```bash
sudo nano /etc/rsyslog.conf
```

Asegurarse de que las siguientes líneas no están comentadas:

```plaintex
module(load="imuxsock") # provides support for local system logging

module(load="imklog") # provides kernel logging support
```
Reiniciar el servicio rsyslog:

```bash
sudo systemctl restart rsyslog
```
**9\. Backups y Recuperación**

Configurar un sistema de backups regular y seguro.

Instalar y configurar una herramienta de backup como rsync:

```bash
sudo apt install rsync
```
Configurar un script de backup diario:

```bash
sudo nano /usr/local/bin/backup.sh
```
Añadir el siguiente contenido:

```bash
# !/bin/bash

rsync -a --delete /ruta/del/directorio/origen /ruta/del/directorio/destino
```
Hacer el script ejecutable:

```bash
sudo chmod +x /usr/local/bin/backup.sh
```
Configurar una tarea cron para ejecutar el script diariamente:

```bash
sudo crontab -e
```
Añadir la siguiente línea:

```plaintext
0 2 \* \* \* /usr/local/bin/backup.sh
```
**Conclusión**

Esta guía proporciona una base sólida para el hardening de un servidor Ubuntu cumpliendo con los estándares de PCI DSS y protección de datos. Es importante revisar y ajustar estas configuraciones según las necesidades específicas y mantener un monitoreo continuo para garantizar la seguridad del sistema.
