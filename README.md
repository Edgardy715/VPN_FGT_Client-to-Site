# 🛡️ Client-to-Site VPN (Remote Access) en FortiGate (FortiOS)

## 👤 Perfil del Autor
- **Ingeniero/Desarrollador:** Edgardy Olivero Flores
- **Especialidad:** Seguridad Informática y Redes

## 🗺️ Arquitectura de Red
![Diagrama de Topología](Topologia%20CTS%20Fortigate.png)

## ⚙️ Resumen Técnico
Este repositorio documenta el diseño y despliegue de una solución de **Acceso Remoto (Client-to-Site VPN)** utilizando un firewall perimetral **Fortinet FortiGate** (FortiOS 7.0.9) como concentrador VPN, integrado con un router **Cisco (R-Core)** para la segmentación de la red empresarial interna.

La topología simula un entorno corporativo donde usuarios móviles (teletrabajo) se conectan desde redes externas hacia la infraestructura interna de manera segura utilizando el agente de **FortiClient**. La solución abarca la creación de políticas de firewall, gestión de identidades locales, asignación dinámica de direccionamiento y sincronización de parámetros criptográficos entre el cliente y el NGFW.

### 🐛 Análisis de Entorno y Requisito NAT para FortiClient
Un hallazgo arquitectónico documentado en este laboratorio es el comportamiento de inicialización del agente FortiClient. En entornos emulados o de laboratorio aislado, si la máquina cliente no cuenta con una pasarela de internet funcional (resolución o módulo NAT activo), la aplicación cliente retiene la solicitud de negociación y el proceso de *handshake* IKE nunca inicia. Para resolver esto, se desplegó un módulo NAT en la topología de prueba, garantizando que el cliente de Windows tuviera salida a internet simulada, permitiendo así el establecimiento exitoso del túnel IPsec.

### 🔐 Diseño Criptográfico y Gestión de Accesos
* **Infraestructura Híbrida:** Configuración base del FortiGate (WAN/LAN) mediante CLI e integración de enrutamiento estático hacia el R-Core Cisco interno (`10.0.0.1`).
* **Gestión de Identidades (IAM):** Creación de perfiles de usuario local (`user definition`) y asignación a un grupo de seguridad remoto (`user group`) dentro de FortiOS para centralizar la autenticación.
* **Dial-up IPsec:** Orquestación mediante IPsec Wizard (Remote Access).
  * *Asignación IP:* Pool virtual dedicado para los clientes remotos (`172.16.2.10 - 172.16.2.50`).
  * *Criptografía:* Ajuste estricto (matching) entre la plantilla del FortiGate y los parámetros avanzados del FortiClient (Pre-Shared Key, Diffie-Hellman, encriptación AES y hash SHA).

## 🛠️ Archivos en este Repositorio
* `FGT-BRDR.conf`: Configuración base (CLI) del firewall perimetral FortiGate (Interfaces y Ruteo).
* `R-Core.ios`: Configuración del router Cisco interno, gestionando la LAN corporativa y el ruteo hacia el firewall.

## 🔍 Auditoría y Diagnóstico
Herramientas utilizadas en la terminal de FortiOS para auditar túneles dinámicos de acceso remoto:

```bash
# Listar los usuarios conectados actualmente por VPN de acceso remoto
get vpn ipsec tunnel details

# Ver el resumen de túneles dial-up activos
get vpn ipsec tunnel summary

# Diagnóstico avanzado de las fases de negociación para clientes entrantes
diagnose vpn tunnel list
diagnose debug application ike -1
diagnose debug enable
