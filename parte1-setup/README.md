
# **Parte 1 – Setup del Ambiente (MicroK8s + Addons + Configuración v2.0)**

Esta sección describe la instalación, configuración inicial y habilitación de los componentes necesarios para ejecutar el Proyecto Final utilizando **MicroK8s**, **registry local**, **addons de Kubernetes**, y la preparación del entorno para desplegar imágenes personalizadas versión **v2.0**.

---

## 1. Instalación de MicroK8s

MicroK8s se instaló utilizando Snap:

```bash
sudo snap install microk8s --classic
```

Se añadió el usuario actual al grupo `microk8s`:

```bash
sudo usermod -aG microk8s $USER
sudo chown -f -R $USER ~/.kube
```

Luego, se verificó el estado:

```bash
microk8s status --wait-ready
```

---

## 2. Activación de Addons esenciales

Los addons requeridos para Kubernetes se habilitaron así:

```bash
microk8s enable dns
microk8s enable ingress
microk8s enable storage
microk8s enable registry
microk8s enable metrics-server
```

 **registry** es fundamental porque alojará nuestras imágenes `v2.0` internas.

---

## 3. Verificar funcionamiento del registro local (registry)

Se comprobó que el registry interno estaba corriendo:

```bash
microk8s kubectl get pods -n container-registry
```

Y su puerto:

```bash
microk8s status | grep registry
# -> registry: enabled (localhost:32000)
```

---

## 4. Prueba de conexión al registry

Listado de imágenes dentro del registry:

```bash
curl http://localhost:32000/v2/_catalog
```

Resultado esperado:

```json
{"repositories":[]}
```

(Al inicio está vacío, es normal.)

---

## 5. Construcción de imágenes Docker v2.0 (API y Frontend)

### Backend (API)

Desde la raíz del proyecto:

```bash
docker build -t localhost:32000/api:v2 -f Dockerfile .
docker push localhost:32000/api:v2
```

### Frontend (Angular)

Desde la carpeta `frontend/`:

```bash
docker build -t localhost:32000/frontend:v2 -f Dockerfile .
docker push localhost:32000/frontend:v2
```

---

## 6. Verificación de las imágenes cargadas

Se confirmó que ambas imágenes estaban dentro del registry:

```bash
microk8s ctr images ls | grep api
microk8s ctr images ls | grep frontend
```

Ejemplo de salida:

```
docker.io/library/api:v2
docker.io/library/frontend:v2
```

---

## 7. Estructura inicial del proyecto K8s (v2.0)

Al finalizar la Parte 1, el directorio `k8s/` quedó organizado así:

```
k8s/
├── 00-namespace/
├── 01-configmaps/
├── 02-secrets/
├── 03-storage/
├── 04-databases/
├── 05-backend/
└── 06-frontend/
```

Cada carpeta contiene archivos YAML para configurar todos los componentes del proyecto.

---

## 8. Resultado de la Parte 1

Al finalizar esta etapa:

* MicroK8s quedó instalado y operando.
* Registry local funcionando en `localhost:32000`.
* Addons de DNS, Storage, Metrics y Ingress habilitados.
* Imágenes **v2.0 del backend y frontend** construidas y subidas exitosamente al registry.
* Ambiente preparado para iniciar los despliegues de Kubernetes en la Parte 2.

---


