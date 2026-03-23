# Secure Gate - Infraestructura de Red y Seguridad

## Descripción del Proyecto
**Secure Gate** es un laboratorio de ciberseguridad diseñado bajo una arquitectura segmentada para maximizar el control del tráfico y la seguridad de los activos. El entorno está desplegado de forma virtual utilizando VirtualBox. El objetivo central es alojar una API segura en un contenedor Docker, protegida por un firewall perimetral y monitoreada por un SIEM.

---

## Arquitectura de Red
La red se divide en tres segmentos principales administrados por un firewall **pfSense 24.0**:
<img width="729" height="485" alt="image" src="Infraestructura.png" />
### 1. WAN (Interfaz Externa)
* **Interfaz:** `em0`.
* **Configuración:** IP dinámica vía DHCP.
* **Función:** Conexión a internet y recepción de tráfico externo mediante Port Forwarding para dirigir peticiones web a la DMZ.

### 2. DMZ - Zona Desmilitarizada (10.0.0.0/24)
* **Interfaz:** `em1`.
* **Servidor Principal:** `Webserver-Debian` (IP estática: `10.0.0.50`).
* **Sistema Operativo:** Debian 13 (Trixie).
* **Rol:** Aloja el motor Docker donde se despliega la API y el stack de Wazuh.
* **Seguridad:** Aislamiento estricto; el tráfico desde la DMZ hacia la LAN está bloqueado.

### 3. LAN - Red Interna (172.16.0.0/24)
* **Interfaz:** `em2`.
* **Cliente Admin:** `Debian-Client` (IP estática: `172.16.0.10`).
* **Función:** Estación de trabajo para la administración del pfSense vía navegador web.

---

## Implementación de Seguridad y Monitoreo

### SIEM: Wazuh (v4.14.3)
Se ha implementado un stack de Wazuh sobre Docker para la recolección centralizada de eventos.
* **Recolección de Logs:** El firewall pfSense envía logs de sistema, firewall y DNS mediante Syslog (UDP/514) al servidor en la DMZ (`10.0.0.50`).
* **Configuraciones Críticas:**
  * El puerto del Dashboard se cambió de 443 a **8443** para evitar conflictos de servicios.
  * Se habilitó `<logall>` y `<logall_json>` en el `wazuh_manager.conf` para asegurar la recepción de los logs de pfSense.
  * Se configuró el archivo `filebeat.yml` para habilitar el módulo de archivos (`archives: enabled: true`).

### Reglas de Firewall Destacadas
* **Acceso Externo:** Se permite tráfico HTTP (80) y HTTPS (443) desde cualquier origen hacia el servidor web (`10.0.0.50`).
* **Administración SSH:** Permitida exclusivamente desde el cliente LAN (`172.16.0.10`) hacia el servidor web (`10.0.0.50`).
* **Aislamiento:** Denegación explícita de todo tráfico desde WAN que no esté específicamente permitido.

---

## Credenciales del Laboratorio

| Dispositivo | Usuario | Contraseña |
| :--- | :--- | :--- |
| **pfSense WebGUI/Shell** | `admin` | `securegate.fw.123` |
| **Web Server (Debian)** | `gatekeeper` | `gatekeeper.pass.321` |
| **Web Server (Root)** | `root` | `securegate.server.321` |
| **Admin Client (Debian)** | `gateadmin` | `gateadmin.pass.321` |
| **Admin Client (Root)** | `root` | `securegate.client.321` |
| **Wazuh API/Dashboard** | `admin` / `wazuh-wui` | `Wazuh-Secret-2026!` |

---

## Estado del Proyecto y Notas
* **Pendiente:** Actualizar la documentación con las mejoras recientes, Decoders, Reglas, Suricata, etc...