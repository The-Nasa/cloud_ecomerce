# 📘 Guía Paso a Paso: Despliegue de Tienda Virtual en AWS EC2 (EverShop)
> **Taller:** Computación en la Nube – Despliegue de Tienda Virtual con Base de Datos en AWS  
> **Fecha límite:** 10 de noviembre de 2026  
> **Infraestructura:** AWS EC2 (Ubuntu 24.04 LTS) + Docker & Docker Compose + GitHub Actions CI/CD  
> **Servidor EC2:** `shopsphere-servidor-web` (IP Pública: `54.209.168.175`)  
> **Repositorio GitHub:** [https://github.com/The-Nasa/cloud_ecomerce.git](https://github.com/The-Nasa/cloud_ecomerce.git)  
> **Stack Tecnológico:**
> - **Plataforma E-Commerce:** EverShop (React + Node.js + GraphQL + Tailwind CSS)
> - **Base de Datos Relacional:** PostgreSQL 16
> - **Contenedores:** Docker & Docker Compose (Puertos 80 y 3000)

---

## 📑 Tabla de Contenidos
1. [Arquitectura del Sistema](#1-arquitectura-del-sistema)
2. [Checklist de Evidencias (Capturas para el Informe)](#2-checklist-de-evidencias-capturas-para-el-informe)
3. [Paso 1: Creación del Grupo de Seguridad (Security Group)](#paso-1-creación-del-grupo-de-seguridad-security-group)
4. [Paso 2: Par de Claves SSH (Key Pair)](#paso-2-par-de-claves-ssh-key-pair)
5. [Paso 3: Lanzamiento de la Instancia EC2](#paso-3-lanzamiento-de-la-instancia-ec2)
6. [Paso 4: Conexión SSH a la Instancia](#paso-4-conexión-ssh-a-la-instancia)
7. [Paso 5: Configuración del Servidor (Swap de 4 GB + Docker)](#paso-5-configuración-del-servidor-swap-de-4-gb--docker)
8. [Paso 6: Despliegue con GitHub Actions (CI/CD) & Manejo del Archivo .env](#paso-6-despliegue-con-github-actions-cicd--manejo-del-archivo-env)
9. [Paso 7: Verificación del Despliegue y Contenedores Activos](#paso-7-verificación-del-despliegue-y-contenedores-activos)
10. [Paso 8: Evidencia de la Base de Datos (PostgreSQL)](#paso-8-evidencia-de-la-base-de-datos-postgresql)
11. [Paso 9: Pruebas Funcionales en el Navegador Web](#paso-9-pruebas-funcionales-en-el-navegador-web)
12. [Paso 10: Control de Costos (Apagar o Terminar Recursos)](#paso-10-control-de-costos-apagar-o-terminar-recursos)
13. [Plantilla y Redacción para el Informe Académico](#13-plantilla-y-redacción-para-el-informe-académico)

---

## 1. Arquitectura del Sistema

```
                   INTERNET / NAVEGADOR DEL CLIENTE
                                 │
                         (Puerto 80 HTTP)
                                 ▼
                     AWS INTERNET GATEWAY
                                 │
                     SECURITY GROUP (Firewall)
               (Permite Puertos 22 SSH, 80 HTTP)
                                 │
                     INSTANCIA AWS EC2 (Ubuntu 24.04)
                     IP Pública: 54.209.168.175
┌──────────────────────────────────────────────────────────────────┐
│  DOCKER COMPOSE ORCHESTRATION                                    │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │                    EVERSHOP APP CONTAINER                  │  │
│  │          Node.js 20 + React + GraphQL + Tailwind           │  │
│  │                     (Expone Puerto 80)                     │  │
│  └─────────────────────────────┬──────────────────────────────┘  │
│                                │ (Conexión TCP :5432)            │
│                                ▼                                 │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │                 POSTGRESQL 16 CONTAINER                    │  │
│  │             (Base de Datos Relacional de la Tienda)        │  │
│  │                     (Puerto Interno 5432)                  │  │
│  └────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. Checklist de Evidencias (Capturas para el Informe)

Toma captura de pantalla a cada uno de estos momentos clave para adjuntar a tu informe:

| # | Evidencia a Capturar | Descripción de la Imagen |
|---|----------------------|---------------------------|
| 📸 1 | **Security Group** | Reglas de entrada (Inbound rules) con puertos 22, 80 y 443 abiertos. |
| 📸 2 | **Lanzamiento de EC2** | Formulario en AWS Console con tipo `t3.small` / `t3.medium` y almacenamiento 25-30 GB. |
| 📸 3 | **Instancia en Ejecución** | Panel de EC2 con la instancia en estado `Running` y la **IP pública IPv4** (`54.209.168.175`) visible. |
| 📸 4 | **Conexión SSH** | Terminal (PowerShell) mostrando la conexión exitosa y el banner de bienvenida de Ubuntu. |
| 📸 5 | **Memoria Swap** | Comando `free -h` mostrando los 4 GB de Swap activos en el servidor. |
| 📸 6 | **Docker Instalado** | Salida de los comandos `docker --version` y `docker compose version`. |
| 📸 7 | **GitHub Secrets (DevOps)** | Pantalla de GitHub **Settings > Secrets and variables > Actions** con los 4 secretos (`EC2_HOST`, `EC2_USERNAME`, `EC2_PRIVATE_KEY`, `ENV_FILE`). |
| 📸 8 | **Pipeline GitHub Actions** | Pestaña **Actions** en GitHub con el flujo completado en verde ✅ (**Success**). |
| 📸 9 | **Logs del Despliegue** | Detalle de los pasos en GitHub Actions ejecutando `docker compose up -d` remotamente en la EC2. |
| 📸 10 | **Contenedores Activos** | Salida de `docker compose ps` con los contenedores `evershop-app` y `evershop-db` en estado `Up / healthy`. |
| 📸 11 | **Base de Datos PostgreSQL** | Consulta SQL en el contenedor (`\dt` y `SELECT * FROM product LIMIT 5;`) demostrando la base de datos en AWS. |
| 📸 12 | **Tienda en el Navegador** | Página principal de EverShop con la IP `http://54.209.168.175` visible en la barra del navegador. |
| 📸 13 | **Catálogo y Carrito** | Vista de productos agregados al carrito de compras con sus cálculos de precios. |
| 📸 14 | **Panel de Administración** | Acceso a `http://54.209.168.175/admin` para gestionar productos, pedidos y clientes. |

---

## Paso 1: Creación del Grupo de Seguridad (Security Group)

Este grupo actúa como firewall virtual permitiendo el tráfico web y la administración remota.

- Reglas de entrada necesarias:
  - **SSH (22):** `0.0.0.0/0`
  - **HTTP (80):** `0.0.0.0/0`
  - **HTTPS (443):** `0.0.0.0/0`

> 📸 **Captura:** Tabla de reglas de entrada del Security Group (Evidencia 1).

---

## Paso 2: Par de Claves SSH (Key Pair)

Clave utilizada: `shopsphere-key.pem` guardada en:  
`C:\Users\ThikPad\Downloads\shopsphere-key.pem`

---

## Paso 3: Lanzamiento de la Instancia EC2

- **Nombre:** `shopsphere-servidor-web`
- **AMI:** Ubuntu Server 24.04 LTS
- **Tipo:** `t3.small` (o `t3.medium`)
- **IP Pública:** `54.209.168.175`

> 📸 **Captura:** Instancia en estado `Running` mostrando la IP pública asignada (Evidencia 2 y 3).

---

## Paso 4: Conexión SSH a la Instancia

```powershell
cd C:\Users\ThikPad\Downloads
ssh -i "shopsphere-key.pem" ubuntu@54.209.168.175
```

> 📸 **Captura:** Terminal con la conexión SSH exitosa y el banner de bienvenida de Ubuntu (Evidencia 4).

---

## Paso 5: Configuración del Servidor (Swap de 4 GB + Docker)

```bash
# 1. Configurar memoria Swap de 4 GB
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
free -h

# 2. Instalar Docker y Docker Compose
sudo apt update && sudo apt upgrade -y
sudo apt install -y docker.io docker-compose-v2 git
sudo usermod -aG docker ubuntu
newgrp docker
docker --version
docker compose version
```

> 📸 **Captura:** Comandos `free -h` y `docker --version` (Evidencias 5 y 6).

---

## Paso 6: Despliegue con GitHub Actions (CI/CD) & Manejo del Archivo .env

### 1. Configurar los 4 Secretos en tu Nuevo Repositorio
1. Abre tu nuevo repositorio en el navegador:  
   👉 [**https://github.com/The-Nasa/cloud_ecomerce**](https://github.com/The-Nasa/cloud_ecomerce)
2. Ve a **Settings** > Menú lateral izquierdo **Secrets and variables** > **Actions**.
3. Clic en **New repository secret** y crea cada uno de estos 4 secretos:

| Nombre del Secreto (Name) | Valor (Secret) |
|---|---|
| **`EC2_HOST`** | `54.209.168.175` |
| **`EC2_USERNAME`** | `ubuntu` |
| **`EC2_PRIVATE_KEY`** | Abre `C:\Users\ThikPad\Downloads\shopsphere-key.pem` con el **Bloc de notas**, copia **todo** el texto (con `BEGIN` y `END`) y pégalo. |
| **`ENV_FILE`** | Copia y pega el siguiente bloque: |

```env
DB_HOST=database
DB_PORT=5432
DB_USER=postgres
DB_PASSWORD=postgresPass2026
DB_NAME=evershop
```

> 📸 **Captura:** Pantalla de GitHub con la lista de los 4 Repository Secrets creados (Evidencia 7).

---

### 2. Subir tu Código al Nuevo Repositorio de GitHub
Abre una terminal de PowerShell en tu PC dentro de la carpeta del proyecto y ejecuta:

```powershell
git push -u origin main --force
```

---

### 3. Monitorear el Despliegue Automático
1. En tu repositorio, entra a la pestaña **Actions**:  
   👉 [https://github.com/The-Nasa/cloud_ecomerce/actions](https://github.com/The-Nasa/cloud_ecomerce/actions)
2. Verás el flujo **Deploy EverShop to AWS EC2** ejecutándose.
3. Se conectará vía SSH a `54.209.168.175`, inyectará el `.env` y levantará los contenedores `evershop-app` y `evershop-db`.
4. Al terminar (en 1 a 2 minutos), verás el checkmark verde ✅ **Success**.

> 📸 **Captura:** Flujo de GitHub Actions en verde ✅ y logs del despliegue exitoso (Evidencias 8 y 9).

---

### 💡 Alternativa Manual Directa (En la terminal SSH de tu EC2):
```bash
# Dentro de tu servidor EC2 vía SSH:
cd ~
git clone https://github.com/The-Nasa/cloud_ecomerce.git
cd cloud_ecomerce

# Crear el archivo .env:
cat << 'EOF' > .env
DB_HOST=database
DB_PORT=5432
DB_USER=postgres
DB_PASSWORD=postgresPass2026
DB_NAME=evershop
EOF

# Levantar contenedores:
docker compose pull
docker compose up -d
```

---

## Paso 7: Verificación del Despliegue y Contenedores Activos

En la terminal SSH de tu EC2, verifica el estado de los contenedores:

```bash
cd ~/cloud_ecomerce
docker compose ps
```

Deberás observar los 2 servicios en estado `Up`:
```
NAME           IMAGE                      COMMAND                  SERVICE    STATUS
evershop-app   evershop/evershop:latest   "docker-entrypoint.s…"   app        Up
evershop-db    postgres:16-alpine         "docker-entrypoint.s…"   database   Up (healthy)
```

> 📸 **Captura:** Salida de `docker compose ps` con los contenedores activos (Evidencia 10).

---

## Paso 8: Evidencia de la Base de Datos (PostgreSQL)

Para cumplir con la consigna del docente: *"Desplegar una tienda virtual con base de datos en AWS"*:

Conéctate al contenedor de PostgreSQL y realiza una consulta SQL sobre las tablas de la tienda:

```bash
docker exec -it evershop-db psql -U postgres -d evershop -c "
\dt
SELECT tablename FROM pg_tables WHERE schemaname = 'public' LIMIT 10;
"
```
Verás las tablas creadas por EverShop (`product`, `category`, `customer`, `cart`, `order`, etc.).

> 📸 **Captura:** Terminal mostrando las tablas de la base de datos PostgreSQL en AWS (Evidencia 11).

---

## Paso 9: Pruebas Funcionales en el Navegador Web

Abre cualquier navegador web e ingresa la dirección IP pública de tu servidor EC2:

👉 **`http://54.209.168.175`**

### Evidencias a documentar:
1. **Página Principal de la Tienda (Storefront):**
   - La tienda EverShop cargará directamente en el puerto 80 con un diseño profesional y responsivo.
   - La IP `54.209.168.175` debe verse en la barra de direcciones.
   > 📸 **Captura:** Landing page de EverShop (Evidencia 12).

2. **Catálogo de Productos y Carrito de Compras:**
   - Explora productos, selecciona tallas/colores y añádelos al carrito.
   - Ve al carrito para revisar subtotales e impuestos.
   > 📸 **Captura:** Carrito de compras con artículos agregados (Evidencia 13).

3. **Panel de Administración de EverShop:**
   - Entra a:  
     👉 **`http://54.209.168.175/admin`**
   - Configura o ingresa al panel de control para administrar inventario, órdenes y analíticas de ventas.
   > 📸 **Captura:** Panel de administración de EverShop (Evidencia 14).

---

## Paso 10: Control de Costos (Apagar o Terminar Recursos)

Una vez que tengas todas las capturas listas para tu informe:

- **Para pausar el servidor sin perder nada (Stop):**
  En AWS Console > Selecciona `shopsphere-servidor-web` > **Instance state** > **Stop instance**.  
  *(Así no te cobra cómputo y puedes volver a encenderlo si el profesor pide sustentación).*

- **Para eliminarlo definitivamente (Terminate):**
  En AWS Console > **Instance state** > **Terminate instance**.

---

## 13. Plantilla y Redacción para el Informe Académico

Copia esta estructura para tu informe en Word o Google Docs:

```markdown
# INFORME DE LABORATORIO: DESPLIEGUE DE TIENDA VIRTUAL EN AWS
**Asignatura:** Computación en la Nube
**Estudiante:** [Tu Nombre]
**Fecha:** Noviembre 2026

## 1. INTRODUCCIÓN
El presente informe documenta el despliegue y puesta en producción de una tienda virtual moderna (EverShop) sobre la infraestructura de Amazon Web Services (AWS). Se implementó un modelo de computación en la nube IaaS utilizando contenedores Docker orquestados con Docker Compose y un pipeline de entrega continua (CI/CD) mediante GitHub Actions.

## 2. ARQUITECTURA DE LA INFRAESTRUCTURA
- Proveedor Cloud: Amazon Web Services (AWS)
- Región: us-east-1 (N. Virginia)
- Cómputo: Amazon Elastic Compute Cloud (EC2), tipo t3.small, Ubuntu Server 24.04 LTS
- Almacenamiento: Volumen EBS gp3 de 25 GB con memoria Swap de 4 GB
- Red: Virtual Private Cloud (VPC) con Security Group (Puertos 22 SSH, 80 HTTP)
- Orquestación: Docker Compose v2
- Pipeline CI/CD: GitHub Actions con despliegue automatizado por SSH
- Componentes del Sistema:
  - Aplicación E-Commerce: EverShop (Node.js 20, React, GraphQL, Tailwind CSS) en el puerto 80.
  - Base de Datos Relacional: PostgreSQL 16 con persistencia en volumen Docker.

[Pegar aquí el Diagrama de Arquitectura]

## 3. PROCEDIMIENTO PASO A PASO (EVIDENCIAS)
- 3.1. Configuración de Seguridad en AWS (Security Group)
  [Pegar Captura 1: Inbound rules]
- 3.2. Lanzamiento y Aprovisionamiento de la Instancia EC2
  [Pegar Capturas 2 y 3: Formulario de lanzamiento e IP pública 54.209.168.175]
- 3.3. Conexión Remota y Optimización de Recursos (Memoria Swap)
  [Pegar Capturas 4 y 5: SSH y comando free -h]
- 3.4. Instalación del Motor Docker y Docker Compose
  [Pegar Captura 6: Versiones de Docker]
- 3.5. Pipeline DevOps con GitHub Actions y Gestión de Secretos (.env)
  [Pegar Capturas 7, 8 y 9: Secretos en GitHub y ejecución exitosa del pipeline]
- 3.6. Orquestación y Estado de Contenedores
  [Pegar Captura 10: Salida de docker compose ps]

## 4. VALIDACIÓN DE LA BASE DE DATOS
Demostración de la inicialización de esquemas relacionales y tablas en PostgreSQL:
[Pegar Captura 11: Consultas SQL con \dt y tablas de la base de datos en AWS]

## 5. PRUEBAS DE FUNCIONAMIENTO EN PRODUCCIÓN
Validación del aplicativo web interactivo accesible públicamente mediante la IP del servidor:
- Navegación del catálogo en vivo [Pegar Captura 12]
- Operaciones del carrito de compras [Pegar Captura 13]
- Panel de Administración y Gestión [Pegar Captura 14]

## 6. CONCLUSIONES
1. La implementación de una arquitectura contenerizada con Docker Compose permitió desplegar la tienda virtual y su base de datos PostgreSQL de forma desacoplada y reproducible en AWS EC2.
2. La integración de GitHub Actions facilitó un flujo de CI/CD moderno, donde cada cambio en el repositorio se despliega automáticamente en la nube sin intervención manual.
3. El uso de imágenes optimizadas redujo significativamente el consumo de recursos de cómputo y memoria RAM, permitiendo una ejecución fluida en una instancia económica de tipo t3.small.
```
