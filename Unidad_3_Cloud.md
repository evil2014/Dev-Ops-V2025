## 3.1 Aplicaciones con contenedores: desarrollo y despliegue en la nube

### 3.1.1 Ventajas de usar contenedores en la nube

**Teoría:**

* Portabilidad: los contenedores pueden ejecutarse en cualquier entorno con Docker o compatible.
* Aislamiento: cada contenedor tiene su propio entorno, reduciendo conflictos.
* Escalabilidad: los contenedores son ideales para microservicios y pueden escalar rápidamente.
* Velocidad: inician más rápido que máquinas virtuales.

**Práctica:**
Crear un contenedor Docker con una app simple:

1. Instalar Docker: [https://docs.docker.com/get-docker/](https://docs.docker.com/get-docker/)
2. Crear archivo `app.py`:

```python
from flask import Flask
app = Flask(__name__)
@app.route('/')
def hello():
    return "Hola desde un contenedor!"
```

3. Crear `Dockerfile`:

```Dockerfile
FROM python:3.9
WORKDIR /app
COPY app.py ./
RUN pip install flask
CMD ["python", "app.py"]
```

4. Construir la imagen:

```bash
docker build -t app-contenedor .
```

**Explicación:** `build` crea una imagen a partir del Dockerfile, `-t` asigna un nombre.

5. Ejecutar el contenedor:

```bash
docker run -p 5000:5000 app-contenedor
```

**Explicación:** `-p` mapea el puerto del contenedor al host. Accede en navegador a `localhost:5000`

---

### 3.1.2 Despliegue de aplicaciones usando contenedores

**Teoría:**

* Los contenedores pueden ejecutarse en instancias de nube pública como EC2 o GCE.
* Es posible automatizar el despliegue en entornos productivos.

**Práctica:**
Desplegar en una instancia EC2:

1. Crear instancia EC2 (Amazon) o similar.
2. Conectarse vía SSH:

```bash
ssh -i llave.pem ubuntu@<IP>
```

3. Instalar Docker en la instancia:

```bash
sudo apt update
sudo apt install docker.io -y
```

4. Subir el `Dockerfile` y `app.py`, o clonarlos de GitHub.
5. Repetir el build y run:

```bash
sudo docker build -t app-contenedor .
sudo docker run -p 80:5000 app-contenedor
```

6. Abrir el puerto 80 en el firewall (grupo de seguridad EC2).

---

### 3.1.3 Herramientas para la gestión de contenedores en la nube (ECS, GKE, AKS)

**Teoría:**

* ECS: Servicio de AWS para correr contenedores en clústeres.
* GKE: Kubernetes administrado por Google.
* AKS: Kubernetes administrado por Azure.

**Práctica:**
Desplegar en GKE:

1. Crear proyecto en Google Cloud y habilitar GKE.
2. Crear clúster.
3. Configurar acceso:

```bash
gcloud container clusters get-credentials nombre-cluster --zone us-central1-a
```

4. Crear archivo `deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app
  template:
    metadata:
      labels:
        app: app
    spec:
      containers:
      - name: app
        image: <tu-imagen>
        ports:
        - containerPort: 5000
```

5. Desplegar:

```bash
kubectl apply -f deployment.yaml
```

6. Exponer servicio:

```bash
kubectl expose deployment app --type=LoadBalancer --port=80 --target-port=5000
kubectl get svc
```

---

### 3.2 Introducción a los proveedores de nube (AWS, Azure, GCP)

#### 3.2.1 Comparación

**Teoría:**

* AWS: más servicios y madurez.
* Azure: ideal para empresas Microsoft.
* GCP: más fuerte en inteligencia artificial y datos.

#### 3.2.2 Servicios fundamentales

| Proveedor | Computación  | Almacenamiento |
| --------- | ------------ | -------------- |
| AWS       | EC2          | S3             |
| Azure     | Azure VM     | Blob Storage   |
| GCP       | Compute Eng. | Cloud Storage  |

**Práctica:**

1. Crear cuenta gratuita.
2. Lanzar VM (EC2 o similar).
3. Subir archivo a un bucket:

```bash
# AWS
aws s3 cp archivo.txt s3://mi-bucket/

# Azure
az storage blob upload --container-name mi-bucket --name archivo.txt --file archivo.txt

# GCP
gsutil cp archivo.txt gs://mi-bucket/
```

---

### 3.3 Gestión de servicios en la nube (IaaS, PaaS, SaaS)

**Teoría:**

* IaaS: infraestructura (ej. EC2, VMs).
* PaaS: plataforma lista para usar (Heroku, App Engine).
* SaaS: software como servicio (Google Drive, Gmail).

**Práctica:**
Clasificar servicios:

* EC2: IaaS
* App Engine: PaaS
* Google Docs: SaaS

Caso de uso:

* Web app empresarial: usar App Engine (PaaS) y Cloud SQL (PaaS).

---

### 3.4 Herramientas de despliegue en la nube (IaC)

#### 3.4.1 Terraform y CloudFormation

**Teoría:**

* Terraform: multiplataforma.
* CloudFormation: nativo de AWS.

**Práctica con Terraform:**

1. Instalar Terraform.
2. Crear `main.tf`:

```hcl
provider "aws" {
  region = "us-east-1"
}
resource "aws_instance" "ejemplo" {
  ami = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
}
```

3. Ejecutar:

```bash
terraform init
terraform plan
terraform apply
```

#### 3.4.2 Automatización

**Práctica:**

1. Crear repositorio con `.tf`.
2. Crear `.github/workflows/deploy.yml`:

```yaml
name: Deploy
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - name: Terraform
      run: |
        terraform init
        terraform plan
        terraform apply -auto-approve
```

3. Hacer commit y push para probar el pipeline.
