## 2.4 Automatización de tareas con Shell scripting

### 2.4.1 Creación de scripts básicos

#### Teoría

**Shebang (`#!`)**
La primera línea de un script indica al sistema qué intérprete debe usarse para ejecutarlo.
Ejemplo:

```bash
#!/bin/bash
```

Esto especifica que el script debe ejecutarse con Bash.

**Variables y quoting (comillas)**

* Variables se declaran sin espacios:

  ```bash
  NOMBRE="Eduardo"
  ```
* Uso de comillas:

  * `"..."`: permite la expansión de variables.

    ```bash
    echo "Hola $NOMBRE"
    ```
  * `'...'`: trata el contenido como literal.

    ```bash
    echo 'Hola $NOMBRE'
    ```

**Manipulación de variables**

* Eliminar prefijos o sufijos:

  ```bash
  ARCHIVO="reporte_final.txt"
  echo "${ARCHIVO%.txt}"      # reporte_final
  echo "${ARCHIVO#reporte_}"  # final.txt
  ```

**Control de flujo**

* Condicional `if`:

  ```bash
  if [ -f archivo.txt ]; then
    echo "Es un archivo"
  elif [ -d archivo.txt ]; then
    echo "Es un directorio"
  else
    echo "No existe"
  fi
  ```

* Bucle `for`:

  ```bash
  for f in *.txt; do
    echo "Archivo: $f"
  done
  ```

* Bucle `while`:

  ```bash
  i=1
  while [ $i -le 5 ]; do
    echo "Iteración $i"
    ((i++))
  done
  ```

**Procesamiento de opciones con `getopts`**
Permite manejar argumentos como `-f archivo -o salida` en la línea de comandos:

```bash
while getopts ":f:o:h" opt; do
  case $opt in
    f) FILE="$OPTARG";;
    o) OUT="$OPTARG";;
    h) echo "Uso: -f archivo -o salida"; exit 0;;
    \?) echo "Opción inválida: -$OPTARG"; exit 1;;
  esac
done
```

---

#### Prácticas

**Script de respaldo (`backup_home.sh`):**

```bash
#!/bin/bash
SHELL=/bin/bash
ORIG="$HOME/documentos_importantes"
DEST="$HOME/respaldos"
TS=$(date +%Y%m%d_%H%M%S)
FILE="backup_$TS.tar.gz"
mkdir -p "$DEST"
tar -czf "$DEST/$FILE" -C "$ORIG" .
echo "Respaldo generado en $DEST/$FILE"
```

Explicación:

* Define origen y destino.
* Crea un nombre único con fecha/hora.
* Comprime los archivos de `ORIG` y los guarda en `DEST`.

---

**Script de chequeo de disco (`check_disk.sh`):**

```bash
#!/bin/bash
USO=$(df / | tail -1 | awk '{print $5}' | tr -d '%')
if [ "$USO" -ge 80 ]; then
  echo "ALERTA: uso de / es $USO%"
  exit 1
else
  echo "Uso de /: $USO% (OK)"
fi
```

Explicación:

* Usa `df` para obtener el uso del disco `/`.
* Si es mayor o igual a 80%, emite una alerta.
* Si es menor, informa que está bien.

---

**Script con `getopts` (`parse_args.sh`):**

```bash
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
```

Explicación:

* Procesa las opciones `-f`, `-o` y `-h`.
* Muestra los valores introducidos al final.

---

### 2.4.2 Programación de tareas automáticas

#### Teoría

**cron**
Permite programar tareas recurrentes. Usa los siguientes campos:

```
m h dom mon dow comando
```

* `m`: minuto (0-59)
* `h`: hora (0-23)
* `dom`: día del mes (1-31)
* `mon`: mes (1-12)
* `dow`: día de la semana (0-7; 0 y 7 son domingo)

Variables útiles:

* `SHELL=/bin/bash` para definir el intérprete
* `PATH=...` para asegurar acceso a comandos del sistema
* `MAILTO=usuario` para recibir la salida del script por correo (si está configurado)

**at**
Permite programar tareas una sola vez en un momento futuro.

Formatos válidos:

```bash
at now + 2 hours
at 09:30 tomorrow
```

Comandos útiles:

* `atq`: ver tareas pendientes
* `atrm <número>`: eliminar tarea programada

---

#### Prácticas

**Tarea diaria con `cron`**
Edita el crontab:

```bash
crontab -e
```

Agregar:

```bash
SHELL=/bin/bash
PATH=/usr/local/bin:/usr/bin:/bin
0 23 * * * /home/usuario/backup_home.sh >> /home/usuario/backup.log 2>&1
```

Explicación:

* Ejecuta `backup_home.sh` todos los días a las 23:00.
* Guarda la salida y errores en `backup.log`.

Ver tareas programadas:

```bash
crontab -l
```

---

**Tarea puntual con `at`**

1. Asegurar que el servicio esté activo:

```bash
sudo systemctl start atd
sudo systemctl enable atd
```

2. Programar tarea:

```bash
echo "/home/usuario/check_disk.sh >> ~/check.log" | at 09:30 tomorrow
```

3. Ver tareas pendientes:

```bash
atq
```

4. Eliminar tarea programada:

```bash
atrm <número>
```

