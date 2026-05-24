# 📦 Proyecto DevOps - Innovatech Chile

Solución de contenedorización y despliegue automatizado de una aplicación de microservicios en AWS EC2 mediante Docker, Docker Compose y GitHub Actions.

---

## 📋 Tabla de Contenidos

- [Descripción General](#descripción-general)
- [Requisitos Previos](#requisitos-previos)
- [Arquitectura](#arquitectura)
- [Estructura del Proyecto](#estructura-del-proyecto)
- [Instalación Local](#instalación-local)
- [Despliegue en AWS](#despliegue-en-aws)
- [Pipeline CI/CD](#pipeline-cicd)
- [Documentación Técnica](#documentación-técnica)
- [Troubleshooting](#troubleshooting)

---

## 📖 Descripción General

Este proyecto implementa la **contenedorización y automatización de despliegue** para una aplicación de gestión de ventas y despachos. La solución consta de:

- **Frontend**: React + Nginx (aplicación web)
- **Backend Ventas**: Spring Boot API REST
- **Backend Despacho**: Spring Boot API REST
- **Database**: MySQL 8

### Características Principales

✅ **Multi-stage builds** optimizados para reducir tamaño de imágenes  
✅ **Usuario no-root** en todos los contenedores (seguridad)  
✅ **Volúmenes nombrados** para persistencia de datos  
✅ **Health checks** integrados en cada servicio  
✅ **GitHub Actions** para CI/CD automatizado  
✅ **Variables de entorno** configurables por entorno  
✅ **Redes Docker** aisladas (frontend/backend)  

---

## 🔧 Requisitos Previos

### Local (Desarrollo)
- Docker Desktop (v20.10+)
- Docker Compose (v2.0+)
- Git
- Node.js 20+ (solo para desarrollo frontend)
- Java 17 (solo para desarrollo backend)
- Maven 3.9+ (solo para desarrollo backend)

### AWS (Despliegue)
- Cuenta AWS Academy (Learner/Educate Lab)
- 3 instancias EC2 (Amazon Linux 2023, t3.micro)
- VPC con subredes públicas/privadas
- Security Groups configurados
- Par de claves EC2 (.pem)

### GitHub
- Repositorio con acceso a Secrets
- Rama `deploy` configurada

---

## 🏗️ Arquitectura

```
┌─────────────────────────────────────────────────────────────┐
│                         INTERNET                             │
└────────────────────────┬────────────────────────────────────┘
                         │ Puerto 80 (HTTP)
                         ▼
        ┌─────────────────────────────────┐
        │    EC2 Frontend (Público)       │
        │  ┌─────────────────────────┐   │
        │  │  nginx (puerto 80)      │   │
        │  │  React App              │   │
        │  └─────────────────────────┘   │
        └──────────────┬──────────────────┘
                       │ Solicitudes HTTP (puerto 8081/8082)
                       ▼
        ┌─────────────────────────────────┐
        │   EC2 Backend (Privada)         │
        │ ┌──────────┐     ┌──────────┐  │
        │ │ ventas   │     │ despacho │  │
        │ │ (8081)   │     │ (8082)   │  │
        │ └────┬─────┘     └────┬─────┘  │
        │      └─────────┬──────┘        │
        │                ▼               │
        │      ┌──────────────────┐      │
        │      │ MySQL:3306       │      │
        │      │ (Privada)        │      │
        │      └──────────────────┘      │
        │      Volume: mysql_data       │
        └─────────────────────────────────┘
```

---

## 📁 Estructura del Proyecto

```
proyecto-devops/
├── .github/
│   └── workflows/
│       ├── deploy-frontend.yml          # Workflow CI/CD Frontend
│       ├── deploy-backend-ventas.yml    # Workflow CI/CD Backend Ventas
│       └── deploy-backend-despacho.yml  # Workflow CI/CD Backend Despacho
├── front_despacho/
│   ├── Dockerfile                       # Multi-stage build (Node + Nginx)
│   ├── src/
│   ├── package.json
│   └── ...
├── back-Ventas_SpringBoot/
│   ├── dockerfile                       # Multi-stage build (Maven + JRE)
│   └── Springboot-API-REST/
│       ├── src/
│       ├── pom.xml
│       └── ...
├── back-Despachos_SpringBoot/
│   ├── dockerfile                       # Multi-stage build (Maven + JRE)
│   └── Springboot-API-REST-DESPACHO/
│       ├── src/
│       ├── pom.xml
│       └── ...
├── docker-compose.yml                   # Stack completo local
├── README.md                            # Esta documentación
└── datos_prueba.sql                     # Datos iniciales

```

---

## 🚀 Instalación Local

### 1. Clonar el Repositorio

```bash
git clone https://github.com/bencasolocaso/proyecto-devops.git
cd proyecto-devops
git checkout deploy
```

### 2. Levantar Stack Completo

```bash
# Construir e iniciar todos los servicios
docker-compose up -d

# Ver logs en tiempo real
docker-compose logs -f

# Verificar que todos estén corriendo
docker-compose ps
```

### 3. Acceder a los Servicios

- **Frontend**: http://localhost
- **Backend Ventas**: http://localhost:8081/swagger-ui.html
- **Backend Despacho**: http://localhost:8082/swagger-ui.html
- **MySQL**: localhost:3306 (usuario: root, contraseña: root)

### 4. Insertar Datos de Prueba

```bash
# Opción 1: Ejecutar SQL directamente
docker exec mysql-db mysql -uroot -proot < datos_prueba.sql

# Opción 2: Usar herramienta como MySQL Workbench o phpMyAdmin
# Host: localhost:3306
# Usuario: root
# Contraseña: root
```

### 5. Detener Stack

```bash
docker-compose down

# Si deseas eliminar volúmenes (cuidado, perderas datos)
docker-compose down -v
```

---

## ☁️ Despliegue en AWS

### Paso 1: Preparar Instancias EC2

```bash
# En cada instancia EC2 (SSH como ec2-user)

# Actualizar sistema
sudo yum update -y
sudo yum install -y docker git

# Iniciar Docker daemon
sudo systemctl start docker
sudo systemctl enable docker

# Agregar usuario al grupo docker
sudo usermod -aG docker ec2-user

# Verificar instalación
docker --version
```

### Paso 2: Configurar Security Groups

**Frontend-SG (Pública)**:
- HTTP (80): 0.0.0.0/0
- SSH (22): [tu IP]

**Backend-SG (Privada)**:
- SSH (22): [tu IP]
- TCP 8081: frontend-sg
- TCP 8082: frontend-sg
- TCP 3306: backend-sg

**Database-SG (Privada)**:
- SSH (22): [tu IP]
- TCP 3306: backend-sg

### Paso 3: Obtener DNS Privados/Públicos

Desde AWS Console, obtén:
- **Frontend DNS Público**: ec2-XX-XXX-XX-XX.compute-1.amazonaws.com
- **Backend DNS Público**: ec2-XX-XXX-XX-XX.compute-1.amazonaws.com
- **Database DNS Privado**: ip-10-0-XXX-XXX.ec2.internal

### Paso 4: Desplegar Servicios

**En EC2 Frontend**:
```bash
ssh -i tu-clave.pem ec2-user@frontend-dns

sudo docker pull bencasolocaso/proyectosemestral-frontend:latest
sudo docker run -d \
  --name frontend-app \
  -p 80:80 \
  bencasolocaso/proyectosemestral-frontend:latest
```

**En EC2 Backend**:
```bash
ssh -i tu-clave.pem ec2-user@backend-dns

# Ventas API
sudo docker pull bencasolocaso/proyectosemestral-ventas:latest
sudo docker run -d \
  --name ventas-api \
  -e DB_ENDPOINT=ip-10-0-155-39.ec2.internal \
  -e DB_PORT=3306 \
  -e DB_NAME=ventas \
  -e DB_USERNAME=root \
  -e DB_PASSWORD=root \
  -p 8081:8080 \
  bencasolocaso/proyectosemestral-ventas:latest

# Despacho API
sudo docker pull bencasolocaso/proyectosemestral-despacho:latest
sudo docker run -d \
  --name despacho-api \
  -e DB_ENDPOINT=ip-10-0-155-39.ec2.internal \
  -e DB_PORT=3306 \
  -e DB_NAME=despacho \
  -e DB_USERNAME=root \
  -e DB_PASSWORD=root \
  -p 8082:8081 \
  bencasolocaso/proyectosemestral-despacho:latest
```

**En EC2 Database**:
```bash
ssh -i tu-clave.pem ec2-user@database-dns

sudo docker pull mysql:8
sudo docker run -d \
  --name mysql-db \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=despacho \
  -p 3306:3306 \
  -v mysql_data:/var/lib/mysql \
  mysql:8
```

---

## 🔄 Pipeline CI/CD

### Estructura del Pipeline

```
Push a rama 'deploy'
    ↓
[GitHub Actions] Detecta cambios
    ↓
1. Build: Construye imagen Docker multi-stage
2. Push: Publica en Docker Hub
3. Deploy: SSH a EC2 → detiene contenedor → inicia nuevo
    ↓
Aplicación actualizada en vivo
```

### Secrets Requeridos en GitHub

Ve a `Settings → Secrets and variables → Actions` y agrega:

```
DOCKER_USERNAME          # Tu usuario de Docker Hub
DOCKER_PASSWORD          # Token de Docker Hub
EC2_FRONTEND_HOST        # DNS público EC2 Frontend
EC2_BACKEND_HOST         # DNS público EC2 Backend
EC2_SSH_KEY              # Contenido de tu archivo .pem
DB_ENDPOINT              # DNS privado MySQL
DB_USERNAME              # Usuario MySQL
DB_PASSWORD              # Contraseña MySQL
AWS_ROLE_TO_ASSUME       # ARN del role (si usas STS)
```

### Triggers de Despliegue

- **Frontend**: cambios en `front_despacho/` + push a `deploy`
- **Backend Ventas**: cambios en `back-Ventas_SpringBoot/` + push a `deploy`
- **Backend Despacho**: cambios en `back-Despachos_SpringBoot/` + push a `deploy`

---

## 📚 Documentación Técnica

Ver archivos complementarios:
- `ARQUITECTURA.md` - Decisiones técnicas detalladas
- `CHECKLIST_EVALUACION.md` - Verificación de requisitos rúbrica
- `QUICKSTART.md` - Setup rápido local y AWS

---

## 🐛 Troubleshooting

### Frontend muestra error de conexión

**Síntoma**: `ERR_CONNECTION_REFUSED` al acceder a frontend

**Solución**:
```bash
# Verificar que Nginx está escuchando
sudo netstat -tlnp | grep :80

# Ver logs del contenedor
sudo docker logs frontend-app

# Revisar Security Group: puerto 80 abierto
aws ec2 describe-security-groups --group-ids sg-xxxxx
```

### Backend no conecta a MySQL

**Síntoma**: `java.sql.SQLException: Access denied`

**Solución**:
```bash
# Verificar que MySQL está corriendo
sudo docker ps | grep mysql

# Ver logs de MySQL
sudo docker logs mysql-db

# Probar conexión manualmente
sudo docker exec mysql-db mysql -uroot -proot -e "SELECT 1;"

# Verificar variable de entorno
sudo docker inspect ventas-api | grep DB_ENDPOINT
```

### Volumen MySQL no persiste

**Síntoma**: Datos desaparecen al reiniciar contenedor

**Solución**:
```bash
# Verificar que volumen existe
docker volume ls | grep mysql

# Inspeccionar volumen
docker volume inspect mysql_data

# Verificar permisos
sudo ls -la /var/lib/docker/volumes/mysql_data/_data/

# Recrear volumen si está corrupto
docker volume rm mysql_data
docker volume create mysql_data
```

### Pipeline GitHub Actions falla

**Síntoma**: Workflow rojo en GitHub

**Solución**:
1. Click en workflow → ver logs
2. Verificar Secrets: `Settings → Secrets and variables → Actions`
3. Verificar rama: debe ser `deploy`
4. Ver si EC2 está accesible: `ssh -i key.pem ec2-user@host`

---

**Última actualización**: Mayo 2025  
**Autores**: Benjamin Serrano - Luis Villalobos
**Estado**: ✅ Producción
