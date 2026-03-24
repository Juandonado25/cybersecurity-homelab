# Secure Gate — Laboratorio de Ciberseguridad

Laboratorio de infraestructura SOC virtualizado orientado a la practica de segmentacion de redes, defensa perimetral, deteccion de intrusiones y monitoreo de seguridad. El entorno simula una arquitectura empresarial realista protegida por un firewall pfSense, monitorizada por un stack SIEM Wazuh y equipada con una aplicacion web intencionalmente vulnerable para ejercicios de seguridad ofensiva.

---

## Descripcion general

| Propiedad | Valor |
|---|---|
| Hipervisor | VirtualBox |
| Firewall | pfSense 24.0 |
| SIEM | Wazuh v4.14.3 (Docker) |
| IDS | Suricata (integrado en pfSense) |
| SO del servidor | Debian 13 (Trixie) |
| SO del cliente admin | Debian (LAN) |

---

## Arquitectura de red

La infraestructura se divide en tres segmentos de red aislados administrados por pfSense:

<img width="729" height="485" alt="image" src="/images/Infraestructura.png" />

### WAN — Interfaz externa

- **Interfaz:** `em0`
- **IP:** Dinamica (DHCP)
- **Rol:** Interfaz de cara a internet. Recibe trafico externo y reenvía peticiones HTTP/HTTPS al servidor en la DMZ mediante reglas de Port Forwarding.

### DMZ — Zona Desmilitarizada (`10.0.0.0/24`)

- **Interfaz:** `em1`
- **Host:** `Webserver-Debian` — IP estatica `10.0.0.50`
- **SO:** Debian 13 (Trixie)
- **Rol:** Host Docker que ejecuta el stack SIEM Wazuh y la aplicacion web vulnerable. El trafico desde la DMZ hacia la LAN esta bloqueado explicitamente a nivel de firewall.

### LAN — Red interna (`172.16.0.0/24`)

- **Interfaz:** `em2`
- **Host:** `Debian-Client` — IP estatica `172.16.0.10`
- **Rol:** Estacion de trabajo administrativa. Se utiliza para gestionar pfSense mediante la interfaz web y para conectarse por SSH al servidor en la DMZ.

---

## Stack de seguridad y monitoreo

### IDS perimetral — Suricata

Suricata corre directamente dentro de pfSense e inspecciona el trafico que atraviesa la interfaz WAN. Provee deteccion de intrusiones en tiempo real y genera alertas ante patrones de trafico anomalos o maliciosos antes de que alcancen los segmentos internos.

### SIEM — Wazuh v4.14.3

Wazuh esta desplegado como un stack Docker Compose en el servidor de la DMZ. Centraliza la recoleccion de logs, la correlacion de eventos y la generacion de alertas para toda la infraestructura.

**Fuentes de logs:**

- pfSense reenvía logs de sistema, eventos de firewall y consultas DNS mediante Syslog-ng sobre UDP/5140 hacia `10.0.0.50`.

**Cambios de configuracion clave:**

- El puerto del Dashboard de Wazuh fue cambiado de `443` a `8443` para evitar conflictos con la aplicacion web que corre en el mismo host.
- Se habilitaron `<logall>` y `<logall_json>` en `wazuh_manager.conf` para garantizar la ingesta completa de los logs de pfSense.
- El archivo `filebeat.yml` fue actualizado para activar el modulo de archivos (`archives: enabled: true`).

### Resumen de reglas de firewall

| Direccion | Protocolo | Origen | Destino | Accion |
|---|---|---|---|---|
| WAN → DMZ | HTTP (80) | Cualquiera | 10.0.0.50 | Permitir |
| WAN → DMZ | HTTPS (443) | Cualquiera | 10.0.0.50 | Permitir |
| LAN → DMZ | SSH (22) | 172.16.0.10 | 10.0.0.50 | Permitir |
| WAN → * | Cualquiera | Cualquiera | Cualquiera | Denegar |
| DMZ → LAN | Cualquiera | Cualquiera | Cualquiera | Denegar |

---

## Estructura del repositorio

```
cybersecurity-homelab/
├── images/              # Diagramas de arquitectura y capturas de pantalla
├── pfsense-conf/        # Configuracion del firewall pfSense y exportacion de reglas
├── server-setup/        # Aprovisionamiento del servidor DMZ y configuracion Docker
├── wazuh-setup/         # Archivos de configuracion del stack Wazuh
└── web-app-setup/       # Despliegue de la aplicacion web vulnerable
```

---

## Objetivos de aprendizaje

- Disenar e implementar una red segmentada con arquitectura DMZ.
- Configurar y administrar un firewall stateful con conjuntos de reglas granulares.
- Desplegar y operar una solucion SIEM para la centralizacion de logs y correlacion de alertas.
- Integrar un IDS de nivel de red con telemetria SIEM.
- Practicar escenarios de ataque sobre aplicaciones web en un entorno controlado e intencionalmente vulnerable.
- Analizar trafico de ataque a traves de los dashboards del firewall y del SIEM.

---

## Requisitos previos

- VirtualBox (u hipervisor equivalente) instalado en la maquina anfitriona.
- Conocimientos basicos de administracion de sistemas Linux.
- Conocimientos basicos de redes (subnetting, enrutamiento, conceptos de firewall).
- Docker y Docker Compose instalados en el servidor de la DMZ.

---

## Licencia

Este proyecto tiene fines exclusivamente educativos y de investigacion. Todos los componentes estan desplegados en un entorno virtual aislado. El autor no asume ninguna responsabilidad por el uso indebido de las tecnicas o configuraciones documentadas aqui.
