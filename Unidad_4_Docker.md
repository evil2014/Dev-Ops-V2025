# Bloque 4: Contenedores, Docker, Dockerfile, Docker Compose, Kubernetes, Redes y Alta Disponibilidad

> Versión: 2025-07-18
> Plataformas objetivo: Fedora Linux y Ubuntu Linux (soporta Debian derivado)
> Nivel: Principiante → Intermedio → Avanzado
> Estilo: Explicación técnica sin íconos, con prácticas paso a paso y comandos verificados. Ajusta versiones según tu entorno.

---

## Tabla de contenido

* [4.1 Introducción a los contenedores](#41-introducción-a-los-contenedores)

  * [4.1.1 Historia y evolución](#411-historia-y-evolución)
  * [4.1.2 Contenedores vs Máquinas Virtuales](#412-comparación-entre-máquinas-virtuales-y-contenedores)
  * [4.1.3 Ventajas y desventajas](#413-ventajas-y-desventajas-de-los-contenedores)
* [4.2 Docker y Kubernetes: introducción y despliegue básico](#42-docker-y-kubernetes-introducción-y-despliegue-básico)

  * [4.2.1 Instalación y configuración de Docker](#421-instalación-y-configuración-de-docker)
  * [4.2.2 Creación de imágenes Docker (Dockerfile básico y avanzado)](#422-creación-de-imágenes-docker-dockerfile-básico-y-avanzado)
  * [4.2.3 Introducción a Kubernetes: Pods, Services y Deployments](#423-introducción-a-kubernetes-concepto-de-pods-servicios-y-despliegues)
  * [4.2.4 Docker Compose: teoría y prácticas](#424-docker-compose-teoría-y-prácticas)
* [4.3 Redes y conectividad entre contenedores](#43-redes-y-conectividad-entre-contenedores)

  * [4.3.1 Redes Docker: bridge, host, none](#431-introducción-a-la-red-de-contenedores-docker)
  * [4.3.2 Redes Docker personalizadas (bridge, macvlan, overlay)](#432-configuración-de-redes-docker-bridge-macvlan-overlay)
  * [4.3.3 Redes en Kubernetes: Services, LoadBalancers y CNI](#433-redes-en-kubernetes-servicios-balanceadores-de-carga-y-networking-entre-pods)
* [4.4 Clusters y balanceadores de carga con Kubernetes](#44-clusters-y-balanceadores-de-carga-con-kubernetes)

  * [4.4.1 Creación y gestión de clusters Kubernetes](#441-creación-y-gestión-de-clusters-kubernetes)
  * [4.4.2 Despliegue de aplicaciones escalables](#442-despliegue-de-aplicaciones-escalables-en-kubernetes)
  * [4.4.3 Balanceadores y alta disponibilidad](#443-configuración-de-balanceadores-de-carga-y-servicios-de-alta-disponibilidad)
* [4.5 Laboratorios integradores](#45-laboratorios-integradores)

  * [Lab 1: Primer contenedor (Fedora/Ubuntu)](#lab-1-primer-contenedor-fedoraubuntu)
  * [Lab 2: Construye tu primera imagen con Dockerfile](#lab-2-construye-tu-primera-imagen-con-dockerfile)
  * [Lab 3: Dockerfile avanzado (multi-stage, usuario no root, cache)](#lab-3-dockerfile-avanzado-multi-stage-usuario-no-root-cache)
  * [Lab 4: Docker Compose básico (web + Redis)](#lab-4-docker-compose-básico-web--redis)
  * [Lab 5: Docker Compose con base de datos y volúmenes persistentes](#lab-5-docker-compose-con-base-de-datos-y-volúmenes-persistentes)
  * [Lab 6: Escalamiento con docker compose y balance con Nginx reverse proxy](#lab-6-escalamiento-con-docker-compose-y-balance-con-nginx-reverse-proxy)
  * [Lab 7: De Docker Compose a Kubernetes con Kompose](#lab-7-de-docker-compose-a-kubernetes-con-kompose)
  * [Lab 8: Microservicio escalable en Kubernetes con Service tipo LoadBalancer (MetalLB)](#lab-8-microservicio-escalable-en-kubernetes-con-service-tipo-loadbalancer-metallb)
* [4.6 Guías rápidas de referencia](#46-guías-rápidas-de-referencia)

  * [Comandos Docker más usados](#comandos-docker-más-usados)
  * [Directivas Dockerfile de referencia rápida](#directivas-dockerfile-de-referencia-rápida)
  * [Claves docker-compose.yaml más comunes](#claves-docker-composeyaml-más-comunes)
  * [kubectl esenciales](#kubectl-esenciales)
* [4.7 Buenas prácticas, seguridad y optimización](#47-buenas-prácticas-seguridad-y-optimización)
* [4.8 Preguntas de repaso / evaluación](#48-preguntas-de-repaso--evaluación)
* [4.9 Anexos](#49-anexos)

  * [A: Instalación offline de Docker](#a-instalación-offline-de-docker)
  * [B: Registro privado (Harbor/Registry)](#b-registro-privado-harborregistry)
  * [C: Troubleshooting rápido](#c-troubleshooting-rápido)

---

## 4.1 Introducción a los contenedores

### 4.1.1 Historia y evolución

**Línea de tiempo resumida**

* 1979: chroot en Unix separa vistas del sistema de archivos.
* 2000: FreeBSD Jails ofrece aislamiento de procesos y red.
* 2001-2004: Solaris Containers / Zones introduce control de recursos.
* 2008: LXC (Linux Containers) usa cgroups y namespaces.
* 2013: Docker abstrae LXC, empaqueta aplicaciones en imágenes portables, registra capas y simplifica distribución.
* 2015+: OCI (Open Container Initiative) normaliza formato de imagen y runtime.
* 2014-2016: Nace Kubernetes (Google → CNCF) para orquestar contenedores a escala.

 Relaciona cada hito con un problema de la época: aislamiento, densidad, portabilidad, escalamiento.

Actividad rápida de discusión en clase: ¿Por qué Docker explotó en adopción frente a LXC directo?

---

### 4.1.2 Comparación entre máquinas virtuales y contenedores

| Dimensión       | Máquinas Virtuales (VM)                  | Contenedores                                                 |
| --------------- | ---------------------------------------- | ------------------------------------------------------------ |
| Aislamiento     | Kernel independiente por VM (hipervisor) | Comparte kernel del host; aislamiento por namespaces/cgroups |
| Peso imagen     | GB (OS completo)                         | MB a cientos MB                                              |
| Tiempo arranque | Decenas de segundos a minutos            | Segundos o menos                                             |
| Densidad        | Menor (overhead OS completo)             | Alta (muchos contenedores por host)                          |
| Portabilidad    | Depende del hypervisor/imagen            | Imagen única multiplataforma (si arquitectura coincide)      |
| Administración  | Requiere gestión de sistemas completos   | Enfocado en app + dependencias                               |
| Persistencia    | Disco virtual por VM                     | Ephemeral por diseño; persistencia externa (volúmenes)       |

Ejercicio analítico: Haz un cuadro comparativo con un caso real (tu laboratorio) midiendo RAM usada por una VM Ubuntu mínima vs un contenedor Alpine con tu app.

---

### 4.1.3 Ventajas y desventajas de los contenedores

**Ventajas**

* Arranque rápido → ciclos de despliegue cortos.
* Portabilidad entre ambientes (dev, QA, prod).
* Mejor utilización de recursos.
* Facilita CI/CD.
* Versionamiento reproducible (imagen inmutable).

**Desventajas / Riesgos**

* Comparte kernel: vulnerabilidades kernel afectan a todos los contenedores.
* Persistencia no trivial: datos deben externalizarse.
* Observabilidad y debugging distintos al de host tradicional.
* Aislamiento más débil que VM completa (depende de configuración).

Actividad: Clasifica escenarios y elige VM vs contenedor justificando seguridad, rendimiento y gobernanza.

---

## 4.2 Docker y Kubernetes: introducción y despliegue básico

### 4.2.1 Instalación y configuración de Docker

#### Requisitos previos

* CPU con soporte para virtualización (no obligatorio, útil para algunas funciones).
* Kernel Linux moderno con cgroups v2 recomendado.
* Usuario con privilegios sudo.

#### Instalación rápida (Ubuntu LTS)

```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg lsb-release
# Agregar la llave oficial de Docker (ajusta si cambia la URL oficial)
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

#### Instalación rápida (Fedora)

```bash
sudo dnf -y install dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo
sudo dnf install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

#### Habilitar e iniciar

```bash
sudo systemctl enable --now docker
sudo systemctl status docker
```

#### Usar Docker sin sudo (opcional)

```bash
sudo usermod -aG docker $USER
newgrp docker
```

Verifica:

```bash
docker run --rm hello-world
```

Notas docentes: Explica diferencia entre los paquetes `docker-ce` (community edition) vs `docker.io` de repos distro; plugin de compose vs binario clásico `docker-compose`.

---

### 4.2.2 Creación de imágenes Docker (Dockerfile básico y avanzado)

#### Concepto

Una imagen es una plantilla inmutable de capas. Un contenedor es una instancia en ejecución de esa imagen.

#### Anatomía de un Dockerfile

Orden típico recomendado (para maximizar cache):

1. `FROM` (imagen base)
2. `ARG` (valores build-time)
3. `ENV` (variables runtime default)
4. `WORKDIR`
5. Dependencias del sistema (`RUN apt-get ...` / `dnf ...`)
6. Dependencias app (`requirements.txt`, `package.json`)
7. Copia código (`COPY . .`)
8. Build / compile (`RUN make build`)
9. Exponer puertos (`EXPOSE 8080`)
10. Configurar usuario (`USER appuser`)
11. Orden por defecto (`CMD` o `ENTRYPOINT`)

#### Directivas clave

* `FROM imagen[:tag]` Imagen base. Usa tags explícitos.
* `LABEL key=value` Metadatos (autor, versión, VCS).
* `RUN` Ejecuta comandos en capa de build. Combina en una sola línea para reducir capas.
* `COPY` vs `ADD`: pref. `COPY` para archivos locales; `ADD` solo si necesitas extraer tar o URL remota.
* `ENV VAR valor` Define variables de entorno.
* `ARG` Variables disponibles solo en build.
* `WORKDIR /ruta` Establece directorio de trabajo.
* `EXPOSE 80` Documenta puerto (no publica).
* `USER app` Ejecuta proceso como usuario no root (buena práctica).
* `VOLUME /data` Punto de montaje esperado.
* `ENTRYPOINT` Define comando principal inmutable; `CMD` argumentos por defecto (ver combinación).
* `HEALTHCHECK` Mide salud del contenedor.

#### Ejemplo mínimo (Hola mundo Node)

```Dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 3000
CMD ["node", "index.js"]
```

Construcción y prueba:

```bash
docker build -t ejemplo-node:1.0 .
docker run --rm -p 3000:3000 ejemplo-node:1.0
```

#### Multi-stage builds

Reduce tamaño final copiando solo artefactos necesarios.

```Dockerfile
# Etapa de build
FROM golang:1.22-alpine AS build
WORKDIR /src
COPY . .
RUN go build -o app

# Etapa final minimalista
FROM alpine:3.20
WORKDIR /app
COPY --from=build /src/app /app/app
USER 1000:1000
ENTRYPOINT ["/app/app"]
```

#### Optimización de cache

* Ordena capas: primero dependencias que cambian poco.
* Usa `.dockerignore` para excluir node\_modules, .git, tmp.
* Congela versiones de paquetes.

#### Seguridad básica

* Usuario no root.
* Imágenes base mínimas (distroless, alpine, ubi-micro).
* Escanear vulnerabilidades (trivy, grype).

---

### 4.2.3 Introducción a Kubernetes: concepto de Pods, Servicios y Despliegues

#### Conceptos básicos

* Plano de control (API Server, Scheduler, Controller Manager, etcd).
* Nodos (kubelet, kube-proxy, runtime de contenedores).
* Pod: 1+ contenedores que comparten red/almacenamiento.
* ReplicaSet: asegura número deseado de pods.
* Deployment: gestiona ReplicaSets y actualizaciones declarativas.
* Service: endpoint estable para acceder a pods.
* Namespace: segmentación lógica.

#### Mini flujo de trabajo

```bash
kubectl create deployment web --image=nginx --replicas=1
kubectl expose deployment web --port=80 --type=NodePort
kubectl get pods -o wide
kubectl describe svc web
```

---

### 4.2.4 Docker Compose: teoría y prácticas

#### ¿Qué es Docker Compose?

Herramienta (y especificación YAML) para definir y ejecutar aplicaciones multicontenedor. Permite declarar servicios, redes, volúmenes y dependencias en un solo archivo (`compose.yaml` o `docker-compose.yml`). Facilita ambientes de desarrollo reproducibles.

Se ejecuta como `docker compose` (plugin moderno) o `docker-compose` (binario legado). Se recomienda el plugin moderno incluido con Docker Engine reciente.

#### Estructura general de un archivo Compose (versión 3+)

```yaml
services:
  servicio1:
    image: imagen:tag
    build: ./ruta  # opcional
    ports:
      - "8080:80"
    environment:
      - VAR=valor
    volumes:
      - datos:/var/lib/datos
    networks:
      - redapp
  servicio2:
    image: imagen2:tag
    depends_on:
      - servicio1
volumes:
  datos:
networks:
  redapp:
    driver: bridge
```

#### Campos frecuentes

* `build`: ruta a Dockerfile o contexto.
* `image`: nombre/tag imagen (si no se construye).
* `container_name`: opcional, nombre fijo.
* `command` / `entrypoint`: override comando.
* `environment` / `env_file`: variables.
* `ports`: mapeos host\:contenedor.
* `expose`: puertos solo internos a la red.
* `volumes`: persistencia o montajes bind.
* `networks`: redes definidas abajo o externas.
* `depends_on`: orden de arranque (no espera salud real, salvo extensiones).
* `restart`: políticas (no, always, on-failure, unless-stopped).
* `profiles`: activar subconjuntos (dev, test).
* Bloque `deploy:` (modo Swarm u orquestadores compatibles) para replicas, recursos, políticas de actualización.

#### Instalación / verificación del plugin Compose

Verifica:

```bash
docker compose version  # plugin
# o
docker-compose version  # binario legado
```

Instalación binario legado (si lo necesitas):

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/v2.27.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

#### Ciclo básico de trabajo con Compose

1. Crear Dockerfile(s) y código app.
2. Crear `compose.yaml` con servicios (web, db, cache, etc.).
3. Levantar stack: `docker compose up -d`.
4. Ver estado: `docker compose ps`.
5. Logs: `docker compose logs -f nombre_servicio`.
6. Escalar: `docker compose up -d --scale web=3` (modo clásico; no persistente en YAML).
7. Derribar: `docker compose down` (opcional `-v` para volúmenes).

---

## 4.3 Redes y conectividad entre contenedores

### 4.3.1 Introducción a la red de contenedores Docker

Al iniciar Docker, se crean redes por defecto:

* `bridge`: red NAT local. Contenedores pueden resolverse por nombre.
* `host`: comparte stack de red del host (sin NAT, sin aislamiento de puertos).
* `none`: interfaz aislada sin red.

Inspecciona:

```bash
docker network ls
docker network inspect bridge
```

---

### 4.3.2 Configuración de redes Docker (bridge, macvlan, overlay)

#### Redes bridge personalizadas

Ventaja: DNS interno, aislamiento, subred configurable.

```bash
docker network create --driver bridge --subnet 172.25.0.0/16 red_lab
```

Crear contenedores en esa red:

```bash
docker run -d --name srv1 --network red_lab nginx
docker run -it --name cli1 --network red_lab alpine sh
# Dentro de cli1
ping srv1
```

#### Redes macvlan (contendores con IP en LAN física)

Requiere cuidado con NIC y switches.

```bash
docker network create -d macvlan \
  --subnet=192.168.50.0/24 --gateway=192.168.50.1 \
  -o parent=eth0 red_macvlan
```

#### Redes overlay (multi-host)

Necesitas Swarm o plugin. Se cubre brevemente, ver anexo.

---

### 4.3.3 Redes en Kubernetes: servicios, balanceadores de carga y networking entre pods

Principios del modelo de red de Kubernetes:

1. Cada Pod obtiene su propia IP única dentro del clúster.
2. Todos los Pods pueden comunicarse entre sí (sin NAT) por defecto a nivel de red del clúster.
3. Los agentes en un nodo pueden comunicarse con los Pods en ese nodo.

Esto se implementa mediante un plugin CNI (Container Network Interface) como Calico, Flannel, Cilium, Weave, etc.

Tipos de Service:

* `ClusterIP` (por defecto, interno).
* `NodePort` (expone puerto en todos los nodos, rango 30000-32767).
* `LoadBalancer` (usa LB externo o MetalLB en bare metal).
* `ExternalName` (alias DNS externo).

Ejemplo Service YAML:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-svc
spec:
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 8080
  type: ClusterIP
```

---

## 4.4 Clusters y balanceadores de carga con Kubernetes

### 4.4.1 Creación y gestión de clusters Kubernetes

Opciones de laboratorio:

* Minikube (un solo nodo, muy usado en clase).
* kind (Kubernetes in Docker; útil para CI y demos).
* kubeadm (crea clúster real multi-nodo sobre VMs o bare metal).

Ejemplo Minikube:

```bash
minikube start --driver=docker
kubectl cluster-info
kubectl get nodes
```

Ejemplo kind (cluster multi-nodo en contenedores Docker):

```bash
cat > kind-cluster.yaml <<'EOF'
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
EOF
kind create cluster --name lab --config kind-cluster.yaml
kubectl get nodes -o wide
```

### 4.4.2 Despliegue de aplicaciones escalables en Kubernetes

Deployment básico:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: demo-web
  template:
    metadata:
      labels:
        app: demo-web
    spec:
      containers:
      - name: web
        image: nginx:stable-alpine
        ports:
        - containerPort: 80
```

Aplicar y exponer:

```bash
kubectl apply -f deployment.yaml
kubectl expose deployment demo-web --type=NodePort --port=80
kubectl get svc demo-web -o wide
```

Escalar:

```bash
kubectl scale deployment demo-web --replicas=5
```

Actualización rolling:

```bash
kubectl set image deployment/demo-web web=nginx:alpine-slim
kubectl rollout status deployment/demo-web
```

### 4.4.3 Configuración de balanceadores de carga y servicios de alta disponibilidad

Escenarios:

* Cloud: El tipo `LoadBalancer` crea un balanceador externo (AWS ELB, GCP LB, Azure LB).
* Bare metal: Usa MetalLB o ingress controller + reverse proxy.
* Ingress: Reglas HTTP/HTTPS con enrutamiento basado en host/path.

Ejemplo Ingress Nginx + Servicio backend (ver Lab 8).

---

## 4.5 Laboratorios integradores

Cada laboratorio incluye: Objetivo, Requisitos, Pasos, Validación, Limpieza, Preguntas.

### Lab 1: Primer contenedor (Fedora/Ubuntu)

**Objetivo:** Verificar instalación Docker y comprender ciclo run/stop/rm.
**Pasos:**

```bash
docker run --rm hello-world
# Contenedor interactivo
docker run -it --name caja alpine sh
# Instala utilidades dentro del contenedor
apk add --no-cache curl
exit
# Contenedor sigue? No, porque sh terminó.
```

**Validación:** `docker ps -a` muestra historial.

---

### Lab 2: Construye tu primera imagen con Dockerfile

**Objetivo:** Crear imagen Node.js simple.
**Código de ejemplo:**

```
index.js  package.json  Dockerfile
```

`index.js`:

```js
const http = require('http');
const port = process.env.PORT || 3000;
http.createServer((req,res)=>{res.end('Hola desde contenedor');}).listen(port);
```

`package.json` mínimo:

```json
{
  "name": "hola-node",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {"start": "node index.js"},
  "dependencies": {}
}
```

`Dockerfile`:

```Dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci || npm install
COPY . .
ENV PORT=3000
EXPOSE 3000
CMD ["npm", "start"]
```

Construir y correr:

```bash
docker build -t hola-node:1.0 .
docker run -d --name web1 -p 3000:3000 hola-node:1.0
curl http://localhost:3000/
```

---

### Lab 3: Dockerfile avanzado (multi-stage, usuario no root, cache)

**Objetivo:** Minimizar imagen Go.
Ver sección [4.2.2](#422-creación-de-imágenes-docker-dockerfile-básico-y-avanzado) para Dockerfile multi-stage. Pasos:

```bash
docker build -t go-mini:1.0 .
docker image ls | grep go-mini
docker history go-mini:1.0
```

Comparar tamaño vs imagen con etapa única.

---

### Lab 4: Docker Compose básico (web + Redis)

**Objetivo:** Levantar dos servicios conectados en red definida por Compose.

Estructura:

```
compose-lab4/
├── web/
│   ├── app.py
│   └── Dockerfile
└── compose.yaml
```

`web/app.py` (Flask cache simple):

```python
from flask import Flask
import redis, os
app = Flask(__name__)
r = redis.Redis(host=os.environ.get('REDIS_HOST','redis'), port=6379)
@app.route('/')
def index():
    count = r.incr('hits')
    return f"Hola! Visitas: {count}\n"
if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

`web/Dockerfile`:

```Dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY . .
RUN pip install --no-cache-dir flask redis
EXPOSE 5000
CMD ["python", "app.py"]
```

`compose.yaml`:

```yaml
services:
  web:
    build: ./web
    ports:
      - "5000:5000"
    environment:
      - REDIS_HOST=redis
    depends_on:
      - redis
  redis:
    image: redis:7-alpine
    volumes:
      - redisdata:/data
volumes:
  redisdata:
```

Levantar:

```bash
docker compose up -d --build
curl http://localhost:5000/
```

Ver logs:

```bash
docker compose logs -f web
```

---

### Lab 5: Docker Compose con base de datos y volúmenes persistentes

**Objetivo:** Stack web + PostgreSQL con persistencia.

Estructura:

```
compose-lab5/
├── app/
│   ├── server.py
│   ├── requirements.txt
│   └── Dockerfile
└── compose.yaml
```

`app/server.py` ejemplo mínimo con psycopg y tabla hits.
`app/requirements.txt`:

```
flask
psycopg[binary]
```

`app/Dockerfile`:

```Dockerfile
FROM python:3.12-slim
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8000
CMD ["python", "server.py"]
```

`compose.yaml`:

```yaml
services:
  web:
    build: ./app
    ports:
      - "8000:8000"
    environment:
      - DB_HOST=db
      - DB_USER=postgres
      - DB_PASSWORD=postgres
      - DB_NAME=labdb
    depends_on:
      db:
        condition: service_healthy
  db:
    image: postgres:16-alpine
    environment:
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DB=labdb
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      retries: 5
volumes:
  pgdata:
```

Levantar y probar conexión desde contenedor web.

---

### Lab 6: Escalamiento con docker compose y balance con Nginx reverse proxy

**Objetivo:** Ejecutar múltiples réplicas del servicio web y balancear tráfico.

Agrega un servicio `proxy`:

```yaml
services:
  proxy:
    image: nginx:stable-alpine
    ports:
      - "8080:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - web
  web:
    build: ./app
    expose:
      - "8000"
    # sin publicar directo; solo red interna
  ...
```

Escalar:

```bash
docker compose up -d --scale web=3 --build
```

`nginx.conf` upstream dinámico: usa DNS round-robin de Docker (resolver). Ejemplo de configuración en anexo.

Prueba: Realiza múltiples `curl` y observa cabeceras personalizadas por instancia.

---

### Lab 7: De Docker Compose a Kubernetes con Kompose

**Objetivo:** Convertir un stack definido en Compose a manifiestos Kubernetes iniciales.

Instala Kompose:

```bash
curl -L https://github.com/kubernetes/kompose/releases/download/v1.32.0/kompose-linux-amd64 -o kompose
chmod +x kompose
sudo mv kompose /usr/local/bin/
```

Generar YAMLs desde Lab 5:

```bash
cd compose-lab5
kompose convert -f compose.yaml -o k8s/
ls k8s/
```

Aplica:

```bash
kubectl apply -f k8s/
kubectl get all
```

Revisa limitaciones: Kompose produce objetos básicos; ajusta recursos, health probes, secretos.

---

### Lab 8: Microservicio escalable en Kubernetes con Service tipo LoadBalancer (MetalLB)

**Objetivo:** Publicar servicio accesible desde red externa en entorno bare metal o Minikube + MetalLB.

Habilita MetalLB en Minikube (addon) o aplica manifiestos oficiales. Define pool de IPs:

```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: pool-local
  namespace: metallb-system
spec:
  addresses:
  - 192.168.59.240-192.168.59.250
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: adv1
  namespace: metallb-system
spec: {}
```

Service tipo LoadBalancer:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: demo-lb
spec:
  selector:
    app: demo-web
  ports:
    - port: 80
      targetPort: 80
  type: LoadBalancer
```

Tras aplicar, `kubectl get svc demo-lb` debe mostrar una IP externa del pool.

---

## 4.6 Guías rápidas de referencia

### Comandos Docker más usados

```
docker pull imagen:tag
docker images
docker run [opciones] imagen [cmd]
docker ps [-a]
docker exec -it contenedor sh|bash
docker logs -f contenedor
docker stop|start|rm contenedor
docker build -t nombre:tag .
docker tag origen destino
```

### Directivas Dockerfile de referencia rápida

```
FROM, ARG, ENV, RUN, COPY, ADD, WORKDIR, EXPOSE, USER,
ENTRYPOINT, CMD, HEALTHCHECK, VOLUME, ONBUILD, STOPSIGNAL, SHELL
```

### Claves docker-compose.yaml más comunes

```
services: {build, image, container_name, command, entrypoint, environment, env_file,
  ports, expose, volumes, networks, depends_on, restart, healthcheck, profiles, deploy}
networks: {name, driver, external, attachable, ipam}
volumes: {name, driver, external}
```

### kubectl esenciales

```
kubectl get nodes|pods|svc -A
kubectl describe pod <pod>
kubectl logs <pod> [-c contenedor]
kubectl exec -it <pod> -- sh
kubectl apply -f manifiesto.yaml
kubectl delete -f manifiesto.yaml
kubectl scale deployment <dep> --replicas=3
kubectl rollout restart deployment <dep>
```

---

## 4.7 Buenas prácticas, seguridad y optimización

* Usa imágenes base mínimas y fijas por digest.
* Ejecuta como usuario no root; asigna UID fijo.
* Usa secretos (Docker: `--env-file` o Docker Secrets/Swarm; K8s: Secret objects, External Secrets).
* Escanea imágenes regularmente (trivy, grype, snyk). Integra en CI.
* Reduce superficie: elimina herramientas de build en etapas finales.
* Registra y monitorea (Prometheus, Grafana, Loki, ELK).
* Limita recursos en Kubernetes (`requests` y `limits`).
* Usa políticas de red (NetworkPolicies) para reducir exposición Este-Oeste.

---

## 4.8 Preguntas de repaso / evaluación

1. Explica por qué los contenedores arrancan más rápido que las VMs.
2. Diferencia `COPY` vs `ADD` en un Dockerfile.
3. ¿Qué problema resuelve un Service en Kubernetes?
4. ¿Qué tipo de Service usarías para exponer una app dentro del clúster únicamente?
5. ¿Qué hace `docker compose up -d --build`?
6. ¿Por qué usarías multi-stage builds?
7. ¿Cómo harías persistentes los datos de una base en Compose?
8. Menciona dos plugins CNI.
9. ¿Qué diferencia hay entre `NodePort` y `LoadBalancer`?
10. ¿Por qué es recomendable ejecutar el proceso como usuario no root dentro del contenedor?

---

## 4.9 Anexos

### A: Instalación offline de Docker

1. Descarga paquetes RPM/DEB en máquina con Internet.
2. Transfiere vía USB.
3. Instala dependencias, luego paquetes docker-ce.
4. Copia binario docker-compose si usas legado.

### B: Registro privado (Harbor/Registry)

Ejemplo Compose mínimo para registry:

```yaml
services:
  registry:
    image: registry:2
    ports:
      - "5001:5000"
    volumes:
      - regdata:/var/lib/registry
volumes:
  regdata:
```

Push a registro privado inseguro (solo laboratorio):

```bash
# Edita /etc/docker/daemon.json
{
  "insecure-registries": ["mi-registro:5001"]
}
sudo systemctl restart docker

docker tag hola-node:1.0 mi-registro:5001/hola-node:1.0
docker push mi-registro:5001/hola-node:1.0
```

### C: Troubleshooting rápido

* Error DNS al construir imagen: revisa red host y /etc/resolv.conf dentro del contenedor.
* Permiso denegado al mapear volumen: verifica UID/GID en host vs contenedor.
* Contenedor sale inmediatamente: proceso principal terminó; usa `docker logs`.
* Imagen muy grande: revisa capas, limpia caches, multi-stage.



