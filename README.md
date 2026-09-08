# 🛡️ Servidor VPN Personal (WireGuard) en Oracle Cloud

Este repositorio documenta el proceso de despliegue de un servidor VPN personal utilizando **WireGuard** sobre la infraestructura gratuita de **Oracle Cloud (Free Tier)**. 

El objetivo principal del proyecto es eludir el filtrado de paquetes y las restricciones de red impuestas por los ISP locales (bloqueos de tráfico de streaming y P2P), asegurando una conexión cifrada, privada y de alto rendimiento.

## 🏗️ Arquitectura e Infraestructura

* **Proveedor Cloud:** Oracle Cloud Infrastructure (OCI).
* **Instancia:** Máquina virtual ARM (Ampere A1 Compute).
* **Ubicación:** París, Francia.
* **Sistema Operativo:** Ubuntu Server.
* **Protocolo VPN:** WireGuard (Capa 3, conexión a través de UDP).
* **Cliente Final:** Android TV (Xiaomi TV) operando como *peer*.

## ⚙️ Proceso de Despliegue

### 1. Configuración de Red en OCI (VCN)
Para permitir el tráfico del túnel, se modificaron las Listas de Seguridad (Security Lists) de la Red Virtual en la Nube de Oracle:
* **Ingress Rule:** Apertura del puerto `51820` (Protocolo UDP) para cualquier origen (`0.0.0.0/0`).

### 2. Acceso Seguro por SSH
El acceso a la instancia se realiza mediante claves criptográficas ED25519. En entornos Windows, fue necesario asegurar los permisos del archivo de clave privada mediante `icacls` para cumplir con los estándares del demonio SSH:
```powershell
icacls clave.key /inheritance:r
icacls clave.key /grant:r "%USERNAME%:(R)"
```

### 3. Instalación del Servidor WireGuard
Se optó por WireGuard debido a su ligereza, integración directa en el kernel de Linux y alta eficiencia energética y de procesamiento en arquitecturas ARM.
El enrutamiento, la creación de la interfaz virtual (`wg0`) y la generación de pares de claves se automatizó mediante script estándar.

### 4. Configuración del Cliente
Se extrajo el archivo de configuración `.conf` del servidor (conteniendo las claves públicas/privadas y el *Endpoint*) y se importó en el cliente de Android TV, estableciendo el túnel cifrado persistente.

## ⚠️ Advertencia de Seguridad
Los archivos de claves privadas (`.key`, `.pem`) y las configuraciones de los clientes (`.conf`) están excluidos de este repositorio por motivos de seguridad.
