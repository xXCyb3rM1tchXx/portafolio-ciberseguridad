# 🛡️ Portafolio de Laboratorios CTF — OverTheWire (Bandit)
**Autor:** Mitch  
**Última actualización:** 31 de octubre de 2025

> Documentación ordenada de los ejercicios Bandit (OverTheWire).  
> Contiene objetivos, comandos útiles, procedimiento y evidencia sugerida para los niveles 0–9.  
> **Aviso:** evita publicar contraseñas reales en repositorios públicos. Usa repositorio privado si contienen credenciales.

---

## 📋 Índice
- [Instrucciones generales](#-instrucciones-generales)
- [Estructura del repositorio](#-estructura-del-repositorio)
- [Niveles](#-niveles)
  - [Nivel 0](#nivel-0)
  - [Nivel 1](#nivel-1)
  - [Nivel 2](#nivel-2)
  - [Nivel 3](#nivel-3)
  - [Nivel 4](#nivel-4)
  - [Nivel 5](#nivel-5)
  - [Nivel 6](#nivel-6)
  - [Nivel 7](#nivel-7)
  - [Nivel 8](#nivel-8)
  - [Nivel 9](#nivel-9)
  - [Nivel 10 (nota)](#nivel-10-nota)
- [Recomendaciones finales](#-recomendaciones-finales)
- [Metadatos por laboratorio](#-metadatos-por-laboratorio)

---

# 🔧 INSTRUCCIONES GENERALES
- Los comandos muestran el flujo utilizado en cada ejercicio.  
- Conexión SSH al host de Bandit usualmente en `bandit.labs.overthewire.org` puerto `2220`.  
- No subir contraseñas reales a repositorios públicos. Este repo debe contener **evidencias sanitizadas** o bien mantenerse privado.  
- Para cada nivel incluye: **Objetivo**, **Herramientas**, **Procedimiento**, **Resultado** y **Evidencia**.

---

# 📁 Estructura sugerida del repositorio
```
mitch-cybersecurity-portfolio/
│
├── README.md
└── evidencias/
    ├── capturas/

---

# 🧩 NIVELES

## Nivel 0
**Objetivo:** Conectarse al servidor Bandit y obtener la primera bandera (archivo `readme`).  
**Comandos útiles:** `pwd`, `ls`, `cat`, `ssh`  
**Procedimiento (resumido):**
```bash
pwd
ssh bandit0@bandit.labs.overthewire.org -p 2220   # password: bandit0
ls
cat readme
```
**Resultado:** Lectura del archivo `readme` con la bandera.  
**Evidencia sugerida:** Captura de la salida `cat readme`.

---

## Nivel 1
**Objetivo:** Leer el archivo cuyo nombre es `-`.  
**Comandos útiles:** `ls`, `cat`  
**Procedimiento:**
```bash
ssh bandit1@bandit.labs.overthewire.org -p 2220   # password obtenido en nivel 0
ls
cat ./-
# o
cat -- -
# o
cat < -
```
**Resultado:** Se obtiene la contraseña del siguiente nivel.  
**Evidencia:** Captura de `cat ./-`.

---

## Nivel 2
**Objetivo:** Leer el archivo con nombre `--` (o nombres con caracteres especiales).  
**Comandos útiles:** `ls`, `cat`  
**Procedimiento:**
```bash
ssh bandit2@bandit.labs.overthewire.org -p 2220
ls
cat ./--spaces\ in\ this\ filename--
```
**Resultado:** Lectura del archivo y obtención de la bandera.  
**Evidencia:** Captura de `cat ./--spaces\ in\ this\ filename--`.

---

## Nivel 3
**Objetivo:** Localizar archivo oculto dentro de `inhere`.  
**Comandos útiles:** `ls`, `ls -a`, `cd`, `cat`  
**Procedimiento:**
```bash
ssh bandit3@bandit.labs.overthewire.org -p 2220
ls -a
cd inhere
ls -la
cat ./...Hiding-From-You
```
**Resultado:** Se obtiene la contraseña contenida en el archivo oculto.  
**Evidencia:** `ls -a` y `cat` del archivo.

---

## Nivel 4
**Objetivo:** Encontrar el único archivo legible por humanos dentro de `inhere`.  
**Comandos útiles:** `ls`, `file`, `cat`, `du`, `find`  
**Procedimiento:**
```bash
ssh bandit4@bandit.labs.overthewire.org -p 2220
ls -la inhere/
file inhere/*
cat inhere/<archivo_ascii>
```
**Resultado:** Lectura y obtención de la bandera.  
**Evidencia:** `file inhere/*` y `cat` del archivo ASCII.

---

## Nivel 5
**Objetivo:** Encontrar el archivo legible, no ejecutable y con tamaño **1033 bytes** dentro de `inhere`.  
**Comandos útiles:** `ls`, `find`, `cat`  
**Procedimiento:**
```bash
ssh bandit5@bandit.labs.overthewire.org -p 2220
cd inhere
find . -type f -readable -size 1033c
cat <ruta_resultante>
```
**Resultado:** Se obtiene la contraseña del nivel.  
**Evidencia:** Salida de `find` y `cat`.

---

## Nivel 6
**Objetivo:** Encontrar archivo con propietario `bandit7`, grupo `bandit6`, tamaño **33 bytes**.  
**Comandos útiles:** `find`, `cat` , `user` , `group` , `2>/dev/null`
**Procedimiento:**
```bash
ssh bandit6@bandit.labs.overthewire.org -p 2220
find / -user bandit7 -group bandit6 -size 33c 2>/dev/null
cat <ruta_encontrada>
```
**Resultado:** Obtención de la contraseña del siguiente nivel.  
**Evidencia:** Salida de `find` y `cat`. (Redirigir errores para evitar mucho ruido por permisos).

---

## Nivel 7
**Objetivo:** En `data.txt` encontrar la contraseña asociada a la palabra `"millionth"`.  
**Comandos útiles:** `grep`, `cat`  
**Procedimiento:**
```bash
ssh bandit7@bandit.labs.overthewire.org -p 2220
ls
grep millionth data.txt
```
**Resultado:** Línea que contiene la palabra `millionth` y la contraseña.  
**Evidencia:** Salida de `grep "millionth" data.txt`.

---

## Nivel 8
**Objetivo:** Encontrar la línea única (que aparece sólo una vez) en `data.txt`.  
**Comandos útiles:** `sort`, `uniq`, `cat`  
**Procedimiento:**
```bash
ssh bandit8@bandit.labs.overthewire.org -p 2220
ls
sort data.txt | uniq -u
```
**Resultado:** Línea única que contiene la contraseña.  
**Evidencia:** `sort data.txt | uniq -u`.

---

## Nivel 9
**Objetivo:** Extraer la línea legible precedida por varios `=` en `data.txt`.  
**Comandos útiles:** `strings`, `grep`  
**Procedimiento:**
```bash
ssh bandit9@bandit.labs.overthewire.org -p 2220
ls
strings data.txt | grep "=="
```
**Resultado:** Cadena legible con `==` que contiene la contraseña.  
**Evidencia:** Salida de `strings data.txt | grep "=="`.

---

## Nivel 10
**Objetivo:** La contraseña para el siguiente nivel está almacenada en el archivo `data.txt`, el cual contiene datos codificados en **base64** 
**Comandos útiles:** `cat`, `base64`  , `-d`
**Procedimiento:**
```bash
ssh bandit9@bandit.labs.overthewire.org -p 2220
ls
cat data.txt | base64
cat data.txt | base64 -d
```
**Resultado:** Cadena decodificada en base64  
**Evidencia:** Salida de `cat data.txt | base64 -d`.

---
