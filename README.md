# 🚀 Traccar Server — Guía Definitiva de Despliegue

<p align="center">
  <img src="https://img.shields.io/badge/Java-17%20%7C%2021-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white" alt="Java" />
  <img src="https://img.shields.io/badge/Node.js-18+-339933?style=for-the-badge&logo=nodedotjs&logoColor=white" alt="Node.js" />
  <img src="https://img.shields.io/badge/Google_Cloud-Compute_Engine-4285F4?style=for-the-badge&logo=googlecloud&logoColor=white" alt="Google Cloud" />
  <img src="https://img.shields.io/badge/Ubuntu-22.04_LTS-E95420?style=for-the-badge&logo=ubuntu&logoColor=white" alt="Ubuntu" />
  <img src="https://img.shields.io/badge/Traccar-v6.5-0288D1?style=for-the-badge&logo=traccar&logoColor=white" alt="Traccar" />
  <img src="https://img.shields.io/badge/License-Apache_2.0-009688?style=for-the-badge" alt="License" />
</p>

Guía técnica integral para la instalación, configuración y puesta en marcha de **Traccar Server** (plataforma de rastreo GPS open source de alto rendimiento). Este documento contempla dos estrategias de despliegue según el caso de uso: **Entorno de Desarrollo Local** y **Entorno de Producción en Google Cloud Platform (GCP)**.

---

## 📑 Tabla de Contenidos

- [📌 Introducción y Arquitectura](#-introducción-y-arquitectura)
- [💻 Manual 1: Despliegue Local (Entorno de Desarrollo)](#-manual-1-despliegue-local-entorno-de-desarrollo)
  - [Requisitos Previos](#requisitos-previos)
  - [Paso 1: Clonar Repositorios Oficiales](#paso-1-clonar-repositorios-oficiales)
  - [Paso 2: Compilar la Aplicación Web (Frontend)](#paso-2-compilar-la-aplicación-web-frontend)
  - [Paso 3: Compilar y Ejecutar el Servidor (Backend)](#paso-3-compilar-y-ejecutar-el-servidor-backend)
  - [Paso 4: Verificación en el Navegador](#paso-4-verificación-en-el-navegador)
- [☁️ Manual 2: Despliegue en Google Cloud (Producción)](#️-manual-2-despliegue-en-google-cloud-producción)
  - [Fase 1: Infraestructura en Google Cloud Platform (GCP)](#fase-1-infraestructura-en-google-cloud-platform-gcp)
    - [1. Creación de Instancia Compute Engine](#1-creación-de-instancia-compute-engine)
    - [2. Asignación de IP Externa Estática](#2-asignación-de-ip-externa-estática)
    - [3. Reglas de Firewall (VPC Network)](#3-reglas-de-firewall-vpc-network)
  - [Fase 2: Instalación y Configuración de Traccar Server](#fase-2-instalación-y-configuración-de-traccar-server)
    - [1. Descarga e Instalación del Paquete Precompilado](#1-descarga-e-instalación-del-paquete-precompilado)
    - [2. Configuración de Puertos y Protocolos GPS](#2-configuración-de-puertos-y-protocolos-gps)
    - [3. Habilitación y Arranque del Servicio Systemd](#3-habilitación-y-arranque-del-servicio-systemd)
  - [Fase 3: Configuración Opcional con Nginx y Certificado SSL (HTTPS)](#fase-3-configuración-opcional-con-nginx-y-certificado-ssl-https)
- [📡 Matriz de Puertos y Protocolos GPS](#-matriz-de-puertos-y-protocolos-gps)
- [🛠️ Comandos Útiles de Mantenimiento](#️-comandos-útiles-de-mantenimiento)

---

## 📌 Introducción y Arquitectura

Traccar está compuesto por dos componentes principales:
1. **Traccar Backend (Java):** Motor de socket TCP/UDP multihilo que decodifica tramas GPS de cientos de protocolos propietarios, persiste telemetría en base de datos y provee una API REST.
2. **Traccar Web (React):** Interfaz moderna para visualización en tiempo real, gestión de geocercas, reportes y dispositivos.

| Entorno | Propósito | Estrategia de Instalación |
| :--- | :--- | :--- |
| **Desarrollo (Local)** | Modificar código, customizar interfaz React o depurar protocolos. | Código fuente con Gradle y Node.js. |
| **Producción (Cloud)** | Conectar dispositivos físicos 24/7 con alta estabilidad y bajo consumo. | Binario oficial precompilado con `systemd`. |

---

## 💻 Manual 1: Despliegue Local (Entorno de Desarrollo)

Este entorno es ideal para desarrolladores que requieren modificar el código fuente de Traccar, implementar plugins, ajustar decodificadores o personalizar la interfaz web en React.

### Requisitos Previos

Antes de comenzar, asegúrate de contar con las siguientes herramientas instaladas en tu sistema:

- **Java Development Kit (JDK):** Versión 17 o 21 (LTS).
- **Node.js:** Versión 18 o superior con gestor de paquetes `npm`.
- **Git:** Para clonar los repositorios.

### Paso 1: Clonar Repositorios Oficiales

Crea un directorio de trabajo y clona tanto el núcleo del servidor como el cliente web:

```bash
# Clonar backend de Traccar
git clone https://github.com/traccar/traccar.git

# Clonar interfaz web moderna (React)
git clone https://github.com/traccar/traccar-web.git
```

### Paso 2: Compilar la Aplicación Web (Frontend)

Ingresa al repositorio del frontend, instala las dependencias de Node.js y genera la versión de producción optimizada:

```bash
cd traccar-web
npm install
npm run build
```

> **Nota:** La carpeta resultante `build/` contiene todos los activos estáticos que el servidor Java servirá en la ruta web por defecto.

### Paso 3: Compilar y Ejecutar el Servidor (Backend)

Regresa a la carpeta del servidor e inicia la aplicación mediante el wrapper de Gradle:

```bash
cd ../traccar

# En Linux / macOS:
./gradlew run

# En Windows (PowerShell / CMD):
.\gradlew.bat run
```

Gradle descargará automáticamente las dependencias, compilará el código y levantará el servidor Traccar en segundo plano.

### Paso 4: Verificación en el Navegador

Una vez inicializado el servidor, abre tu navegador web y visita:

```text
http://localhost:8082
```

- **Usuario administrador predeterminado:** `admin`
- **Contraseña predeterminada:** `admin`

---

## ☁️ Manual 2: Despliegue en Google Cloud (Producción)

Diseñado para soportar dispositivos GPS físicos en operación continua. En producción se utiliza la versión precompilada empaquetada oficialmente por Traccar para maximizar la estabilidad, la eficiencia de memoria y el rendimiento del CPU.

> [!WARNING]
> **No compilar desde código fuente en la máquina de producción:** Compilar Traccar (Java/Gradle) o el frontend en instancias con recursos ajustados (como `e2-micro`) provocará cuelgues por agotamiento de memoria RAM (OOM Killer) y consumo excesivo de CPU. Emplea siempre el instalador oficial precompilado.

---

### Fase 1: Infraestructura en Google Cloud Platform (GCP)

#### 1. Creación de Instancia Compute Engine
- **Nombre:** `traccar-server` (o de tu preferencia)
- **Tipo de Máquina:** `e2-micro` (2 vCPU, 1 GB de memoria - elegible para la capa gratuita de GCP)
- **Sistema Operativo:** `Ubuntu 22.04 LTS` (x86/64)
- **Disco de arranque:** 20 GB - 30 GB SSD estándar

#### 2. Asignación de IP Externa Estática

> [!CAUTION]
> **Requisito Crítico - IP Externa Estática:**  
> Por defecto, las instancias de GCP tienen una IP efímera que cambia si la máquina se detiene o reinicia. Los dispositivos GPS físicos se configuran apuntando a una IP pública fija. Si la IP cambia, **todos los rastreadores perderán conexión**.  
>  
> Dirígete a: **VPC network** > **IP addresses** > Selecciona la IP externa de tu máquina y cámbiala de **Ephemeral** a **Static**.

#### 3. Reglas de Firewall (VPC Network)

Crea una regla de firewall en la consola de GCP para permitir el tráfico entrante hacia la máquina virtual:

- **Nombre de regla:** `allow-traccar-ports`
- **Dirección del tráfico:** Entrada (Ingress)
- **Destino:** Instancias en la red
- **Rangos de IPv4 de origen:** `0.0.0.0/0`
- **Protocolos y puertos:**

```text
tcp: 80, 443, 8082, 5001, 5013, 5023, 5027, 5046, 5055, 5093
udp: 5001, 5013, 5023, 5027, 5046, 5055, 5093
```

---

### Fase 2: Instalación y Configuración de Traccar Server

Conéctate a tu máquina virtual mediante SSH y ejecuta los siguientes pasos:

#### 1. Descarga e Instalación del Paquete Precompilado

Actualiza los paquetes del sistema, descarga la versión oficial v6.5 e instálala:

```bash
# Actualizar repositorios e instalar descompresor
sudo apt update && sudo apt install -y unzip wget

# Descargar instalador de Traccar para Linux 64 bits (v6.5)
wget https://github.com/traccar/traccar/releases/download/v6.5/traccar-linux-64-6.5.zip

# Descomprimir paquete
unzip traccar-linux-64-6.5.zip

# Ejecutar el instalador
sudo ./traccar.run
```

El instalador desplegará los binarios en `/opt/traccar/` y registrará el servicio `traccar` en `systemd`.

#### 2. Configuración de Puertos y Protocolos GPS

Edita el archivo de configuración principal de Traccar:

```bash
sudo nano /opt/traccar/traccar.xml
```

Localiza la etiqueta de cierre `</properties>` e inserta inmediatamente antes los protocolos y puertos asignados:

```xml
    <!-- ============================================== -->
    <!--     HABILITACIÓN DE PROTOCOLOS Y PUERTOS       -->
    <!-- ============================================== -->
    <entry key='protocols.enable'>osmand,http,gt06,h02,sinotrack,gps103,teltonika,ruptela</entry>
    
    <!-- Puertos específicos para cada protocolo -->
    <entry key='gps103.port'>5001</entry>
    <entry key='h02.port'>5013</entry>
    <entry key='gt06.port'>5023</entry>
    <entry key='teltonika.port'>5027</entry>
    <entry key='ruptela.port'>5046</entry>
    <entry key='sinotrack.port'>5093</entry>
```

> **Consejo:** Guarda los cambios en `nano` presionando `Ctrl + O`, confirma con `Enter` y sal con `Ctrl + X`.

#### 3. Habilitación y Arranque del Servicio Systemd

Configura Traccar para que inicie automáticamente junto con el sistema operativo y arranca el servicio:

```bash
# Habilitar servicio para auto-arranque en reinicios
sudo systemctl enable traccar

# Iniciar el servicio Traccar
sudo systemctl start traccar

# Comprobar estado del servicio
sudo systemctl status traccar
```

---

### Fase 3: Configuración Opcional con Nginx y Certificado SSL (HTTPS)

Para exponer el servicio en un puerto estándar seguro (`https://tu-dominio.com`), configura **Nginx** como Reverse Proxy y **Certbot** para obtener certificados gratuitos de Let's Encrypt:

```bash
# Instalar Nginx y Certbot
sudo apt install -y nginx certbot python3-certbot-nginx

# Crear configuración de proxy inverso
sudo nano /etc/nginx/sites-available/traccar
```

Pega la siguiente configuración reemplazando `gps.tudominio.com` por tu dominio o subdominio real:

```nginx
server {
    server_name gps.tudominio.com;

    location / {
        proxy_pass http://127.0.0.1:8082;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Habilita el sitio y genera el certificado SSL automático:

```bash
# Activar sitio en Nginx
sudo ln -s /etc/nginx/sites-available/traccar /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl restart nginx

# Solicitar e instalar certificado SSL gratuito
sudo certbot --nginx -d gps.tudominio.com
```

---

## 📡 Matriz de Puertos y Protocolos GPS

Configura tus equipos GPS apuntando a la **IP Estática de tu servidor GCP** con su respectivo puerto según el protocolo:

| Protocolo | Puerto | Dispositivos / Fabricantes Comunes | Transporte |
| :--- | :---: | :--- | :--- |
| **GPS103** | `5001` | Coban, GPS103-A/B, TK103, TK102 | TCP / UDP |
| **H02** | `5013` | Rastreadores genéricos chinos, H02, GT02 | TCP / UDP |
| **GT06** | `5023` | Concox, Accuragps, WeTrack, GT06N | TCP / UDP |
| **Teltonika** | `5027` | Teltonika FMB920, FMB120, FMC130 | TCP / UDP |
| **Ruptela** | `5046` | Ruptela Eco4, Pro4, HCV | TCP / UDP |
| **OsmAnd / HTTP** | `5055` | Traccar Client App (Android / iOS) | TCP / HTTP |
| **SinoTrack** | `5093` | SinoTrack ST-901, ST-902, ST-906 | TCP / UDP |
| **Consola Web** | `8082` | Interfaz gráfica de administración | TCP / HTTP |

---

## 🛠️ Comandos Útiles de Mantenimiento

A continuación, una recopilación de comandos indispensables para operar Traccar en Linux:

```bash
# Ver el log en tiempo real de Traccar
sudo tail -f /opt/traccar/logs/tracker-server.log

# Ver logs del servicio systemd
sudo journalctl -u traccar -f

# Reiniciar el servicio tras cambios en traccar.xml
sudo systemctl restart traccar

# Detener el servicio
sudo systemctl stop traccar

# Verificar si los puertos están escuchando correctamente
sudo ss -tulpn | grep -E '8082|5001|5013|5023|5027|5046|5055|5093'
```

---

<p align="center">
  Desarrollado con ❤️ para despliegues profesionales de telemetría y rastreo GPS.
</p>
