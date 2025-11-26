
#  **Parte 2 – Despliegue de Servicios en Kubernetes (MicroK8s, Deployments, Services, Ingress, HPA & StatefulSets)**

Esta sección documenta **cómo se desplegaron todos los componentes del Proyecto Integrador** dentro del clúster Kubernetes de MicroK8s.
Incluye backend, frontend, bases de datos, cache, configuración, secrets, almacenamiento persistente y balanceadores de carga internos.

---

# **1. Creación del Namespace del Proyecto**

Para aislar los recursos del proyecto se creó un namespace dedicado:

```bash
microk8s kubectl apply -f 00-namespace/namespace.yaml
```

Luego se confirmó:

```bash
microk8s kubectl get ns
```

Output esperado:

```
proyecto-integrador   Active
```

---

# **2. ConfigMaps (Variables que no requieren secreto)**

Se configuraron ConfigMaps para el backend y el frontend:

### Backend (`api-config.yaml`)

```yaml
DB_HOST: postgres
DB_PORT: "5432"
REDIS_HOST: redis
REDIS_PORT: "6379"
```

### Frontend (`frontend-config.yaml`)

```yaml
API_URL: "http://cotel.test/api"
```

Aplicación:

```bash
microk8s kubectl apply -f 01-configmaps/
```

---

# **3. Secrets (Contraseñas y credenciales sensibles)**

Se gestionan como Kubernetes Secrets:

* `POSTGRES_PASSWORD`
* `REDIS_PASSWORD` (si aplica)
* credenciales internas

Aplicación:

```bash
microk8s kubectl apply -f 02-secrets/
```

Validación:

```bash
microk8s kubectl get secrets -n proyecto-integrador
```

---

# **4. Almacenamiento Persistente**

Para PostgreSQL y Redis se configuraron **PersistentVolumes** y **PersistentVolumeClaims**:

```bash
microk8s kubectl apply -f 03-storage/
```

Esto garantiza que:

* los datos de PostgreSQL se mantengan entre reinicios
* Redis pueda persistir si está configurado en modo AOF

---

# **5. Bases de Datos (StatefulSets + Headless Services)**

La carpeta `04-databases` despliega:

###  PostgreSQL (StatefulSet)

Incluye:

* headless service (`postgres-headless`)
* statefulset
* PVC

Se crea con:

```bash
microk8s kubectl apply -f 04-databases/postgres-*.yaml
```

### 👉 Redis (Deployment o StatefulSet)

```bash
microk8s kubectl apply -f 04-databases/redis-*.yaml
```

Verificación:

```bash
microk8s kubectl get pods -n proyecto-integrador
```

---

# **6. Despliegue del Backend (API – Spring Boot)**

El backend utiliza:

* Deployment
* Service (ClusterIP)
* ConfigMap + Secrets
* Imagen v2 desde el registry local (`localhost:32000/api:v2`)

Comando aplicado:

```bash
microk8s kubectl apply -f 05-backend/
```

Verificación:

```bash
microk8s kubectl get pods -n proyecto-integrador
microk8s kubectl logs deployment/api -n proyecto-integrador
```

---

# **7. Despliegue del Frontend (Angular v17 con Proxy BFF)**

El frontend también utiliza:

* Deployment
* Service
* ConfigMap
* Imagen `localhost:32000/frontend:v2`

Se aplica:

```bash
microk8s kubectl apply -f 06-frontend/
```

Verificación:

```bash
microk8s kubectl get pods -n proyecto-integrador
```

---

# **8. Ingress (Exposición del sistema por dominio interno)**

Se habilitó Ingress en MicroK8s:

```bash
microk8s enable ingress
```

Se añadió en `/etc/hosts`:

```
127.0.0.1  cotel.test
```

Y se creó el archivo:

```bash
microk8s kubectl apply -f 07-ingress/ingress.yaml
```

El Ingress enruta:

| Path   | Servicio destino | Puerto |
| ------ | ---------------- | ------ |
| `/`    | frontend-service | 80     |
| `/api` | api-service      | 8080   |

Prueba final:

 [http://cotel.test](http://cotel.test)

---

# **9. Horizontal Pod Autoscaler (HPA)**

El HPA escala el backend automáticamente según CPU.

```bash
microk8s kubectl autoscale deployment api --cpu-percent=50 --min=1 --max=5 -n proyecto-integrador
```

Verificación:

```bash
microk8s kubectl get hpa -n proyecto-integrador
```

---

# **10. Resultado final de la Parte 2**

Al finalizar esta etapa, el cluster queda con:

* Frontend + Backend completamente funcionales
* PostgreSQL y Redis persistentes
* Variables configuradas (ConfigMap + Secrets)
* Imágenes v2 cargadas en el registry local
* Balanceo y ruteo interno con Ingress
* HPA para autoescalado del backend
* Dominio personalizado `cotel.test` para acceso al sistema

El sistema queda accesible y operativo en:

```
http://cotel.test
```
