# 🛡️ Laboratorio de Infraestructura de Redes y Servidores (FortiGate, DMZ y VLANs)

<p align="center">
  <b>Demostración práctica de segmentación de red, control de tráfico, políticas de seguridad y despliegue de servicios en entorno empresarial.</b>
</p>

---

## 📹 Video Demostrativo
*(Coloca aquí el enlace o inserción de tu video demostrativo al principio del repositorio, tal como lo solicitó el profesor)*
- **Enlace del Video:** https://youtu.be/L5cMvKlyf-w

---

## 🎯 Propósito del Laboratorio
El propósito de este laboratorio es implementar una arquitectura de red segura utilizando un firewall **FortiGate** para segmentar una infraestructura crítica. Esto incluye:
* El aislamiento de servidores sensibles en una zona **DMZ**.
* La separación de usuarios y administración mediante **VLANs**.
* La aplicación de políticas de seguridad estrictas (restringiendo fugas de tráfico hacia la LAN e internet).
* El bloqueo selectivo de tráfico web con notificación visual de violación de políticas.
* El despliegue funcional de servidores comerciales y de inventario conectados a una base de datos centralizada.

---

## 🏗️ Topología de Red y Direccionamiento IP

### 📋 Tabla de Direccionamiento

| Zona / Segmento | Dispositivo / Interfaz | Dirección IP / Máscara | Máscara de Subred | Gateway | Propósito / Servicio |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **DMZ Servidores** | Subred Completa | `10.23.13.0/25` | `255.255.255.128` | — | Segmento aislado de servidores |
| **DMZ Servidores** | Interfaz FortiGate | `10.23.13.1` | `255.255.255.128` | — | Puerta de enlace de la DMZ |
| **DMZ Servidores** | Web Server Comercial | `10.23.13.2` | `255.255.255.128` | `10.23.13.1` | Sistema de Punto de Venta |
| **DMZ Servidores** | Web Server Inventario | `10.23.13.3` | `255.255.255.128` | `10.23.13.1` | Sistema de Inventario |
| **DMZ Servidores** | DB Server | `10.23.13.4` | `255.255.255.128` | `10.23.13.1` | Base de datos unificada |
| **VLAN 10** | Subred Completa | `10.23.13.128/25` | `255.255.255.128` | — | Usuarios sin privilegios (DHCP) |
| **VLAN 10** | Interfaz Gateway | `10.23.13.129` | `255.255.255.128` | — | Acceso restringido (Solo Caja) |
| **VLAN 20** | Subred Completa | `10.23.23.0/25` | `255.255.255.128` | — | Soporte / Admin SSH (DHCP) |
| **VLAN 20** | Interfaz Gateway | `10.23.23.1` | `255.255.255.128` | — | Acceso total de gestión |

---
## Diagrama

<img width="1024" height="642" alt="image" src="https://github.com/user-attachments/assets/2a1b230b-3618-4bd9-b89f-f72b3fb4fa19" />

Topologia

<img width="1097" height="462" alt="image" src="https://github.com/user-attachments/assets/742f4c67-b6ca-4a8b-ad7d-2076b02f0a1b" />

## Reglas y perfiles de seguridad
<img width="1282" height="647" alt="image" src="https://github.com/user-attachments/assets/4239d4fe-3f35-4d1b-86ff-4f5404871c8e" />

## ⚙️ Configuración y Despliegue de Servidores

### 1. Web Server Comercial (FacturaScripts)

<img width="1604" height="731" alt="image" src="https://github.com/user-attachments/assets/7d5b176a-1f1e-4b03-9985-dc3b249e4f44" />

* **SO:** Ubuntu 26.04 LTS
* **IP:** `10.23.13.2/28`
* **Servidor Web:** Apache2

```bash
# Actualización y preparación del entorno
lsb_release -a
php -v
sudo apt update
sudo apt install apache2 -y
sudo systemctl status apache2

# Instalación de dependencias de PHP
sudo apt install php libapache2-mod-php php-cli php-curl php-gd php-mbstring php-mysql php-xml php-zip unzip wget -y
sudo apt install php-bcmath php-curl php-gd php-mbstring php-mysql php-xml php-zip unzip wget -y
php -m | grep -E 'bcmath|curl|fileinfo|gd|mbstring|openssl|SimpleXML|zip'
sudo a2enmod rewrite
sudo systemctl restart apache2

# Descarga y despliegue de FacturaScripts
cd /var/www/html
sudo wget [https://facturascripts.com/DownloadBuild/1/stable](https://facturascripts.com/DownloadBuild/1/stable) -O facturascripts.zip
sudo unzip facturascripts.zip
sudo rm facturascripts.zip
sudo mv facturascripts facturas
sudo chown -R www-data:www-data /var/www/html/facturas
sudo find /var/www/html/facturas -type d -exec chmod 755 {} \;
sudo find /var/www/html/facturas -type f -exec chmod 644 {} \;
sudo systemctl restart apache2

2. DB Server (MariaDB)
SO: Ubuntu 26.04 LTS

IP: 10.23.13.4/28

Bash
# Instalación del motor de base de datos
sudo apt update
sudo apt install mariadb-server mariadb-client -y
sudo systemctl status mariadb
sudo mariadb
SQL
-- Creación de base de datos y usuario autorizado
CREATE DATABASE facturascripts CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'facturascripts'@'10.23.13.2' IDENTIFIED BY 'FacturaFS2026!';
GRANT ALL PRIVILEGES ON facturascripts.* TO 'facturascripts'@'10.23.13.2';
FLUSH PRIVILEGES;
SHOW DATABASES;
EXIT;
3. Web Server Inventario (InvenTree)
<img width="1604" height="807" alt="image" src="https://github.com/user-attachments/assets/7a2ebae9-895d-4485-8e26-b87810320c7e" />

Servidor: SERVER-INVENTARIO (10.0.0.42)

Stack: Python 3.14.4, Django 5.2.16, Nginx

Bash
# Instalación automatizada mediante script oficial
wget -qO install.sh [https://get.inventree.org](https://get.inventree.org)
bash install.sh

# Validación de versiones del entorno
/opt/inventree/env/bin/python --version
/opt/inventree/env/bin/python -c "import django; print(django.get_version())"
Panel Administrativo de InvenTree:

🔒 Políticas de Seguridad en FortiGate (GUI)
Bloqueo de Acceso de la VLAN 10 al Servidor de Inventario
Para cumplir con la restricción de que los usuarios de la VLAN 10 no accedan al sistema de inventario (10.23.13.3), mostrando un aviso claro de violación de política, se realizaron los siguientes pasos en la GUI de FortiGate:

Creación de la Regla de Bloqueo: Se configuró una política de tráfico saliente desde la VLAN 10 hacia la DMZ (port2) denegando explícitamente el acceso hacia el servidor de inventario por protocolos HTTP/HTTPS.

Perfil de Web Filter: Se implementó un filtro de URL estático que intercepta de forma estricta la ruta protegida (10.23.13.3/*).

Demostración de Restricción al Usuario: Al intentar ingresar desde un endpoint de la VLAN 10, el sistema muestra la página de bloqueo impidiendo el acceso y reportando la violación de política local.

🖧 Archivos de Configuración (Running-Configs)
📄 Running-Config: SW-USUARIO
Cisco CLI
Building configuration...
Current configuration : 3021 bytes
!
version 15.2
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
service compress-config
!
hostname Switch
!
spanning-tree mode pvst
spanning-tree extend system-id
!
interface GigabitEthernet0/0
 switchport trunk allowed vlan 10,20
 switchport trunk encapsulation dot1q
 switchport mode trunk
 negotiation auto
!
interface GigabitEthernet0/1
 switchport access vlan 20
 switchport mode access
 negotiation auto
!
interface GigabitEthernet0/2
 switchport access vlan 10
 switchport mode access
 negotiation auto
!
ip http server
ip http secure-server
!
ip ssh server algorithm encryption aes128-ctr aes192-ctr aes256-ctr
ip ssh client algorithm encryption aes128-ctr aes192-ctr aes256-ctr
!
end
📄 Running-Config: SW-DMZ
Cisco CLI
Building configuration...
Current configuration : 3053 bytes
!
version 15.2
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
service compress-config
!
hostname Switch
!
spanning-tree mode pvst
spanning-tree extend system-id
!
interface GigabitEthernet0/0
 description ENLACE-FGT-DMZ
 switchport access vlan 30
 switchport mode access
 negotiation auto
!
interface GigabitEthernet0/1
 switchport access vlan 30
 switchport mode access
 negotiation auto
!
interface GigabitEthernet0/2
 switchport access vlan 30
 switchport mode access
 negotiation auto
!
interface GigabitEthernet0/3
 switchport access vlan 30
 switchport mode access
 negotiation auto
!
ip http server
ip http secure-server
!
ip ssh server algorithm encryption aes128-ctr aes192-ctr aes256-ctr
ip ssh client algorithm encryption aes128-ctr aes192-ctr aes256-ctr
!
end
