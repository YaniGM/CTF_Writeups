<style>
body {
    text-align: justify;
}
</style>

# 🚩 Writeup: [Windows_7]

| 🏷️ Metadatos | Detalle |
| :--- | :--- |
| **Autor** | **Yani Giatas** |
| **Fecha** | 2026-02-04 |
| **Máquina** | [Windows_7] (IP: 10.127.81.193) |
| **S.O.** | Windows 7 |
| **Dificultad** | Medio |
| **Técnicas** | #EternalBlue #SMB #RCE #ExploitationRemoteServices |

---
<br>

## 1. 🔍 Reconocimiento (Recon)

### Escaneo de Puertos
Se realiza un escaneo de la Red local en busca de posibles ip's.

![alt text](<Images/Captura de pantalla (136).png>)

```bash
sudo arp-scan -I enp0s3 --localnet
```

Se expone una ip candidata al corresponder a una VM de VirtualBox, cuya MAC de fabricante comienza por **08:00**

![alt text](<Images/VirtualBox_ParrotOS Security Edition ISO_04_02_2026_11_53_34.png>)

Se lanza un ping de la ip obtenida para comprobar mediante el parámetro TTL si corresponde a una máquina Windows.

![alt text](<Images/VirtualBox_ParrotOS Security Edition ISO_04_02_2026_11_54_26.png>)

A juzgar por la TTL obtenida **128** la ip corresponde a una máquina Windows. Por lo que se deduce que la ip **10.127.81.193** es de la máquina objetivo.

A continuación se lanza el escaneo avanzado de puertos.

```bash
nmap -sV -sC -sS -T5 -n -vvv -Pn -oN escaneo.txt 10.127.81.193
```
<br>

## 2. 🕵️ Enumeración

Protocolo **SMB** corriendo por el puerto **445**.

![alt text](<Images/VirtualBox_ParrotOS Security Edition ISO_04_02_2026_17_26_00.png>)

<br>

## 3. 💥 Explotación (User Flag)

### Análisis de la Vulnerabilidad

Identifiqué el puerto **445** donde corría un servicio de protocolo **SMB** que presenta una vulnerabilidad según el comando de escaneo de este puerto en concreto. 

```bash
sudo nmap -p445 -sS --script=vuln -vvv -Pn 10.127.81.193
```

La vulnerabilidad que se detecta es la siguiente, conocida como *EternalBlue*: 

  * CVE: **CVE-2017-0143**

![alt text](<Images/VirtualBox_ParrotOS Security Edition ISO_04_02_2026_18_04_52.png>)


### Ejecución del Exploit

Lancé por consola ***metasploit***:

![alt text](<Images/VirtualBox_ParrotOS Security Edition ISO_04_02_2026_18_16_48.png>)

```bash
[msf](Jobs:0 Agents:0) exploit(windows/smb/ms17_010_eternalblue) >> run
```

Abro una sesión con **Meterpreter** para conseguir un control total sobre la máquina. Allana el camino para la postexplotación de procesos en la máquina vulnerada.

La sesión de **Meterpreter** es con privilegios de administrador.

![alt text](<Images/VirtualBox_ParrotOS Security Edition ISO_05_02_2026_05_22_49.png>)

<br>


## 4. 🚀 Escalada de Privilegios (Root Flag)

### Enumeración Interna

Acceso al sistema con privilegios de Administrador.

Dentro de la sesión de **Meterpreter** ejecuto ``sysinfo`` y ``getuid`` 

### Explotación

Abro una shell en la sesión de **Meterpreter** mediante ``shell``

### 🏆 Root Flag: ``whoami`` -> ``nt Authority\System``
<br>

## 5. 👩‍💻 El Rincón del Desarrollador

### ❌ El Código Vulnerable

* **Componente Afectado:** El controlador del Kernel de Windows llamado ``srv.sys`` (SMB Server Driver).

* **El Fallo de Código:** Es un error lógico en cómo el protocolo SMBv1 maneja paquetes específicamente manipulados.

* **La Causa Técnica:** El fallo se produce al procesar paquetes con atributos extendidos de archivo (FEA - File Extended Attributes). El servidor no valida correctamente el tamaño de estos atributos, lo que provoca un Buffer Overflow (desbordamiento de búfer) en el "Non-Paged Pool" (memoria del kernel).

* **Consecuencia:** Al desbordar la memoria del kernel, el atacante puede sobrescribir punteros y ejecutar su propio código con los privilegios más altos posibles (SYSTEM), ya que ``srv.sys`` corre en el núcleo del sistema operativo.

* **Impacto al comprometer el sistema:** Al ser una vulnerabilidad de Kernel, si el exploit falla (si sale mal), suele provocar un BSOD (Pantallazo Azul) y reiniciar el servidor. Esto es crítico en entornos de producción.

### ✅ Solución Propuesta (Remediation)

* **Medidas de Corrección (Mitigación):**

**A. La Solución Definitiva (Patching):** Aplicar el boletín de seguridad de Microsoft **MS17-010**.

  * *Acción*: Instalar las actualizaciones de seguridad acumulativas (KB) correspondientes a la versión de Windows afectada. Microsoft lanzó parches incluso para sistemas fuera de soporte (como XP) debido a la gravedad de este fallo.

**B. La Solución de Hardening (Deshabilitar SMBv1):** El protocolo SMBv1 es obsoleto (tiene más de 30 años) y muy inseguro. Debe eliminarse.

  * *Comando (PowerShell)*: ```Disable-WindowsOptionalFeature -Online -FeatureName smb1protocol```

  * *Acción*: Forzar el uso de SMBv2 o SMBv3.

**C. La Solución de Red (Firewall):** El puerto 445 nunca debería estar expuesto directamente a Internet.

  * *Acción*: Configurar el firewall perimetral y el firewall de Windows para bloquear el tráfico entrante al puerto 445 desde IPs no confiables. 
<br>

## 6. 📚 Referencias y Herramientas

  * **Vulnerabilidad (CVE):** [CVE-2017-0143 (NIST Database)](https://nvd.nist.gov/vuln/detail/CVE-2017-0143)

  * **Boletín del Fabricante (Patch):** [Microsoft Security Bulletin MS17-010](https://learn.microsoft.com/en-us/security-updates/securitybulletins/2017/ms17-010)

  * **Herramientas utilizadas:**
    * [Nmap Security Scanner](https://nmap.org/) (Script: `smb-vuln-ms17-010`)

    * [Metasploit Framework](https://www.metasploit.com/)

* **Módulo Exploit:** `exploit/windows/smb/ms17_010_eternalblue`

---

## Writeup elaborado por Yani Giatas. Si te ha servido, conecta conmigo en LinkedIn: https://www.linkedin.com/in/yani-gm/