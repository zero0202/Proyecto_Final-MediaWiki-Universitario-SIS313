# Proyecto Final SIS313
#  MediaWiki Universitaria 
Despliegue de un Cluster Web Escalable, Resiliente y Monitoreado.
![Ubuntu](https://img.shields.io/badge/OS-Ubuntu_Server_24.04-orange?style=flat&logo=ubuntu)
![Nginx](https://img.shields.io/badge/Web_Server-Nginx-green?style=flat&logo=nginx)
![MariaDB](https://img.shields.io/badge/Database-MariaDB-blue?style=flat&logo=mariadb)
![MediaWiki](https://img.shields.io/badge/App-MediaWiki_1.42-yellow?style=flat&logo=mediawiki)
![Redis](https://img.shields.io/badge/Cache-Redis-red?style=flat&logo=redis)
![Prometheus](https://img.shields.io/badge/Monitoring-Prometheus-orange?style=flat&logo=prometheus)

Asignatura: SIS313: Infraestructura, Plataformas Tecnológicas y Redes
Semestre: 2/2025
Docente: Ing. Marcelo Quispe Ortega

👥 Miembros del Equipo ([Grupo-07])
Integrantes y Rol
[Castro Siñanis Jose Luis]	Arquitecto de Infraestructura y Seguridad	[]
[Villena Mamani Alvaro Fabian]	Ingeniero de Automatización y Monitoreo	[@AlvaroFab28]
[Villca Araca Jhesica]	Administrador de Sistemas y Base de Datos	[@cero0202]

🎯 I. Objetivo del Proyecto
Objetivo: Desplegar una infraestructura escalable, resiliente y de alta disponibilidad para MediaWiki, implementando balanceo de carga, replicación de base de datos, almacenamiento compartido, caché distribuido y monitoreo, siguiendo las mejores prácticas de seguridad.

💡 II. Justificación e Importancia
Justificación: Este proyecto resuelve problemas de continuidad operacional (T1) y seguridad (T5) al eliminar puntos únicos de fallo, distribuir la carga de trabajo, y aplicar hardening en todos los servidores. La implementación de alta disponibilidad (T2) garantiza que el servicio de la wiki esté disponible incluso en caso de fallos de hardware o software. Además, la automatización (T6) y el monitoreo (T4) permiten una gestión eficiente y proactiva de la infraestructura.

🛠️ III. Tecnologías y Conceptos Implementados
3.1. Tecnologías Clave
[Nginx]: Proxy Inverso y Balanceo de Carga con Rate Limiting y SSL/TLS.
[MariaDB]: Servidor de Base de Datos principal.
[Keepalived]: Implementación de VRRP para Failover de la IP Virtual (HA).
[Prometheus/Grafana]: Monitoreo y visualización de métricas de rendimiento/tráfico.
[Redis]: Caché de objetos y sesiones para acelerar el rendimiento.
[NFS]: Almacenamiento compartido para archivos de MediaWiki.
[Dnsmasq]: Servidor DNS local para resolución de nombres en la red interna.
[Ansible/Bash]: Automatización del despliegue y la configuración de hardening (implícito en guías, aunque no se detalló playbook, se usaron scripts bash).

3.2. Conceptos de la Asignatura Puestos en Práctica (T1 - T6)
Alta Disponibilidad (T2) y Tolerancia a Fallos: [Se implementó balanceo de carga con Nginx y failover con Keepalived para los proxies, y almacenamiento compartido NFS para que las aplicaciones puedan ser stateless en cuanto a archivos.]
Seguridad y Hardening (T5): [Se aplicó hardening SSH (cambio de puerto, deshabilitar root), configuración de firewall UFW con reglas específicas, y SSL/TLS con certificados autofirmados y configuraciones seguras en Nginx.]
Automatización y Gestión (T6): [Se utilizaron scripts bash para la configuración de cada VM, y se documentó un proceso replicable. Además, se implementó monitoreo automatizado con Prometheus.]
Balanceo de Carga/Proxy (T3/T4): [Nginx como balanceador de carga entre las dos instancias de MediaWiki, con health checks implícitos.]
Monitoreo (T4/T1): [Prometheus para recolección de métricas y Grafana para visualización, monitoreando todas las VMs.]
Networking Avanzado (T3): [Configuración de IP estáticas, VLAN (implícito en la red 192.168.0.0/24), y enrutamiento estático para la salida a internet.]

🌐 IV. Diseño de la Infraestructura y Topología
4.1. Diseño Esquemático
VM/Host	Rol	IP Física	IP Virtual (si aplica)	Red Lógica	SO
ha1-proxy	Proxy / Load Balancer MASTER	192.168.0.11	192.168.0.10 (VIP)	Red 192.168.0.0/24	Ubuntu 24.04
ha2-proxy	Proxy / Load Balancer BACKUP	192.168.0.12	192.168.0.10 (VIP)	Red 192.168.0.0/24	Ubuntu 24.04
app1-wiki	Servidor de Aplicación 1	192.168.0.13	N/A	Red 192.168.0.0/24	Ubuntu 24.04
app2-wiki	Servidor de Aplicación 2	192.168.0.14	N/A	Red 192.168.0.0/24	Ubuntu 24.04
srv-nfs	Servidor NFS	192.168.0.15	N/A	Red 192.168.0.0/24	Ubuntu 24.04
srv-redis	Servidor Redis	192.168.0.16	N/A	Red 192.168.0.0/24	Ubuntu 24.04
srv-db	Servidor MariaDB	192.168.0.17	N/A	Red 192.168.0.0/24	Ubuntu 24.04
srv-monitor	Servidor Monitor + DNS	192.168.0.20	N/A	Red 192.168.0.0/24	Ubuntu 24.04

4.2. Estrategia Adoptada
Estrategia de Balanceo y Failover: Se optó por un par de proxies con Keepalived en modo MASTER-BACKUP para garantizar la disponibilidad de la IP virtual. Nginx balancea la carga entre las dos aplicaciones de MediaWiki.
Estrategia de Almacenamiento: Se utilizó NFS para compartir los archivos de MediaWiki (imágenes) entre las dos instancias de la aplicación, garantizando consistencia.
Estrategia de Caché: Se implementó Redis para almacenar sesiones y caché de objetos, lo que permite una mayor velocidad y persistencia de sesiones incluso si una aplicación falla.
Estrategia de Monitoreo: Se desplegó Prometheus para recolectar métricas de todas las VMs mediante node-exporter, y Grafana para visualizar los datos en tiempo real.

📋 V. Guía de Implementación y Puesta en Marcha
5.1. Pre-requisitos
- 8 VMs con Ubuntu Server 24.04 instalado.
- Acceso root/sudo a todas las VMs.
- Conexión de red entre las VMs en la misma subred (192.168.0.0/24).
- Router físico con puerta de enlace en 192.168.0.1.

5.2. Despliegue (Pasos generales)
1. Configurar la red en cada VM mediante Netplan con las IPs estáticas según la tabla.
2. En la VM srv-db (192.168.0.17): Instalar MariaDB, configurar acceso remoto y crear la base de datos y usuario para MediaWiki.
3. En la VM srv-nfs (192.168.0.15): Instalar NFS, crear carpeta compartida y exportarla a las IPs de las aplicaciones.
4. En las VMs app1-wiki (192.168.0.13) y app2-wiki (192.168.0.14): Instalar Nginx, PHP, MediaWiki, montar la carpeta NFS y configurar la aplicación para conectarse a la base de datos.
5. En las VMs ha1-proxy (192.168.0.11) y ha2-proxy (192.168.0.12): Instalar Nginx y Keepalived, configurar el balanceo de carga y la IP virtual.
6. En la VM srv-redis (192.168.0.16): Instalar Redis y configurar para aceptar conexiones de las aplicaciones.
7. En la VM srv-monitor (192.168.0.20): Instalar Dnsmasq, Prometheus, Grafana y node-exporter. Configurar DNS para el dominio wiki.usfx.bo.
8. Aplicar hardening: Cambiar puerto SSH a 2222, configurar UFW en todas las VMs, y configurar SSL/TLS en los proxies.

5.3. Ficheros de Configuración Clave
- /etc/netplan/50-cloud-init.yaml: Configuración de red en todas las VMs.
- /etc/nginx/sites-available/default (en proxies): Configuración del balanceo de carga y SSL.
- /etc/keepalived/keepalived.conf: Configuración de VRRP para failover.
- /etc/mysql/mariadb.conf.d/50-server.cnf: Configuración de MariaDB para aceptar conexiones remotas.
- /etc/exports: Configuración de las exportaciones NFS.
- /var/www/html/wiki/LocalSettings.php: Configuración de MediaWiki (se copia entre aplicaciones).
- /etc/redis/redis.conf: Configuración de Redis para aceptar conexiones remotas.
- /etc/dnsmasq.conf: Configuración del servidor DNS local.
- /etc/prometheus/prometheus.yml: Configuración de Prometheus para recolectar métricas.

⚠️ VI. Pruebas y Validación
Prueba Realizada	Resultado Esperado	Resultado Obtenido
Test de Failover de los Proxies (Apagar ha1-proxy)	La VIP (192.168.0.10) debe migrar a ha2-proxy y el servicio debe seguir activo.	OK
Prueba de Balanceo de Carga	El tráfico se distribuye entre app1-wiki y app2-wiki.	OK (se verificó con logs de Nginx)
Test de Sesiones con Redis	Al apagar una app, la sesión del usuario debe persistir en la otra app.	OK
Test de Almacenamiento NFS	Subir una imagen en app1-wiki y visualizarla en app2-wiki.	OK
Test de Seguridad (SSL/Firewall)	El acceso HTTP debe redirigir a HTTPS y el firewall debe bloquear puertos no autorizados.	OK
Test de Monitoreo	Las métricas de todas las VMs deben aparecer en Grafana.	OK

📚 VII. Conclusiones y Lecciones Aprendidas
Se logró implementar una infraestructura de alta disponibilidad para MediaWiki, cumpliendo con los objetivos de resiliencia, seguridad y rendimiento. Se pusieron en práctica conceptos clave de la asignatura, como balanceo de carga, replicación, monitoreo y hardening.

Lecciones aprendidas:
1. La importancia de la planificación y documentación de la red y los roles de cada VM.
2. La configuración de NFS requiere atención a los permisos y a las direcciones IP autorizadas.
3. El uso de una IP virtual con Keepalived es una solución robusta para el failover de los proxies.
4. Redis mejora significativamente el rendimiento y la experiencia de usuario al mantener las sesiones.
5. El monitoreo con Prometheus y Grafana permite tener una visión clara del estado de la infraestructura.

Qué haríamos diferente:
- Automatizar la instalación y configuración con Ansible para reducir errores y tiempo de despliegue.
- Implementar replicación de la base de datos para mayor disponibilidad.
- Utilizar certificados SSL de una entidad certificadora en lugar de autofirmados para evitar advertencias en los navegadores.
