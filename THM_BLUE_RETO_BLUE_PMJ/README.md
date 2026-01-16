# BLUE – Informe CTF (README.md)

# INDICE
- [1. Resumen Ejecutivo](#1-resumen-ejecutivo)
- [2. Objetivo del reto](#2-objetivo-del-reto)
- [3. Banderas del reto](#3-banderas-del-reto)
- [4. Resolución paso a paso - Evidencias](#4-resolución-paso-a-paso---evidencias)
- [5. Herramientas utilizadas](#5-herramientas-utilizadas)
- [6. MITRE ATT&CK](#6-mitre-attck)
- [7. Flags obtenidas](#7-flags-obtenidas)
- [8. Conclusión del reto](#8-conclusión-del-reto)
- [9. Deslinde de responsabilidad](#9-deslinde-de-responsabilidad)

---

## 1. Resumen Ejecutivo

En el laboratorio **BLUE** como parte de la certificación "PMJ" de "HACKER MENTOR" se simuló el compromiso de un servidor Windows vulnerable a **MS17-010 (EternalBlue)** expuesto mediante el servicio **SMB (puerto 445)**.  
Mediante Nmap se identificaron servicios críticos, se confirmó la vulnerabilidad MS17-010 y se seleccionó en **Metasploit** el módulo de explotación correspondiente.  
Tras configurar el host objetivo y las opciones requeridas, se obtuvo una **shell remota** que se actualizó a una sesión **Meterpreter** mediante el módulo de post-explotación `post/multi/manage/shell_to_meterpreter`.  
Con privilegios elevados se ejecutó `hashdump` para extraer hashes de contraseñas, que luego se crackearon en **CrackStation**, obteniendo la contraseña del usuario Jon.  
Finalmente se localizaron y leyeron tres flags en la raíz del sistema, en el directorio de configuración de Windows y en los documentos del usuario Jon, evidenciando un compromiso completo del sistema desde el acceso inicial hasta el acceso a información sensible.

---

## 2. Objetivo del reto

- Enumerar servicios y puertos expuestos en la máquina BLUE.  
- Detectar si el sistema es vulnerable a **MS17-010**.  
- Seleccionar y ejecutar el exploit adecuado en **Metasploit**.  
- Mejorar la shell obtenida a una sesión **Meterpreter** para post-explotación.  
- Extraer y crackear hashes de contraseñas de usuarios locales.  
- Localizar y leer las tres flags distribuidas en ubicaciones clave del sistema Windows.

---

## 3. Banderas del reto

A continuación se listan las banderas del CTF, traducidas al español y con sus respuestas según las evidencias:

| # | Pregunta (traducida)                                                                                                                
|---|---------------------------------------------------------------------------------------------------------------------------------------|
| 1 | ¿Cuántos puertos con número menor a 1000 están abiertos?                                                                              |
| 2 | ¿A qué es vulnerable esta máquina? (Respuesta en formato ms??-???)                                                                   | 
| 3 | Encuentra el código de explotación que se ejecutará contra la máquina. ¿Cuál es la ruta completa del módulo de exploit?             | 
| 4 | Muestra las opciones y configura el valor requerido. ¿Cómo se llama ese valor? (en mayúsculas)                                      | 
| 5 | (THM Bandera 6) Tras investigar cómo convertir una shell a Meterpreter, ¿cuál es el nombre del módulo post que usaremos (ruta completa)? |
| 6 | (THM Bandera 7) Ya en el módulo post, al hacer *show options*, ¿qué opción es obligatorio cambiar?                                   | 
| 7 | Dentro de la sesión Meterpreter elevada, al ejecutar `hashdump`, ¿cuál es el nombre del usuario no predeterminado?                   | 
| 8 | Copia el hash de contraseña a un archivo e investiga cómo crackearlo. ¿Cuál es la contraseña en texto claro?                         | 
| 9 | Flag1. Esta flag se encuentra en la raíz del sistema.                                                                                | 
|10 | Flag2. Esta flag se encuentra en la ubicación donde Windows almacena las contraseñas.                                                |
|11 | Flag3. Esta flag se encuentra en un lugar excelente para saquear información: los documentos del usuario (normalmente interesantes). |

---

## 4. Resolución paso a paso - Evidencias

### Evidencia 1 – Escaneo Nmap de puertos (Bandera 1)

**Objetivo:**  
Identificar puertos y servicios abiertos en la máquina objetivo para conocer la superficie de ataque y responder a la bandera sobre puertos abiertos bajo el 1000.

**Descripción de la bandera:**  
*“How many ports are open with a port number under 1000?”* – ¿Cuántos puertos con número menor a 1000 están abiertos?

**Comandos útiles:**  
- `sudo nmap -sS -vvv -O -A -T4 10.201.36.13`  

**Interpretación técnica**  
El escaneo muestra que, entre los puertos analizados, los puertos **135/tcp**, **139/tcp** y **445/tcp** están abiertos, todos ellos por debajo del 1000. Estos puertos están relacionados con servicios de Windows y SMB, muy utilizados tanto para administración remota como para compartición de archivos y, por ello, frecuentes objetivos de explotación. De esta salida se obtiene la respuesta de la bandera 1: hay **3 puertos** abiertos bajo el 1000.

**Persona común:**  
Es como revisar una casa para ver por qué puertas y ventanas se podría entrar. En este caso se encuentran tres puertas principales abiertas.

**Evidencia:** 
FLAG_1

---

### Evidencia 2 – Confirmación de vulnerabilidad MS17-010 (Bandera 2)

**Objetivo:**  
Comprobar si el servicio SMB expuesto es vulnerable a **MS17-010** y responder a la bandera que pide identificar dicha vulnerabilidad.

**Descripción de la bandera:**  
*“What is this machine vulnerable to? (Answer in the form of: ms??-???)”* – ¿A qué es vulnerable esta máquina? (en formato ms??-???)  

**Comandos útiles:**  
- Reporte HTML de Nmap generado con el script `smb-vuln-ms17-010` (el comando exacto no es visible; solo se dispone del resultado).  

**Interpretación técnica**  
En el reporte HTML de Nmap se observa la sección de **Host Script Output** con el script `smb-vuln-ms17-010`. Allí se indica el estado como **VULNERABLE** y se describe la vulnerabilidad como *Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)*. Esto confirma que el servidor Windows no está parcheado frente al boletín MS17-010 y que el servicio SMBv1 es vulnerable a ejecución remota de código.

**Persona común:**  
Es como consultar una ficha técnica y descubrir que el modelo de cerradura de la casa tiene un defecto grave que permite abrirla con una herramienta que cualquiera puede descargar.

**Evidencia:** 
FLAG_2

---

### Evidencia 3 – Selección del exploit EternalBlue en Metasploit (Bandera 3)

**Objetivo:**  
Encontrar en Metasploit el módulo de explotación adecuado para MS17-010 y responder a la bandera sobre la ruta completa del exploit.

**Descripción de la bandera:**  
*“Find the exploitation code we will run against the machine. What is the full path of the code? (Ex: exploit/........)”* – Encuentra el módulo de explotación que se ejecutará contra la máquina. ¿Cuál es su ruta completa?

**Comandos útiles:**  
- `search ms17-010`  

**Interpretación técnica**  
Dentro de `msfconsole`, se ejecuta `search ms17-010`, que devuelve una lista de módulos relacionados. Entre ellos se observa el exploit **`exploit/windows/smb/ms17_010_eternalblue`**, que es el elegido para explotar la vulnerabilidad MS17-010 sobre el servicio SMB de la máquina objetivo. Esta ruta completa es la respuesta a la bandera 3.

**Persona común:**  
Es como revisar una caja de herramientas y elegir la herramienta exacta diseñada para aprovechar el fallo conocido de una cerradura concreta.

**Evidencia:** 
FLAG_3 & FLAG_3.1

---

### Evidencia 4 – Configuración de RHOSTS en Metasploit (Bandera 4)

**Objetivo:**  
Configurar el host objetivo en el módulo SMB relacionado con MS17-010 y responder a la bandera que pregunta por el nombre del valor requerido.

**Descripción de la bandera:**  
*“Show options and set the one required value. What is the name of this value? (All caps for submission)”* – Muestra las opciones y configura el valor requerido. ¿Cómo se llama este valor?

**Comandos útiles:**  
- `show options`  
- `set RHOSTS 10.201.35.215`  

**Interpretación técnica**  
La salida de `show options` en el módulo `auxiliary/scanner/smb/smb_ms17_010` muestra varias opciones, entre ellas `RHOSTS`, marcada como requerida. El comando `set RHOSTS 10.201.35.215` establece la IP del objetivo en esa opción, permitiendo que el módulo se ejecute contra el host correcto. De esta evidencia se concluye que el nombre del valor requerido es **RHOSTS**, lo cual responde a la bandera 4.

**Persona común:**  
Es como decirle al GPS cuál es la dirección exacta a la que debe llevarte antes de iniciar el viaje.

**Evidencia:** 
FLAG_4

---

### Evidencia 5 – Selección del módulo `shell_to_meterpreter` (Bandera 5 / THM Bandera 6)

**Objetivo:**  
Encontrar un módulo de post-explotación en Metasploit que permita convertir una shell tradicional en una sesión Meterpreter y responder a la bandera sobre su ruta completa.

**Descripción de la bandera:**  
*“If you haven't already, background the previously gained shell (CTRL + Z). Research online how to convert a shell to meterpreter shell in metasploit. What is the name of the post module we will use? (Exact path, similar to the exploit we previously selected)”* – Tras enviar la shell al background, investiga cómo convertirla en una sesión Meterpreter y da la ruta completa del módulo post.

**Comandos útiles:**  
- `search shell_to_meterpreter`  
- `use 0`  

**Interpretación técnica**  
La búsqueda `search shell_to_meterpreter` en `msfconsole` devuelve el módulo **`post/multi/manage/shell_to_meterpreter`**, descrito como un upgrade de shell a Meterpreter. Con `use 0` se selecciona ese módulo para su posterior configuración. Esta ruta exacta es la respuesta a la bandera identificada como THM Bandera 6.

**Persona común:**  
Es como cambiar una linterna sencilla por una multiherramienta mucho más completa, una vez que ya se está dentro de la casa.

**Evidencia:** 
FLAG_5

---

### Evidencia 6 – Opción requerida SESSION (Bandera 6 / THM Bandera 7)

**Objetivo:**  
Identificar qué opción es obligatorio configurar en el módulo `shell_to_meterpreter` para indicar sobre qué sesión se ejecutará y responder a la bandera.

**Descripción de la bandera:**  
*“Select this (use MODULE_PATH). Show options, what option are we required to change?”* – Selecciona el módulo, muestra las opciones y di qué opción es obligatorio cambiar.

**Comandos útiles:**  
- `show options`  

**Interpretación técnica**  
Al ejecutar `show options` dentro del módulo `post/multi/manage/shell_to_meterpreter`, se listan varias opciones. Entre ellas aparece **`SESSION`**, marcada como requerida y descrita como *The session to run this module on*. Esto indica que, para ejecutar el módulo correctamente, se debe indicar el número de sesión sobre el que se va a realizar el upgrade a Meterpreter. El nombre de esta opción, `SESSION`, responde a la bandera.

**Persona común:**  
Es como elegir sobre cuál de varias llamadas telefónicas vas a activar el altavoz y grabarla; hay que indicar el número de la llamada correcta.

**Evidencia:** 
FLAG_6

---

### Evidencia 7 – Volcado de hashes con `hashdump` (Bandera 7 – usuario no predeterminado)

**Objetivo:**  
Extraer los hashes de contraseñas desde la sesión Meterpreter elevada y responder a la bandera sobre el usuario no predeterminado.

**Descripción de la bandera:**  
*“Within our elevated meterpreter shell, run the command 'hashdump'. This will dump all of the passwords on the machine as long as we have the correct privileges to do so. What is the name of the non-default user?”* – Ejecuta `hashdump` en la sesión Meterpreter elevada y di el nombre del usuario no predeterminado.

**Comandos útiles:**  
- `hashdump`  

**Interpretación técnica**  
La ejecución de `hashdump` muestra los hashes de varias cuentas locales, incluyendo `Administrator`, `Guest` y **`Jon`**. Mientras que Administrator y Guest son cuentas estándar, la cuenta Jon (RID 1000) es una cuenta adicional creada en el sistema. Por ello se identifica como el usuario no predeterminado, respondiendo a la bandera 7.

**Persona común:**  
Es como obtener una lista de todas las llaves de una casa y ver que, además de las llaves “de fábrica”, hay una llave adicional creada para un usuario concreto.

**Evidencia:** 
FLAG_7

---

### Evidencia 8 – Crackeo de hashes en CrackStation (Bandera 8 – contraseña de Jon)

**Objetivo:**  
Crackear el hash de la cuenta Jon para obtener su contraseña en texto claro y responder a la bandera correspondiente.

**Descripción de la bandera:**  
*“Copy this password hash to a file and research how to crack it. What is the cracked password?”* – Copia el hash de contraseña, investiga cómo crackearlo y da la contraseña resultante.

**Comandos útiles:**  
- (Acción en navegador) Introducir el hash en **CrackStation**.  

**Interpretación técnica**  
Los hashes obtenidos en la evidencia anterior se pegan en la web de CrackStation. El servicio devuelve el resultado marcando en verde los hashes que ha conseguido resolver. Para el hash asociado a Jon, CrackStation indica la contraseña **`alqfna22`**, lo que muestra que se trata de una contraseña débil incluida en diccionarios públicos.

**Persona común:**  
Es como llevar una llave cifrada a un cerrajero automático que prueba todas las combinaciones hasta descubrir cuál es la correcta y te la escribe en un papel.

**Evidencia:** 
FLAG_8

---

### Evidencia 9 – Flag1 en la raíz del sistema (Bandera 9)

**Objetivo:**  
Localizar y leer la primera flag del reto, almacenada en la raíz del sistema, para confirmar el acceso al sistema de archivos.

**Descripción de la bandera:**  
*“Flag1? This flag can be found at the system root.”* – Flag1, ubicada en la raíz del sistema.

**Comandos útiles:**  
- `search -f flag*.txt`  
- `cd /`  
- `ls`  
- `cat flag1.txt`  

**Interpretación técnica**  
La búsqueda de archivos `flag*.txt` con `search -f flag*.txt` devuelve, entre otras, la ruta `C:\flag1.txt`. Tras cambiar al directorio raíz (`cd /`) y listar su contenido (`ls`), se confirma la existencia del archivo `flag1.txt`. La lectura mediante `cat flag1.txt` muestra la flag **`flag{a...}`**, demostrando que se tiene control suficiente para leer archivos directamente en la raíz del sistema.

**Persona común:**  
Es como encontrar una nota en el recibidor de la casa que dice “ya has accedido a la casa”, confirmando que el intruso está dentro.

**Evidencia:** 
FLAG_9 & FLAG_9.1  

---

### Evidencia 10 – Flag2 en `C:\Windows\System32\config` (Bandera 10)

**Objetivo:**  
Localizar la segunda flag en la ubicación donde Windows almacena información crítica de cuentas y configuración, evidenciando un nivel de acceso elevado.

**Descripción de la bandera:**  
*“Flag2? This flag can be found at the location where passwords are stored within Windows.”* – Flag2, ubicada donde Windows almacena las contraseñas.

**Comandos útiles:**  
- `search -f flag*.txt`  
- `cd C:\Windows\System32\config`  
- `pwd`  
- `ls`  
- `cat flag2.txt`  

**Interpretación técnica**  
En la búsqueda de flags se detecta `C:\Windows\System32\config\flag2.txt`. Tras cambiar a ese directorio, `pwd` confirma la ruta y `ls` muestra múltiples archivos de sistema junto a `flag2.txt`. La lectura con `cat flag2.txt` devuelve **`flag{s...}`**, lo que indica que se tiene acceso a la zona donde se encuentra la base SAM y otros hives del registro de Windows.

**Persona común:**  
Es como entrar en la sala donde se guardan todas las llaves maestras y combinaciones de la organización y encontrar una nota que reconoce que ya tienes acceso a ese almacén.

**Evidencia:** 
FLAG_10, FLAG_10.1 & FLAG_10.2 

---

### Evidencia 11 – Flag3 en `C:\Users\Jon\Documents` (Bandera 11)

**Objetivo:**  
Localizar la tercera flag en los documentos del usuario Jon, representando el acceso a información bajo el perfil del usuario.

**Descripción de la bandera:**  
*“flag3? This flag can be found in an excellent location to loot. After all, Administrators usually have pretty interesting things saved.”* – Flag3, ubicada en un lugar excelente para saquear información: los documentos del usuario.

**Comandos útiles:**  
- `search -f flag*.txt`  
- `cd Users\Jon\Documents\`  
- `ls`  
- `cat flag3.txt`  

**Interpretación técnica**  
La búsqueda de flags muestra la ruta `C:\Users\Jon\Documents\flag3.txt`. Al acceder a dicha carpeta (`cd Users\Jon\Documents\`) y listar su contenido (`ls`), se observan varios archivos, incluido `flag3.txt`. Con `cat flag3.txt` se lee el contenido de la tercera flag, evidenciando que el atacante puede acceder a los documentos del usuario Jon, un lugar donde normalmente se almacenan archivos importantes.

**Persona común:**  
Es como entrar en el despacho privado del administrador y leer los documentos que tiene guardados en su escritorio.

**Evidencia:** 
FLAG_11 & FLAG_11.1

---

## 5. Herramientas utilizadas

- **Nmap** – Escaneo de puertos y servicios, ejecución de scripts de detección de vulnerabilidades SMB.  
- **Metasploit Framework** – Búsqueda y ejecución de módulos de explotación, auxiliares y post-explotación.  
- **Meterpreter** – Post-explotación en la máquina comprometida (hashdump, búsqueda de archivos, lectura de flags).  
- **CrackStation** – Servicio en línea para crackear hashes NTLM mediante diccionarios públicos.  

---

## 6. MITRE ATT&CK

| Fase / Acción principal                               | Táctica MITRE          | Técnica (ID)                               | Comentario breve                                          |
|------------------------------------------------------|------------------------|-------------------------------------------|-----------------------------------------------------------|
| Escaneo de puertos y servicios con Nmap              | Discovery              | Network Service Scanning (T1046)         | Identificación de puertos 135/139/445 y otros servicios.  |
| Comprobación de MS17-010 en SMB                      | Discovery              | Network Service Scanning (T1046)         | Uso de script de Nmap para detectar la vulnerabilidad MS17-010. |
| Explotación de MS17-010 (EternalBlue)                | Initial Access         | Exploitation of Remote Services (T1210)  | Explotación remota del servicio SMB vulnerable.           |
| Mejora de sesión (shell → Meterpreter)               | Execution / Persistence| Remote Services (T1021)                  | Establecimiento y mejora de la sesión remota.             |
| Volcado de hashes con `hashdump`                     | Credential Access      | OS Credential Dumping (T1003)            | Extracción de hashes de cuentas locales.                  |
| Crackeo de hashes en CrackStation                    | Credential Access      | Password Cracking / Brute Force (T1110)  | Obtención de contraseñas en texto claro a partir de hashes.|
| Búsqueda y lectura de archivos `flag*.txt`           | Discovery / Collection | File and Directory Discovery (T1083), Data from Local System (T1005) | Enumeración del sistema de archivos y obtención de flags. |

---

## 7. Flags obtenidas

```text
Flag1
Ruta : C:\flag1.txt

Flag2
Ruta : C:\Windows\System32\config\flag2.txt

Flag3
Ruta : C:\Users\Jon\Documents\flag3.txt
```

---

## 8. Conclusión del reto

El CTF **BLUE** muestra un flujo de ataque realista contra un servidor Windows vulnerable:

1. Un escaneo básico con Nmap permite identificar servicios críticos expuestos (135/139/445).  
2. Un simple script de Nmap revela que el servicio SMB es vulnerable a **MS17-010**, una falla conocida asociada a campañas de ransomware.  
3. Con la ayuda de Metasploit, se selecciona el exploit adecuado y se compromete el servidor de forma automatizada.  
4. La conversión de la shell a Meterpreter facilita la post-explotación y la obtención de información sensible, como hashes de contraseñas.  
5. El uso de un servicio público como CrackStation para crackear los hashes demuestra que contraseñas débiles pueden ser descubiertas rápidamente.  
6. La localización de flags en la raíz del sistema, en el directorio de configuración de Windows y en los documentos del usuario Jon confirma que el atacante tiene control total sobre el equipo y acceso a información de sistema y de usuario.

En conjunto, el reto evidencia la importancia de mantener los sistemas actualizados, deshabilitar protocolos inseguros como SMBv1, aplicar políticas de contraseñas robustas y desplegar monitoreo que permita detectar este tipo de actividades antes de que se conviertan en un incidente real.

---

## 9. Deslinde de responsabilidad

Este documento y las acciones descritas se han elaborado con fines **exclusivamente educativos**, en el contexto de un laboratorio controlado de CTF.  
No deben emplearse estas técnicas contra sistemas, redes o equipos para los que no se cuente con autorización explícita y documentada.  
El autor y los responsables de este material **no asumen responsabilidad** por cualquier uso indebido, ilícito o no autorizado que terceros puedan hacer de la información aquí expuesta.

