## 2.1 Comandos básicos para administración de sistemas

### 2.1.1 Gestión de archivos y directorios (`ls`, `cp`, `mv`, `rm`)

#### Teoría

| Comando | Parámetro | Descripción                                       |
| ------- | --------- | ------------------------------------------------- |
| `ls`    | `-l`      | Detallado (permisos, propietario, tamaño, fecha)  |
|         | `-a`      | Incluye ocultos                                   |
|         | `-h`      | Tamaño legible (KB, MB…)                          |
|         | `-i`      | Muestra número de inodo                           |
| `cp`    | `-r, -R`  | Recursivo (directorios)                           |
|         | `-p`      | Preserva timestamps, permisos, propietario        |
|         | `-a`      | Archive (recursivo + preserva enlaces y permisos) |
|         | `-v`      | Verbose                                           |
| `mv`    | `-i`      | Interactivo (pregunta antes de sobrescribir)      |
|         | `-n`      | No sobrescribir archivos existentes               |
|         | `-v`      | Verbose                                           |
| `rm`    | `-r`      | Recursivo                                         |
|         | `-f`      | Forzar, sin preguntar                             |
|         | `-v`      | Verbose                                           |

Los archivos y directorios en Linux son **inodos**: estructuras con metadatos (UID, GID, permisos, timestamps) y punteros a los bloques de datos.

#### Prácticas

```bash
# Crear estructura de ejemplo
mkdir ~/proyecto_lab && cd ~/proyecto_lab
mkdir documentos scripts resultados

# Crear archivos
touch documentos/informe1.txt documentos/informe2.txt documentos/informe3.txt

# Listar con detalles e inodo
ls -li documentos/

# Copiar preservando atributos y verbose
cp -apv documentos/informe1.txt resultados/resumen1.txt

# Mover interactivamente
mv -iv documentos/informe2.txt scripts/analisis2.txt

# Eliminar con verbose
rm -v documentos/informe3.txt

# Verificar
ls -l documentos scripts resultados

# Simular restauración
echo "Temporal" > resultados/temporal.txt
rm -f resultados/temporal.txt
ls resultados/
echo "Restaurado manualmente" > resultados/temporal.txt
```

---

### 2.1.2 Gestión de usuarios y permisos (`chmod`, `chown`)

#### Teoría

1. **Permisos y clases**

   * Tres clases: propietario (u), grupo (g), otros (o).
   * Tres tipos: lectura (r = 4), escritura (w = 2), ejecución (x = 1).

2. **Notación octal**
   Cada dígito (0–7) es la suma de permisos:

   ```
    7 → rwx  | 6 → rw- | 5 → r-x | 4 → r--  
    3 → -wx  | 2 → -w- | 1 → --x | 0 → ---
   ```

   `chmod 750 archivo` → propietario rwx (7), grupo r-x (5), otros --- (0).

3. **Modo simbólico**

   ```
   chmod [u/g/o/a][+/-/=][r/w/x] archivo
   ```

   * `u+rw` añade lectura/escritura al propietario.
   * `go-w` quita escritura a grupo y otros.
   * `a=x` deja solo ejecución para todos.

4. **Bits especiales**

   * **SUID** (`u+s`): ejecuta con UID del propietario.
   * **SGID** (`g+s`): idem para GID; en directorios, hereda grupo.
   * **Sticky** (`o+t`): en directorios impide borrar archivos ajenos (p.ej. `/tmp`, modo `1777`).

5. **`chown`**
   Cambia propietario y grupo:

   ```bash
   chown usuario:grupo archivo
   ```

#### Prácticas

```bash
# 1. Crear usuarios
sudo useradd -m alumno1
sudo useradd -m alumno2

# 2. Crear archivo y asignar propietario/grupo
touch ~/proyecto_lab/datos_privados.txt
sudo groupadd alumnos
sudo chown alumno1:alumnos ~/proyecto_lab/datos_privados.txt

# 3. Permisos octal
chmod 640 ~/proyecto_lab/datos_privados.txt  # rw- r-- ---
ls -l ~/proyecto_lab/datos_privados.txt

chmod 750 ~/proyecto_lab/datos_privados.txt  # rwx r-x ---
ls -l ~/proyecto_lab/datos_privados.txt

# 4. Permisos simbólicos
chmod g-w,o= ~/proyecto_lab/datos_privados.txt  # quita w a grupo, 0 a otros
ls -l ...

chmod a+r ~/proyecto_lab/datos_privados.txt    # añadir lectura a todos
ls -l ...

# 5. Probar accesos
sudo su - alumno2 -c 'cat ~/proyecto_lab/datos_privados.txt'  # lectura denegada
sudo su - alumno1 -c 'echo "OK" >> ~/proyecto_lab/datos_privados.txt'  # escritura OK

# 6. SUID en script
sudo tee /usr/local/bin/elevado.sh > /dev/null << 'EOF'
#!/bin/bash
echo "UID real: \$UID"
echo "UID efectivo: \$EUID"
EOF
sudo chmod 755 /usr/local/bin/elevado.sh
sudo chown root:root /usr/local/bin/elevado.sh
/usr/local/bin/elevado.sh  # EUID=UID=tu usuario
sudo chmod u+s /usr/local/bin/elevado.sh
/usr/local/bin/elevado.sh  # EUID=0 (root)
```

### 2.1.3 Gestión de procesos (`ps`, `top`, `kill`, `nice`, `renice`)

#### Teoría

| Comando  | Parámetro        | Descripción ampliada                                                                                                                                                                         |
| -------- | ---------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ps`     | `aux`            | Muestra **todos** los procesos en formato BSD: columnas típicas son **USER**, **PID**, **%CPU**, **%MEM**, **VSZ**, **RSS**, **STAT**, **START**, **TIME**, **COMMAND**. STAT indica estado. |
|          | `-ef`            | Muestra todos los procesos en formato UNIX: columnas **UID**, **PID**, **PPID**, **C**, **STIME**, **TTY**, **TIME**, **CMD**. Útil para scripts y parseo programático.                      |
| `top`    | *tecla* `P`      | Ordena dinámicamente la lista de procesos por **%CPU**, de mayor a menor.                                                                                                                    |
|          | *tecla* `M`      | Ordena por **%MEM**, de mayor a menor.                                                                                                                                                       |
|          | *tecla* `k`      | Permite introducir un **PID** y señal para matar un proceso directamente desde la interfaz.                                                                                                  |
| `kill`   | `<PID>`          | Envía **SIGTERM** (15), petición de terminación “amable”. El proceso puede limpiar recursos antes de morir.                                                                                  |
|          | `-9 <PID>`       | Envía **SIGKILL** (9), fuerza cierre inmediato, no puede atraparse ni limpiarse.                                                                                                             |
| `nice`   | `-n <valor>`     | Lanza un comando con un valor de “niceness” entre **-20** (máxima prioridad) y **+19** (mínima). Controla su peso en la CPU.                                                                 |
| `renice` | `<valor> -p PID` | Cambia el nice de un proceso en ejecución (necesita permisos si baja el valor).                                                                                                              |

##### Scheduler CFS (Completely Fair Scheduler)

* CFS asigna a cada proceso un **vruntime** proporcional a su nice y uso de CPU; trata de “compartir” el tiempo de CPU de forma justa.
* Los procesos con menor nice (más prioridad) acumulan vruntime más despacio, accediendo antes a la CPU.
* Internamente usa un **árbol rojo-negro** ordenado por vruntime para elegir al siguiente proceso.

##### Estados de un proceso (`STAT` en `ps`)

| Estado     | Código | Descripción                                                 |
| ---------- | ------ | ----------------------------------------------------------- |
| Running    | `R`    | Proceso ejecutándose o listo para ejecutarse                |
| Sleeping   | `S`    | Interrumpible: espera un evento (I/O, temporizador, señal…) |
| Disk Sleep | `D`    | No interrumpible: espera I/O de disco                       |
| Stopped    | `T`    | Parado por señal o ptrace                                   |
| Zombie     | `Z`    | Terminó pero su padre no ha leído su estado (defunct)       |

---

#### Prácticas

```bash
# 1. Bucle infinito en background para generar carga
python3 -c "while True: pass" &

# 2. Identificar su PID y estado
ps aux | grep '[p]ython3'

# 3. Terminar educadamente con SIGTERM
kill <PID>

# 4. Forzar cierre con SIGKILL si no responde
kill -9 <PID>

# 5. Verificar que ya no aparece
ps -ef | grep '[p]ython3'

# 6. Iniciar un proceso con baja prioridad (niceness +10)
nice -n 10 yes > /dev/null &

# 7. Monitorear en tiempo real con top
#    - Presiona P para ordenar por CPU, M para memoria, k para matar procesos
top
```



## 2.2 Monitoreo de jobs, servicios y sistemas

### 2.2.1 Verificación de estado de servicios (`systemctl`, `service`)

#### Teoría

Una **unidad** `.service` en `systemd` consta de:

```ini
[Unit]
Description=...
After=network.target
Requires=...

[Service]
Type=simple|forking|oneshot|notify
ExecStart=/ruta/al/binario
ExecStartPre=...
ExecStartPost=...
Restart=no|on-failure|always
RestartSec=5
User=...
Group=...
AmbientCapabilities=...

[Install]
WantedBy=multi-user.target
```

* **Type** define cuándo se considera “activo”.
* Dependencias aseguran orden de arranque.
* `systemctl isolate <target>` cambia runlevel.

#### Prácticas

```bash
# Ver estado
systemctl status sshd.service
systemctl status cron.service

# Control básico
sudo systemctl stop sshd.service
sudo systemctl start sshd.service
sudo systemctl reload sshd.service
sudo systemctl enable cron.service
sudo systemctl disable cron.service

# Crear servicio demo
sudo tee /usr/local/bin/mi_servicio.sh > /dev/null << 'EOF'
#!/bin/bash
echo "\$(date): OK" >> /var/log/mi_servicio.log
EOF
sudo chmod +x /usr/local/bin/mi_servicio.sh

sudo tee /etc/systemd/system/mi_servicio.service > /dev/null << 'EOF'
[Unit]
Description=Servicio demo

[Service]
ExecStart=/usr/local/bin/mi_servicio.sh
Restart=always
RestartSec=3
User=nobody

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl start mi_servicio.service
sudo systemctl enable mi_servicio.service

# Logs
journalctl -u mi_servicio.service -o cat -f
```

---

### 2.2.2 Visualización y análisis de logs (`journalctl`, `tail`, `grep`)

#### Teoría

1. **Journal binario**

   * Ubicación: `/var/log/journal/`.
   * Niveles: `emerg(0)` → `debug(7)`.
   * Filtros:

     ```
     journalctl -u nginx -p err --since "2025-06-20" --until "2025-06-25"
     journalctl SYSLOG_IDENTIFIER=sshd _PID=1234
     ```
   * Salidas: `-o short`, `-o verbose`, `-o json`, `-o json-pretty`.

2. **Logrotate**

   * Configuraciones en `/etc/logrotate.d/`.
   * Parámetros: `rotate`, `compress`, `delaycompress`, `postrotate`.

3. **Logs textuales**

   * Rutas comunes: `/var/log/syslog`, `/var/log/auth.log`, `/var/log/nginx/*.log`.
   * Combinar `tail -f` + `grep --line-buffered`.

#### Prácticas

```bash
# Journal
journalctl -k -n 50
journalctl -u sshd.service --since "today" -p warning
journalctl -u cron.service -o json-pretty | jq '.[].MESSAGE'

# Logs clásicos
sudo tail -n 20 /var/log/auth.log
sudo tail -f /var/log/auth.log | grep --line-buffered "Failed password"

# Rotar manual
sudo logrotate -f /etc/logrotate.d/nginx
```

---

### 2.2.3 Monitoreo de rendimiento (`htop`, `iostat`, `vmstat`)

#### Teoría

* **Load Average**: procesos en cola de CPU en 1/5/15 min (ideal ≤ núcleos).
* **iostat** (`sysstat`):

  * `%util`: % de tiempo ocupado.
  * `await`: latencia total.
  * `r/s`, `w/s`: ops por segundo.
* **vmstat**:

  * `r`: procesos listos.
  * `b`: procesos bloqueados.
  * `swpd`: swap usado.
  * `free`, `buff`, `cache`: memoria.
  * `si`, `so`: swap-in/swap-out.

#### Prácticas

```bash
# htop
sudo apt install -y htop
htop  # F6 → PERCENT_CPU, F9 → kill

# iostat
sudo apt install -y sysstat
iostat -x 2 5

# vmstat
vmstat 2 5
```

---

## 2.3 Instalación de aplicaciones y actualizaciones

### 2.3.1 Gestión de paquetes (`apt`, `yum`/`dnf`)

#### Teoría

| Comando              | Parámetro                       | Descripción                              |
| -------------------- | ------------------------------- | ---------------------------------------- |
| `apt update`         | —                               | Actualiza índice de paquetes             |
| `apt upgrade`        | —                               | Actualiza paquetes instalados            |
| `apt install`        | `-y`, `--no-install-recommends` | Instala sin preguntar y sin recomendados |
| `apt remove`         | `--purge`                       | Elimina también configs                  |
| `apt autoremove`     | —                               | Elimina dependencias huérfanas           |
| `apt-cache`          | `policy <paquete>`              | Muestra versiones disponibles            |
| `add-apt-repository` | `<repo>`                        | Agrega PPA (Ubuntu)                      |

RHEL/CentOS usan `yum` o `dnf` con comandos equivalentes y grupos (`groupinstall`).

#### Prácticas

```bash
# Ubuntu/Debian
sudo apt update
apt-cache policy nginx
sudo apt install -y nginx
nginx -v
sudo apt remove --purge -y nginx
sudo apt autoremove -y

# CentOS/RHEL
sudo yum makecache
yum list installed httpd
sudo yum install -y httpd
httpd -v
sudo yum remove -y httpd
sudo yum autoremove -y
```

---

### 2.3.2 Instalación y configuración de servicios

#### Teoría

* **MariaDB/MySQL**:

  * Config en `/etc/mysql/my.cnf` y `/etc/mysql/conf.d/*.cnf`.
  * Secciones: `[mysqld]` (buffers, conexiones), `[client]`.
  * Hardening con `mysql_secure_installation`.
  * Backups con `mysqldump` o `xtrabackup`.

#### Prácticas

```bash
# Instalar
sudo apt update
sudo apt install -y mariadb-server

# Hardening
sudo mysql_secure_installation

# Iniciar y habilitar
sudo systemctl start mariadb
sudo systemctl enable mariadb

# Crear DB y usuario
sudo mysql -u root -p << 'SQL'
CREATE DATABASE prueba_db;
CREATE USER 'usuario_prueba'@'localhost' IDENTIFIED BY 'password123';
GRANT ALL PRIVILEGES ON prueba_db.* TO 'usuario_prueba'@'localhost';
FLUSH PRIVILEGES;
SQL

# Conectar
mysql -u usuario_prueba -p prueba_db
```

---

### 2.3.3 Configuración de servicios y demonios

#### Teoría

* **Nginx**:

  * Modo event-driven basado en `epoll`.
  * Parámetros en `nginx.conf`:

    ```
    worker_processes auto;
    worker_rlimit_nofile 100000;
    events { worker_connections 1024; multi_accept on; }
    http {
      keepalive_timeout 65;
      client_max_body_size 10m;
      proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=my_cache:10m inactive=60m;
    }
    ```
  * `reload` (graceful) vs `restart` (downtime breve).

* **systemd** para demonios:

  * Control de reinicios: `StartLimitBurst`, `StartLimitIntervalSec`.
  * Timeouts: `TimeoutStartSec`, `TimeoutStopSec`.
  * Capacidades: `AmbientCapabilities=CAP_NET_BIND_SERVICE`.

#### Prácticas

```bash
# Ajustar Nginx
sudo sed -i 's/worker_processes .*/worker_processes auto;/' /etc/nginx/nginx.conf
sudo sed -i 's/worker_connections .*/worker_connections 1024;/' /etc/nginx/nginx.conf
sudo nginx -t
sudo systemctl reload nginx

# Crear demonio Python
sudo tee /usr/local/bin/reloj_demo.py > /dev/null << 'EOF'
#!/usr/bin/env python3
import time
while True:
    print(time.strftime("%Y-%m-%d %H:%M:%S"))
    time.sleep(10)
EOF
sudo chmod +x /usr/local/bin/reloj_demo.py

sudo tee /etc/systemd/system/reloj_demo.service > /dev/null << 'EOF'
[Unit]
Description=Demo reloj

[Service]
Type=simple
ExecStart=/usr/local/bin/reloj_demo.py
Restart=always
RestartSec=5
User=nobody
AmbientCapabilities=CAP_SYS_TIME
TimeoutStartSec=10

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl start reloj_demo.service
sudo systemctl enable reloj_demo.service
journalctl -u reloj_demo.service -f
```

---

## 2.4 Automatización de tareas con Shell scripting

### 2.4.1 Creación de scripts básicos

#### Teoría

1. **Shebang**: indica intérprete (`#!/bin/bash`).
2. **Variables** y **quoting**:

   * `"..."` para expansiones,
   * `'...'` para literales.
   * `${VAR#pref}` / `${VAR%suf}` para manipulaciones.
3. **Control de flujo**:

   * `if …; then …; elif …; else …; fi`
   * `for i in …; do …; done`
   * `while …; do …; done`
4. **Parsing de opciones** con `getopts`.

#### Prácticas

```bash
# Script de respaldo
cat > ~/backup_home.sh << 'EOF'
#!/bin/bash
SHELL=/bin/bash
ORIG="$HOME/documentos_importantes"
DEST="$HOME/respaldos"
TS=$(date +%Y%m%d_%H%M%S)
FILE="backup_$TS.tar.gz"
mkdir -p "$DEST"
tar -czf "$DEST/$FILE" -C "$ORIG" .
echo "Respaldo generado en $DEST/$FILE"
EOF
chmod +x ~/backup_home.sh
~/backup_home.sh

# Script de chequeo disco
cat > ~/check_disk.sh << 'EOF'
#!/bin/bash
USO=$(df / | tail -1 | awk '{print $5}' | tr -d '%')
if [ "$USO" -ge 80 ]; then
  echo "ALERTA: uso de / es $USO%"
  exit 1
else
  echo "Uso de /: $USO% (OK)"
fi
EOF
chmod +x ~/check_disk.sh
~/check_disk.sh

# Bucle con getopts
cat > ~/parse_args.sh << 'EOF'
#!/bin/bash
while getopts ":f:o:h" opt; do
  case $opt in
    f) FILE="$OPTARG";;
    o) OUT="$OPTARG";;
    h) echo "-f archivo -o salida"; exit 0;;
    \?) echo "Opción inválida: -$OPTARG"; exit 1;;
  esac
done
echo "Archivo: $FILE, Salida: $OUT"
EOF
chmod +x ~/parse_args.sh
./parse_args.sh -f input.txt -o output.txt
```

---

### 2.4.2 Programación de tareas automáticas

#### Teoría

* **cron**: usa `/etc/crontab`, `/etc/cron.d/*`, `crontab -e`.

  * Campos: `m h dom mon dow`.
  * Entorno: `SHELL`, `PATH`, `MAILTO`.
* **at**: ejecución puntual;

  * Fechas: `at now + 2 hours`, `at 09:30 tomorrow`.
  * `atq` lista, `atrm` borra.

#### Prácticas

```bash
# Cron diario
crontab -e
# Añadir:
SHELL=/bin/bash
PATH=/usr/local/bin:/usr/bin:/bin
0 23 * * * /home/usuario/backup_home.sh >> /home/usuario/backup.log 2>&1

crontab -l

# Programar con at
sudo systemctl start atd && sudo systemctl enable atd
echo "/home/usuario/check_disk.sh >> ~/check.log" | at 09:30 tomorrow
atq
atrm <job>
```

---

### 2.4.3 Ejecución y depuración de scripts

#### Teoría

* **Debug**:

  * `set -x` → muestra cada comando.
  * `set -e` → sale al primer error.
* **Trap**:

  ```bash
  trap 'echo "Error en $LINENO"; exit 1' ERR
  ```
* **Unit testing** con `bats`.

#### Prácticas

```bash
# Debug y trap
cat > ~/count_errors.sh << 'EOF'
#!/bin/bash
set -xe
trap 'echo "Error en linea $LINENO"; exit 1' ERR
DIR=~/logs_demo
for f in "$DIR"/*.log; do
  [ -f "$f" ] || continue
  cnt=$(grep -c "ERROR" "$f")
  echo "$(basename "$f"): $cnt errores"
done
EOF
chmod +x ~/count_errors.sh
~/count_errors.sh

# Ping con retry y trap
cat > ~/test_ping.sh << 'EOF'
#!/bin/bash
set -e
trap 'echo "Fallo ping en linea $LINENO"; exit 1' ERR
IP="8.8.8.8"
if ping -c1 $IP &>/dev/null; then
  echo "Host $IP accesible"
else
  echo "Host inaccesible, reiniciando eth0"
  sudo ip link set eth0 down
  sleep 2
  sudo ip link set eth0 up
fi
EOF
chmod +x ~/test_ping.sh
~/test_ping.sh
```

---

## 2.5 Seguridad en Linux

### 2.5.1 Gestión de tablas de ruteo y configuración de firewall (`iptables`, `firewalld`, `nftables`)

#### Teoría

* **NETFILTER** en el kernel:

  * Tablas: `filter`, `nat`, `mangle`, `raw`.
  * Cadenas: `PREROUTING`, `INPUT`, `FORWARD`, `OUTPUT`, `POSTROUTING`.
* **iptables**

  ```
  iptables -P INPUT DROP      # política por defecto
  iptables -A INPUT -p tcp --dport 22 \
      -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
  iptables -L -v
  ```
* **nftables** (nuevo sucesor):

  ```bash
  nft add table ip filter
  nft 'add chain ip filter input { type filter hook input priority 0; }'
  nft add rule ip filter input tcp dport ssh ct state new,established accept
  ```
* **firewalld**

  ```
  firewall-cmd --get-active-zones
  firewall-cmd --zone=public --add-service=http --permanent
  firewall-cmd --reload
  firewall-cmd --query-service=http
  ```

#### Prácticas

```bash
# Ruteo
sudo ip route add 10.10.10.0/24 via 192.168.1.1 dev eth0
ip route show | grep 10.10.10.0
sudo ip route del 10.10.10.0/24

# iptables
sudo iptables -F
sudo iptables -P INPUT DROP
sudo iptables -A INPUT -p tcp --dport 22 \
    -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
sudo iptables -A INPUT -m conntrack \
    --ctstate ESTABLISHED,RELATED -j ACCEPT
sudo iptables -L -v

# nftables
sudo nft add table ip filter
sudo nft 'add chain ip filter input { type filter hook input priority 0; }'
sudo nft add rule ip filter input tcp dport 22 ct state new,established accept

# firewalld
sudo systemctl start firewalld
sudo firewall-cmd --zone=public --add-service=http --permanent
sudo firewall-cmd --reload
sudo firewall-cmd --query-service=http
```

---

### 2.5.2 Configuración de puertos y servicios seguros (SSH, HTTPS)

#### Teoría

* **SSH** (`/etc/ssh/sshd_config`):

  * `Port` (cambia puerto),
  * `PermitRootLogin no`,
  * `PasswordAuthentication no`,
  * `AllowUsers user1 user2`,
  * `MaxAuthTries`, `LoginGraceTime`.
* **TLS/HTTPS**:

  * Certificados auto-firmados vs CA.
  * Handshake TLS 1.2/1.3.
  * Config en Nginx:

    ```nginx
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'HIGH:!aNULL:!MD5';
    ssl_prefer_server_ciphers on;
    ssl_certificate /etc/ssl/mi_cert/servidor.crt;
    ssl_certificate_key /etc/ssl/mi_cert/servidor.key;
    ```

#### Prácticas

```bash
# Fortalecer SSH
sudo sed -i 's/#Port 22/Port 2222/' /etc/ssh/sshd_config
sudo sed -i 's/#PermitRootLogin .*/PermitRootLogin no/' /etc/ssh/sshd_config
sudo sed -i 's/#PasswordAuthentication .*/PasswordAuthentication no/' /etc/ssh/sshd_config
echo "AllowUsers tu_usuario otro_usuario" | sudo tee -a /etc/ssh/sshd_config
sudo sshd -t
sudo systemctl reload sshd

# Generar certificado
sudo mkdir -p /etc/ssl/mi_cert
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/ssl/mi_cert/servidor.key \
  -out  /etc/ssl/mi_cert/servidor.crt \
  -subj "/C=MX/ST=Coahuila/L=Saltillo/O=MiEmpresa/OU=IT/CN=localhost"

# Configurar Nginx HTTPS
sudo tee /etc/nginx/sites-available/default > /dev/null << 'EOF'
server {
  listen 443 ssl http2;
  server_name localhost;
  ssl_certificate     /etc/ssl/mi_cert/servidor.crt;
  ssl_certificate_key /etc/ssl/mi_cert/servidor.key;
  ssl_protocols       TLSv1.2 TLSv1.3;
  ssl_ciphers         'HIGH:!aNULL:!MD5';
  ssl_prefer_server_ciphers on;
  location / { root /var/www/html; index index.html; }
}
EOF
sudo nginx -t
sudo systemctl reload nginx
```

---

### 2.5.3 Detección y corrección de vulnerabilidades (`nmap`, `fail2ban`, `lynis`)

#### Teoría

* **nmap NSE**

  * Categorías: `auth`, `default`, `vuln`, `exploit`.
  * Ejemplo:

    ```bash
    nmap -sV -O --script vuln 192.168.1.100
    ```
* **fail2ban**

  * Jails en `/etc/fail2ban/jail.d/` o `jail.local`.
  * Parámetros: `enabled`, `port`, `filter`, `logpath`, `maxretry`, `bantime`, `findtime`, `banaction`.
* **lynis**

  * Auditoría con \~200 pruebas: FS, autenticación, paquetes, logs, PCI, HIPAA.
  * Perfil en `/etc/lynis/custom.prf`.

#### Prácticas

```bash
# nmap escaneo vulnerabilidades
sudo nmap -sV -O --script vuln 192.168.1.100

# Configurar fail2ban para SSH
sudo apt install -y fail2ban
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo sed -i 's/\[sshd\]/[sshd]\nenabled = true\nport = 2222\nmaxretry = 5\nbantime = 600\nfindtime = 600/' /etc/fail2ban/jail.local
sudo systemctl restart fail2ban
sudo fail2ban-client status sshd

# Auditoría completa con lynis
sudo apt install -y lynis
sudo lynis audit system
# Ver recomendaciones en /var/log/lynis.log
```
