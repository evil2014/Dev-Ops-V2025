## 2.5.1 Seguridad en Linux

### Gestión de tablas de ruteo y configuración de firewall con iptables (Ubuntu y Fedora)

---

### ¿Qué es `iptables`?

`iptables` es una herramienta que permite configurar reglas de filtrado de paquetes para el firewall del kernel Linux (Netfilter). Estas reglas determinan qué tráfico entra, sale o se reenvía por el sistema.

---

### Conceptos básicos

**Cadenas principales (chains):**

* `INPUT`: tráfico entrante al sistema (por ejemplo, SSH).
* `OUTPUT`: tráfico saliente generado por el sistema.
* `FORWARD`: tráfico que pasa a través del sistema (enrutamiento).

**Políticas (policies):** regla por defecto si no hay coincidencias.

**Estados de conexión (`conntrack`):**

* `NEW`: conexión nueva.
* `ESTABLISHED`: parte de una conexión ya establecida.
* `RELATED`: conexión relacionada (como una transferencia FTP de datos).
* `INVALID`: paquetes mal formados.

---

### Comandos con explicación detallada

#### 1. Borrar reglas existentes

```bash
sudo iptables -F
```

* `-F` o `--flush`: elimina todas las reglas actuales de todas las cadenas.
* Se recomienda hacerlo antes de definir nuevas reglas para evitar conflictos.

---

#### 2. Establecer política por defecto de DROP (bloquear)

```bash
sudo iptables -P INPUT DROP
```

* `-P`: establece la política por defecto para una cadena.
* `INPUT`: aplica a paquetes que ingresan al sistema.
* `DROP`: descarta los paquetes (no responde).

> Equivale a: "bloquea todo lo que no esté explícitamente permitido".

---

#### 3. Permitir conexiones entrantes a SSH (puerto 22)

```bash
sudo iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
```

Desglose:

* `-A INPUT`: agrega (`-A`) esta regla a la cadena `INPUT`.
* `-p tcp`: solo aplica a paquetes del protocolo TCP.
* `--dport 22`: restringe al **puerto de destino** 22 (por defecto usado por SSH).
* `-m conntrack`: utiliza el módulo de seguimiento de conexiones (connection tracking).
* `--ctstate NEW,ESTABLISHED`: aplica solo si es una nueva conexión o parte de una ya establecida.
* `-j ACCEPT`: si se cumple la condición, **acepta** el paquete (permite el tráfico).

> Esto permite iniciar una conexión SSH y continuarla.

---

#### 4. Permitir paquetes relacionados a conexiones ya existentes

```bash
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
```

Desglose:

* `-A INPUT`: agrega esta regla a la cadena `INPUT`.
* `-m conntrack`: invoca el módulo de seguimiento de conexiones.
* `--ctstate ESTABLISHED,RELATED`: permite solo paquetes que ya forman parte de una conexión establecida o que están relacionados (por ejemplo, conexiones de datos FTP).
* `-j ACCEPT`: permite estos paquetes.

> Esto es fundamental para permitir el flujo de respuesta de conexiones iniciadas por el sistema o ya autorizadas.

---

#### 5. Ver reglas activas

```bash
sudo iptables -L -v
```

Desglose:

* `-L`: lista las reglas actuales.
* `-v`: modo detallado (verbose), muestra número de paquetes y bytes que han coincidido con cada regla.

> Útil para verificar el orden de reglas y el tráfico que las ha activado.

---

### Ejemplo completo en Ubuntu o Fedora

```bash
# Borrar reglas existentes
sudo iptables -F

# Política por defecto: bloquear todo el tráfico entrante
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT  # salida permitida

# Permitir conexiones locales (loopback)
sudo iptables -A INPUT -i lo -j ACCEPT

# Permitir tráfico entrante relacionado a conexiones salientes
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Permitir SSH
sudo iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
```

---

### Guardar las reglas

**Ubuntu:**

```bash
sudo apt install iptables-persistent
sudo netfilter-persistent save
```

**Fedora:**

```bash
sudo service iptables save
```

---

### Recomendaciones

* Define reglas **por servicios necesarios** (no uses reglas generales como `ACCEPT ALL`).
* Usa `DROP` como política por defecto en `INPUT` y `FORWARD`.
* Siempre permite `lo` (interfaz local).
* Monitorea las reglas con `iptables -L -v`.
