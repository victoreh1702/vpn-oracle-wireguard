# 🛡️ Proyecto: Despliegue de Servidor VPN (WireGuard) en OCI

Documentación técnica integral del despliegue de una red privada virtual sobre la infraestructura gratuita de Oracle Cloud. Este proyecto ha sido desarrollado como una aplicación práctica de ingeniería telemática para la gestión de tráfico, cifrado y enrutamiento, con el objetivo de eludir el filtrado de paquetes (ISP) y garantizar una conexión segura y de alto rendimiento.

## 🏗️ Arquitectura de la Infraestructura

* **Proveedor Cloud:** Oracle Cloud Infrastructure (OCI - Free Tier).
* **Instancia:** Máquina virtual ARM (Ampere A1 Compute, 4 OCPU, 24GB RAM).
* **Ubicación del Datacenter:** París, Francia.
* **Sistema Operativo:** Ubuntu Server 22.04 LTS.
* **Protocolo de Túnel:** WireGuard (Capa 3, conectividad UDP).
* **Dispositivo Cliente:** Xiaomi TV (Android TV).

---

## ⚙️ Guía de Despliegue Paso a Paso

### Fase 1: Creación de Cuenta y Aprovisionamiento en Oracle OCI

1. **Registro en Oracle Cloud:**
   * Acceder a [Oracle Cloud Free Tier](https://www.oracle.com/cloud/free/) y completar el registro.
   * *Nota:* Se requiere una tarjeta de crédito para verificación de identidad, pero los recursos "Always Free" no generan cargos.
2. **Creación de la Instancia de Computación:**
   * Navegar a **Compute** > **Instances** y hacer clic en **Create Instance**.
   * **Imagen y forma:** Seleccionar la imagen de **Ubuntu** y cambiar la forma (Shape) a **Ampere (ARM)**. Asignar los recursos máximos gratuitos (4 OCPU, 24GB RAM).
   * **Claves SSH:** En el apartado "Add SSH keys", seleccionar "Generate a key pair for me" y descargar la **clave privada** (`clave.key`). *Paso crítico para el acceso posterior.*
   * Clic en **Create** y esperar a que el estado cambie a "Running". Anotar la IP Pública asignada.

### Fase 2: Configuración de Reglas de Red (VCN)

Para que el servidor VPN reciba conexiones, es necesario abrir el puerto en el cortafuegos perimetral de Oracle.
1. En los detalles de la instancia, hacer clic en la red virtual (Subnet) asociada.
2. Entrar en la **Security List** por defecto (Default Security List).
3. Añadir una **Ingress Rule** (Regla de entrada):
   * **Source CIDR:** `0.0.0.0/0` (Permitir tráfico desde cualquier IP global).
   * **IP Protocol:** `UDP` (WireGuard funciona sobre UDP para minimizar la latencia).
   * **Destination Port Range:** `51820`.

### Fase 3: Preparación del Entorno Local (Windows) y Conexión SSH

Para que el cliente SSH de Windows permita la conexión, el archivo de la clave privada no puede tener permisos abiertos a otros usuarios.
1. Abrir PowerShell en el directorio donde se guardó `clave.key`.
2. Restringir permisos con `icacls`:
   ```powershell
   icacls clave.key /inheritance:r
   icacls clave.key /grant:r "%USERNAME%:(R)"
   ```
3. Conectar al servidor de Oracle en París:
   ```powershell
   ssh -i .\clave.key ubuntu@<IP_PUBLICA_ORACLE>
   ```

### Fase 4: Instalación y Configuración del Servidor WireGuard

Dentro de la terminal de Ubuntu:
1. Elevar privilegios y descargar el script automatizado de instalación de WireGuard:
   ```bash
   wget https://git.io/wireguard -O wireguard-install.sh && sudo bash wireguard-install.sh
   ```
2. Durante el asistente de instalación:
   * **Puerto:** Confirmar el puerto `51820`.
   * **Nombre del cliente:** Asignar un nombre identificativo (ej. `xiaomi-tv`).
   * **DNS:** Seleccionar un proveedor (ej. 1.1.1.1 o Google).
3. El script configurará automáticamente las reglas de enrutamiento (`iptables`), las claves criptográficas y levantará la interfaz virtual `wg0`.

### Fase 5: Extracción de Credenciales

1. Leer el contenido del archivo de configuración generado para el cliente:
   ```bash
   sudo cat /root/xiaomi-tv.conf
   ```
2. En el ordenador local, crear un archivo nuevo llamado `vpn-futbol.conf` y pegar el contenido extraído (bloques `[Interface]` y `[Peer]`). 
   * *Precaución:* Asegurarse de que el bloc de notas no añada la extensión oculta `.txt`.

### Fase 6: Despliegue en el Cliente Final (Xiaomi TV)

1. **Transferencia de archivos:** Pasar el archivo `vpn-futbol.conf` a la memoria interna del Xiaomi TV (vía USB o aplicación de transferencia de archivos por WiFi).
2. **Permisos de Android TV:** Ir a Ajustes > Aplicaciones > WireGuard > Permisos y conceder acceso al almacenamiento.
3. **Importación:** Abrir la aplicación de WireGuard, seleccionar el botón "+" e importar desde archivo. Seleccionar `vpn-futbol.conf`.
4. Activar el interruptor de la conexión. Todo el tráfico del dispositivo será ahora enrutado y cifrado hacia la instancia en París.

---

## 🔒 Control de Versiones (Git)

Este proyecto utiliza un archivo `.gitignore` para prevenir la exposición pública de material criptográfico. Los siguientes archivos están excluidos del repositorio:
```text
*.key
*.pem
*.conf
```
