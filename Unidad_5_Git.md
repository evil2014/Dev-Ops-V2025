
# **Bloque 5: Control de Versiones con Git**

> **Enfoque:** Práctico y teórico, adaptable a Fedora, Ubuntu, Windows
> **Nivel:** Principiante → Intermedio → Avanzado

---

## **5.1 Introducción a Git**

### **5.1.1 Historia y concepto de control de versiones distribuido**

Git fue creado por Linus Torvalds en 2005. A diferencia de los sistemas centralizados como SVN, Git es **distribuido**, permitiendo a cada usuario trabajar con una copia completa del historial.

**Ventajas:**

* Trabajo sin conexión.
* Velocidad en operaciones locales.
* Integridad y trazabilidad.

---

### **5.1.2 Comparación con otros sistemas**

| Característica       | Git    | SVN     |
| -------------------- | ------ | ------- |
| Copia local completa | ✅ Sí   | ❌ No    |
| Trabajo offline      | ✅ Sí   | ❌ No    |
| Velocidad            | ✅ Alta | ❌ Media |
| Ramas ligeras        | ✅ Sí   | ❌ No    |

---

### **5.1.3 Instalación y configuración básica**

```bash
# Ubuntu
sudo apt install git

# Fedora
sudo dnf install git

# Configuración
git config --global user.name "Tu Nombre"
git config --global user.email "correo@example.com"
git config --global init.defaultBranch main
```

---

### **¿Qué es un hash en Git?**

Un **hash** (SHA-1) es un identificador único de 40 caracteres asignado a cada commit. Garantiza la integridad del historial y se usa para referenciar cambios.

Ejemplo:

```
3a2f3e6b7a97b4c3a34e6b7d91e9c9e8bb74ef01
```

---

## **5.2 Flujo de trabajo básico en Git**

### **5.2.1 Inicializar repositorio**

```bash
git init
echo "# Proyecto" > README.md
git add README.md
git commit -m "Primer commit"
```

Clonar repositorio:

```bash
git clone https://github.com/usuario/repositorio.git
```

---

### **5.2.2 Gestión de archivos y diferencias**

```bash
git status
git diff
git add archivo.txt
git commit -m "Mensaje"
```

**Comparar diferencias:**

```bash
git diff                # sin agregar
git diff --staged       # después de agregar
```

**Rastrear quién modificó qué:**

```bash
git blame archivo.txt
```

---

### **5.2.3 Sincronización remota**

```bash
git remote add origin https://github.com/usuario/repo.git
git push -u origin main
git pull origin main
```

---

## **5.3 Colaboración en Git**

### **5.3.1 Ramas**

**Teoría:**
Una **rama** es una línea de desarrollo aislada. Permite experimentar sin afectar el código principal.

**Ventajas:**

* Trabajo paralelo.
* Mejor organización.
* Facilita pruebas y revisiones.

**¿Qué es un merge?**
Un **merge** integra el historial de una rama a otra. Si los cambios son compatibles, la fusión es automática; si no, Git pedirá resolver conflictos.

```bash
git switch main #cambiar de rama otro forma 
git merge rama-nueva
```

**¿Qué es `git diff`?**
Permite comparar versiones del proyecto:

```bash
git diff rama1 rama2
git diff <hash1> <hash2>
```

---

### **5.3.2 Uso de plataformas remotas**

**Teoría:**
GitHub, GitLab y Bitbucket son plataformas para alojar repositorios en la nube. Permiten colaboración, control de versiones y automatización.

**Comandos:**

```bash
git remote add origin https://github.com/usuario/repo.git
git push -u origin main
git pull origin main
```

**Práctica:**

* Crea una cuenta en GitHub.
* Sube un proyecto con `git push`.
* Comparte la URL del repositorio.

---

### **5.3.3 Pull Requests y revisión de código**

**Teoría:**
Un **Pull Request (PR)** es una solicitud para fusionar una rama. Se revisa el código, se discute y finalmente se acepta o rechaza.

**Práctica:**

* Haz un fork de un proyecto.
* Realiza cambios en una rama.
* Crea un PR y pide revisión a tu equipo.


---

## **5.4 Gestión avanzada en Git**

### **5.4.1 Visualización y análisis del historial de cambios (`git log`)**

**Teoría:**

* Git registra todo el historial de cambios en forma de commits, cada uno con su hash, autor, fecha y mensaje.
* `git log` permite explorar este historial de manera detallada.

**Comandos útiles:**

```bash
git log                    # Vista completa
git log --oneline          # Vista resumida (hash corto y mensaje)
git log --graph            # Vista jerárquica con ramas
git log --author="Ana"     # Filtrar por autor
git show <hash>            # Ver contenido de un commit específico
```

**Práctica:**

* Crea varios commits.
* Usa `git log` y `git log --graph --all` para visualizar la estructura del proyecto.

---

### **5.4.2 Resolución de conflictos y reversiones (`git reset`, `git revert`)**

**Teoría:**

####  `git reset`

* Modifica el puntero `HEAD` a un commit anterior.
* Tipos:

  * `--soft`: mantiene cambios en staging.
  * `--mixed`: mantiene cambios en archivos.
  * `--hard`: elimina todo cambio posterior al commit.

```bash
git reset --soft HEAD~1
git reset --mixed HEAD~2
git reset --hard <hash>
```

####  `git revert`

* Crea un nuevo commit que revierte los cambios de un commit anterior.
* No borra historial.

```bash
git revert <hash>
```

**Práctica:**

* Simula errores y usa `reset` y `revert` para volver a estados anteriores.
* Usa `git status` y `git log` para observar los efectos.

---

### **5.4.3 Rebase y stash: optimización de flujos de trabajo**

####  `git rebase`

**Teoría:**

* Reordena commits para mantener una historia lineal y limpia.
* Se utiliza especialmente antes de fusionar ramas.

```bash
git switch rama-secundaria
git rebase main
```

**Consejo:** Si ocurre un conflicto durante el rebase:

```bash
# Después de resolverlo
git add .
git rebase --continue

# Cancelar
git rebase --abort
```

####  `git stash`

**Teoría:**

* Guarda temporalmente los cambios sin comprometerlos.
* Útil cuando debes cambiar de rama sin perder tu progreso actual.

```bash
git stash           # Guardar
git stash list      # Ver lista
git stash apply     # Restaurar el más reciente
git stash pop       # Restaurar y eliminar
```

**Práctica:**

* Modifica archivos, usa `stash` y cambia de rama.
* Restaura después con `apply` o `pop`.

---

### **5.4.4 Herramientas gráficas para Git**

**Teoría:**
Herramientas visuales facilitan el manejo de ramas, merges y conflictos para usuarios no expertos en la terminal.

####  Herramientas recomendadas:

| Herramienta    | Sistema         | Características principales                              |
| -------------- | --------------- | -------------------------------------------------------- |
| **GitKraken**  | Multiplataforma | Interfaz moderna, flujo visual de ramas y commits        |
| **SourceTree** | Windows/macOS   | Gratis, interfaz clara, integración con Bitbucket        |
| **VSCode**     | Multiplataforma | Soporte nativo para Git, staging, commits y merge visual |

**Práctica:**

* Instala una de estas herramientas.
* Realiza operaciones básicas: crear ramas, hacer commits, merges y ver el historial gráfico.

