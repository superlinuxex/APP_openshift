# Bike Service App (OpenShift + Quarkus + PostgreSQL)

Aplicación de microservicio para gestión de bicicletas, desarrollada con **Quarkus**, desplegada en **OpenShift**, con persistencia en **PostgreSQL**.

---

## 📅 Características
- Backend en Java con Quarkus 3+
- Conexión a base de datos PostgreSQL via ConfigMap
- Despliegue automatizado desde GitHub
- CRUD de bicicletas (endpoints REST)

---

## 🚀 Despliegue en OpenShift

### 1. Crear base de datos PostgreSQL

#### ConfigMap de credenciales:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: postgres-config
  labels:
    app: postgres
  namespace: todo-app
data:
  POSTGRES_DB: postgresdb
  POSTGRES_USER: admin
  POSTGRES_PASSWORD: s3b4st14nkrk
```

#### Despliegue de Postgres:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
  namespace: todo-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
        - name: postgres
          image: quay.io/enterprisedb/postgresql
          ports:
            - containerPort: 5432
          envFrom:
            - configMapRef:
                name: postgres-config
          volumeMounts:
            - name: postgredb
              mountPath: /var/lib/postgresql/data
              subPath: pgdata
      volumes:
        - name: postgredb
          emptyDir:
            sizeLimit: 500Mi
```

#### Service de Postgres:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: postgres
  namespace: todo-app
spec:
  type: ClusterIP
  ports:
    - port: 5432
  selector:
    app: postgres
```

### 2. Crear base de datos "pedal" y usuario "testdbuser"
### Verificar el nombre del host ejemplo
```bash
oc get svc -n <namespace>
NAME           TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
postgres       NodePort    10.217.5.222   <none>        5432:31815/TCP   12m
```


```bash
psql -h <postgres_pod_ip> -U admin -p 5432 postgresdb
CREATE USER testdbuser SUPERUSER;
ALTER USER testdbuser WITH PASSWORD 'testpassword';
CREATE DATABASE pedal;
```

### 3. Crear ConfigMap de la aplicación Bike Service
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: pedal-config-map
  namespace: todo-app
data:
  quarkus.datasource.db-kind: postgresql
  quarkus.datasource.jdbc.url: jdbc:postgresql://postgres:5432/pedal
  quarkus.datasource.username: testdbuser
  quarkus.datasource.password: testpassword
  quarkus.hibernate-orm.database.generation: drop-and-create
  quarkus.hibernate-orm.sql-load-script: import.sql
```

---

## 💾 Despliegue del microservicio Bike Service

1. Desde consola OpenShift: **Importar desde Git**
   - URL: https://github.com/redhat-developer-demos/pedal-bike-service
   - Directorio de contexto: `/`
   - Imagen de build: `openjdk-17-ubi8`
   - Nombre app: `pedal`
   - Nombre componente: `bike-service`
   - Crear Route: ✅

2. Una vez creado, ir a **Deployment -> bike-service -> Env** y cargar:
   - `envFrom`: ConfigMap `pedal-config-map`

3. Guarda y espera a que el pod se reinicie.

---

## 🔗 Endpoints REST
| Método | Ruta           | Descripción                 |
|---------|----------------|-----------------------------|
| GET     | /bikes         | Listar bicicletas           |
| POST    | /bikes         | Crear nueva bicicleta       |
| GET     | /bikes/{id}    | Obtener bicicleta por ID    |
| PUT     | /bikes/{id}    | Actualizar bicicleta por ID |
| DELETE  | /bikes/{id}    | Eliminar bicicleta por ID   |

---

## 💼 Recursos importantes
- Quarkus datasource: https://quarkus.io/guides/datasource
- GitHub repo app: https://github.com/redhat-developer-demos/pedal-bike-service

---

## 🌟 Autor
Superlinuxec ✨  
Email: superlinuxec@gmail.com  
GitHub: [superlinuxex](https://github.com/superlinuxex)

