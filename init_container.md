
## Ejemplo práctico: Iniciar un contenedor Ubuntu y usar `curl`

### Paso 1: Verificar que Docker está instalado

```bash
docker --version
```

Este comando verifica si Docker está instalado correctamente. Muestra la versión instalada.

---

### Paso 2: Descargar la imagen de Ubuntu desde Docker Hub

```bash
docker pull ubuntu:22.04
```

**Explicación:**

* `docker pull`: descarga una imagen desde el repositorio de Docker.
* `ubuntu:22.04`: nombre de la imagen y su etiqueta. En este caso, Ubuntu versión 22.04.

---

### Paso 3: Ejecutar un contenedor con la imagen de Ubuntu

```bash
docker run -it --name mi_ubuntu ubuntu:22.04
```

**Explicación:**

* `docker run`: ejecuta un nuevo contenedor.
* `-i`: modo interactivo, permite que el terminal reciba entrada del usuario.
* `-t`: asigna un pseudo-terminal (permite ver la salida como en una terminal).
* `--name mi_ubuntu`: le asigna un nombre al contenedor.
* `ubuntu:22.04`: especifica la imagen base a usar.

Después de ejecutar este comando, estarás dentro del contenedor con una terminal de Ubuntu lista para usar.

---

### Paso 4: Actualizar el sistema e instalar curl dentro del contenedor

```bash
apt update && apt install -y curl
```

**Explicación:**

* `apt update`: actualiza la lista de paquetes disponibles.
* `apt install -y curl`: instala `curl` automáticamente (sin solicitar confirmación).

Este paso te permite probar cómo instalar software dentro de un contenedor.

---

### Paso 5: Salir del contenedor

```bash
exit
```

Este comando cierra la sesión del contenedor y te devuelve a tu sistema anfitrión.

---

### Paso 6: Verificar que el contenedor fue creado

```bash
docker ps -a
```

**Explicación:**

* `docker ps`: lista los contenedores en ejecución.
* `-a`: muestra todos los contenedores, incluidos los que están detenidos.

---

### Paso 7: Volver a ingresar al contenedor

```bash
docker start -ai mi_ubuntu
```

**Explicación:**

* `docker start`: inicia un contenedor detenido.
* `-a`: adjunta la salida del contenedor a la consola actual.
* `-i`: permite la interacción con el contenedor.

---

### Paso 8: Eliminar el contenedor (opcional)

```bash
docker rm mi_ubuntu
```

**Explicación:**

* `docker rm`: elimina un contenedor detenido.
* Este paso es útil para limpiar recursos cuando ya no necesitas el contenedor.


