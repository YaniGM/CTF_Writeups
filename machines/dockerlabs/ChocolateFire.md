# 🚩 Writeup: [ChocolateFire]

| 🏷️ Metadatos | Detalle |
| :--- | :--- |
| **Autor** | **Yani Giatas** |
| **Fecha** | 2026-02-03 |
| **Máquina** | [ChocolateFire] (IP: 10.88.0.2) |
| **S.O.** | Linux |
| **Dificultad** | Fácil |
| **Técnicas** | #Auth-Bypass #RCE |

---
<br>

## 1. 🔍 Reconocimiento (Recon)

### Escaneo de Puertos
Comenzamos realizando un escaneo avanzado para identificar servicios expuestos.

```bash
nmap -sV -sC -sS -T5 -n -vvv -Pn -oN escaneo_02.txt 10.88.0.2
```
<br>

## 2. 🕵️ Enumeración

Servicio Web (Puerto 9090)
Al visitar http://10.88.0.2:9090, encontré una página de login.

<br>

## 3. 💥 Explotación (User Flag)

### Análisis de la Vulnerabilidad

Identifiqué el puerto 9090 donde corría un servicio de protocolo http que presentaba un panel de inicio de sesión 'openfire' cuya versión es 4.7.4. Haciendo búquedas en fuentes de acceso abierto se indica que este panel presenta una vulnerabilidad.
La herramienta metasloit de Parrot identifica la vulnerabilidad con en 

### Ejecución del Exploit

Lancé por consola metasploit:

```bash
[msf](Jobs:0 Agents:0) exploit(multi/http/openfire_auth_bypass_rce_cve_2023_32315) >> run
```

![alt text](<Metasploit_VirtualBox_ParrotOS Security Edition ISO_03_02_2026_10_19_26.png>)

<br>


## 4. 🚀 Escalada de Privilegios (Root Flag)

### Enumeración Interna

### Explotación
<br>

## 5. 👩‍💻 El Rincón del Desarrollador

### ❌ El Código Vulnerable

### ✅ Solución Propuesta (Remediation)
<br>

## 6. 📚 Referencias y Herramientas

  * [Enlace al CVE oficial]

  * [Herramienta Nmap]

  * [GTFOBins]

---

## Writeup elaborado por Yani Giatas. Si te ha servido, conecta conmigo en LinkedIn: https://www.linkedin.com/in/yani-gm/