# Proyecto Final SIS313
#  MediaWiki Universitaria 

![Ubuntu](https://img.shields.io/badge/OS-Ubuntu_Server_24.04-orange?style=flat&logo=ubuntu)
![Nginx](https://img.shields.io/badge/Web_Server-Nginx-green?style=flat&logo=nginx)
![MariaDB](https://img.shields.io/badge/Database-MariaDB-blue?style=flat&logo=mariadb)
![MediaWiki](https://img.shields.io/badge/App-MediaWiki_1.42-yellow?style=flat&logo=mediawiki)
![Redis](https://img.shields.io/badge/Cache-Redis-red?style=flat&logo=redis)
![Prometheus](https://img.shields.io/badge/Monitoring-Prometheus-orange?style=flat&logo=prometheus)

Este repositorio documenta el diseño, despliegue y configuración de una infraestructura distribuida para **MediaWiki**, implementando Alta Disponibilidad (HA), balanceo de carga, almacenamiento compartido y monitoreo en tiempo real.

---

## 📋 Información del Proyecto

| Asignatura | SIS313: Infraestructura, Plataformas Tecnológicas y Redes |
| :--- | :--- |
| **Semestre** | 2/2025 |
| **Docente** | Ing. Marcelo Quispe Ortega |
| **Estado** | ✅ Finalizado / 🚀 En Producción |

### 👥 Equipo de Trabajo

| Miembro | Rol Principal | GitHub |
| :--- | :--- | :--- |
| **[Nombre 1]** | Arquitecto de Soluciones / Redes | [@user1](https://github.com/) |
| **[Nombre 2]** | DBA / Storage Specialist | [@user2](https://github.com/) |
| **[Nombre 3]** | DevOps / Automatización | [@user3](https://github.com/) |
| **[Nombre 4]** | Seguridad / Monitoreo | [@user4](https://github.com/) |

---

## 🎯 Objetivos y Justificación

### Objetivo General
Diseñar y desplegar un clúster de servidores para alojar MediaWiki, garantizando:
* **Alta Disponibilidad (HA):** Tolerancia a fallos en la capa de acceso.
* **Escalabilidad:** Capacidad de añadir nodos de aplicación horizontalmente.
* **Rendimiento:** Optimización mediante caché en memoria (Redis).
* **Seguridad:** Hardening de servidores y firewall perimetral.

### Justificación Técnica
En entornos de producción, una arquitectura monolítica representa un **Punto Único de Fallo (SPoF)**. Este proyecto desacopla los servicios (Base de Datos, Archivos, Aplicación y Proxy) para asegurar la continuidad del negocio (**T1**) y mejorar la respuesta ante alta concurrencia (**T4**).

---

## 🌐 Arquitectura de Red y Topología

La infraestructura consta de **9 Máquinas Virtuales** interconectadas en una red LAN estática (`192.168.0.0/24`).

### Mapa de IPs

| Hostname | Rol | IP Física | IP Virtual (VIP) | Descripción |
| :--- | :--- | :--- | :--- | :--- |
| `wiki.usfx.bo` | **VIP Entry Point** | - | **192.168.0.10** | Punto de entrada flotante (Keepalived). |
| `ha1-proxy` | Proxy Master | `192.168.0.11` | - | Nginx Load Balancer + VRRP Master. |
| `ha2-proxy` | Proxy Backup | `192.168.0.12` | - | Nginx Load Balancer + VRRP Backup. |
| `app1-wiki` | App Server 1 | `192.168.0.13` | - | MediaWiki + PHP-FPM. |
| `app2-wiki` | App Server 2 | `192.168.0.14` | - | MediaWiki + PHP-FPM (Clon). |
| `srv-nfs` | Storage | `192.168.0.15` | - | NFS Server (Imágenes compartidas). |
| `srv-redis` | Cache | `192.168.0.16` | - | Redis Object & Session Cache. |
| `srv-db` | Database | `192.168.0.17` | - | MariaDB Server. |
| `srv-monitor` | DNS/Monitor | `192.168.0.20` | - | Prometheus + Grafana + Dnsmasq. |

---

## 🛠️ Guía de Implementación

### 1. Configuración de Red (Netplan)
Todas las VMs utilizan configuración estática. Ejemplo de configuración (`/etc/netplan/50-cloud-init.yaml`):

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: no
      addresses: [192.168.0.XX/24] # Reemplazar XX según tabla
      routes:
        - to: default
          via: 192.168.0.1
      nameservers:
        addresses: [192.168.0.20, 8.8.8.8] # .20 es el DNS interno
