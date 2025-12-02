# Proyecto Final SIS313: MediaWiki Universitario


Este repositorio contiene la documentación, scripts y configuraciones para el **Proyecto Final de la asignatura SIS313**. Implementamos una infraestructura de **Alta Disponibilidad** para MediaWiki, diseñada para aguantar caídas de servidores, picos de tráfico y miradas feas del docente.

---

## 👥 El Dream Team
| Rol | Miembro | GitHub |
| :--- | :--- | :--- |
| **Proxy1, Proxy2, DNS, Monitoreo Grafana** | [Villena Mamani Alvaro Fabian] | [@AlvaroFab28] |
| **App1, App2, NFS** | [Castro Siñanis Jose Luis] | [@tu_usuario] |
| **Redis, Base de Datos** | [Villca Araca Jhesica] | [@cero0202] |

---

## 🏗️ Arquitectura de la Bestia

No es solo instalar un Apache y rezar. Acá desacoplamos todo para que sea **Stateless** y escalable.

### 🗺️ Mapa de Red (Topología)

| Hostname | Rol | IP Física | IP Virtual (VIP) | Software Clave |
| :--- | :--- | :--- | :--- | :--- |
| **ha1-proxy** | Balanceador MASTER | `192.168.0.11` | **`192.168.0.10`** | Nginx, Keepalived |
| **ha2-proxy** | Balanceador BACKUP | `192.168.0.12` | `192.168.0.10` | Nginx, Keepalived |
| **app1-wiki** | Nodo Aplicación 1 | `192.168.0.13` | - | Nginx, PHP-FPM, MediaWiki |
| **app2-wiki** | Nodo Aplicación 2 | `192.168.0.14` | - | Nginx, PHP-FPM, MediaWiki |
| **srv-db** | Base de Datos | `192.168.0.17` | - | MariaDB (Hardened) |
| **srv-redis** | Caché & Sesiones | `192.168.0.16` | - | Redis |
| **srv-nfs** | Storage Compartido | `192.168.0.15` | - | NFS Kernel Server |
| **srv-monitor**| Ojo de Sauron | `192.168.0.20` | - | Prometheus, Grafana, DNS |

---

## 🛠️ Tecnologías Implementadas (El Arsenal)

* **Front-End / Balanceo:** **Nginx** manejando SSL/TLS y **Keepalived** (VRRP) para que la IP `.10` flote entre servidores como Messi en el área.
* **Back-End:** **PHP-FPM** procesando código.
* **Base de Datos:** **MariaDB** configurada para accesos remotos seguros.
* **Performance:** **Redis** para que las sesiones no se pierdan y la carga vuele (Caché de Objetos).
* **Storage:** **NFS** para que las imágenes subidas en la App 1 se vean en la App 2 al toque.
* **Observabilidad:** **Prometheus + Grafana** para ver gráficos lindos y saber cuándo explota todo.

---

## 🚀 Guía de Despliegue Rápido (Para impacientes)

### Fase 1: Los Cimientos 🧱
Levantar `srv-db` y `srv-nfs`.
1.  **DB:** Instalar MariaDB, abrir el `bind-address` a `0.0.0.0` y crear usuario con acceso `%`.
2.  **NFS:** Exportar carpeta `/var/nfs/wikipics` con permisos para las IPs de las apps.

### Fase 2: Las Aplicaciones ⚙️
Levantar `app1-wiki`.
1.  Instalar Stack LEMP.
2.  Montar el NFS en `/var/www/html/wiki/images`.
3.  Instalar MediaWiki apuntando a la DB remota.
4.  **Clonar la VM** para crear `app2-wiki` (Acordate de cambiar la MAC Address o se rompe todo).

### Fase 3: El Portero (High Availability) 🚪
Configurar `ha1-proxy` y `ha2-proxy`.
1.  Configurar Nginx como **Upstream** apuntando a `.13` y `.14`.
2.  Configurar **Keepalived**:
    * `ha1`: Prioridad 101 (Master).
    * `ha2`: Prioridad 100 (Backup).
3.  Apuntar el DNS local `wiki.usfx.bo` a la VIP `192.168.0.10`.

### Fase 4: El Nitro (Redis) 🏎️
Levantar `srv-redis`.
1.  En `LocalSettings.php`, configurar `$wgSessionCacheType` usando Redis.
2.  Ahora podés apagar un servidor y la sesión del usuario sigue viva. Magia pura.

### Fase 5: Seguridad Paranoica (Hardening) 🛡️
1.  **SSH:** Cambiado al puerto `2222`. Root login deshabilitado.
2.  **SSL:** Certificados autofirmados. HTTP redirige a HTTPS.
3.  **Firewall (UFW):** Política *Default Deny*. Solo pasa lo que tiene invitación VIP.

---

## 🧪 Pruebas de Estrés (Rompiendo cosas)

Para verificar que esto no es puro humo, hacé lo siguiente:

1.  **Test de Failover:**
    * Dejá un `ping 192.168.0.10 -t` corriendo.
    * Desenchufá el cable de red del **Proxy Master**.
    * *Resultado:* El ping pierde 1 paquete y sigue. El servicio no cae.

2.  **Test de Balanceo:**
    * Revisá los logs de Nginx en los nodos App.
    * *Resultado:* Las peticiones se reparten una para mí, una para vos (Round Robin).

3.  **Test de Seguridad:**
    * Intentá entrar por SSH puerto 22.
    * *Resultado:* `Connection Refused`. (A casa, hacker).

---

## 📸 Capturas de Pantalla

*(Acá podés poner screenshots de tu Dashboard de Grafana o del sitio funcionando)*

---

## 📜 Licencia

Este proyecto es código abierto bajo la licencia **"Si lo rompés, lo pagás"**.
Desarrollado con ❤️, café y estrés en la **USFX**.
