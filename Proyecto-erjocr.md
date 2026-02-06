# 🚀 Guía de Configuración y Gestión de Proxmox VE

Este repositorio contiene la documentación técnica completa para la implementación, administración y configuración de red en **Proxmox Virtual Environment**.

---

## 1. Configuración Inicial de Proxmox

### 💻 Requisitos de Hardware y Software
Para un rendimiento óptimo del hipervisor, se deben considerar los siguientes parámetros:

| Componente | Requisito Mínimo | Recomendado |
| :--- | :--- | :--- |
| **CPU** | 64-bit (Intel VT/AMD-V) | Intel Xeon / AMD EPYC |
| **RAM** | 2 GB + consumo de VMs | 8 GB o más (ECC) |
| **Disco** | 20 GB (SO + ISOs) | SSD/NVMe de alto rendimiento |
| **Red** | 100 Mbps | 1 Gbps o 10 Gbps |

### 🌐 Requisitos de Red
Proxmox requiere una **IP estática** obligatoria para garantizar que el servidor sea siempre accesible en la infraestructura:
* **IP:** Dirección única asignada al nodo dentro del rango de nuestra red local.
* **Máscara de subred:** Define el alcance y tamaño de la red (ej. 255.255.255.0).
* **Gateway:** La dirección IP del router que permite la salida a Internet.

---

## 2. Gestión de Redes y Almacenamiento

### ¿Qué es el `vmbr0`?
El `vmbr0` es un **Linux Bridge** (puente virtual) fundamental en la arquitectura de Proxmox. Funciona como un **switch virtual** interno.



* **Función principal:** Actúa como puente entre la tarjeta de red física (**NIC**) del servidor y las interfaces virtuales de las máquinas.
* **Objetivo:** Permite que las Máquinas Virtuales (VMs) y cualquier Router virtualizado se comuniquen entre sí y con el mundo exterior utilizando una única conexión física.

---

## 3. Contenedores y Estructura de Archivos

### 📦 LXC vs. Docker
Es importante diferenciar las dos tecnologías de contenedores principales:

> **LXC (Linux Containers):** Se comporta como una "máquina ligera". Tiene su propio sistema de archivos, gestión de usuarios y se comporta casi como una VM completa pero compartiendo el kernel. Proxmox lo gestiona de forma **nativa**.
>  
> **Docker:** Está diseñado específicamente para microservicios. Su objetivo es ejecutar una sola aplicación o proceso de forma aislada.

### 📂 Ubicación de la Información en el Sistema
Para administrar Proxmox desde la consola, es vital conocer estas rutas:
* `/etc/pve/`: Contiene los archivos de configuración. Están sincronizados entre todos los nodos si hay un clúster (sistema *pmxcfs*).
* `/var/lib/vz/`: Es el directorio por defecto para almacenar archivos ISO, plantillas de contenedores y copias de seguridad.

---

## 4. Glosario de Términos

| Término | Definición |
| :--- | :--- |
| **Datacenter** | El nivel jerárquico más alto; permite gestionar múltiples nodos desde una sola interfaz. |
| **Summary** | Panel visual que muestra gráficos de consumo de CPU, RAM y Red en tiempo real. |
| **Shell** | Terminal de línea de comandos integrada directamente en el navegador web. |
| **Node pve** | Se refiere al nombre físico o identificador de cada servidor individual. |
| **LVM** | *Logical Volume Manager*: Gestor de volúmenes que permite redimensionar discos de forma flexible. |
| **LVM-Thin** | Variante de LVM que solo consume espacio real en el disco a medida que la VM escribe datos. |
| **ZFS** | Sistema de archivos avanzado con autorreparación, compresión y soporte nativo para RAID. |

---

## 5. 🌐 Configuración de la Red Interna y NAT

Para que una VM cliente sea accesible desde el exterior cuando está en una red privada, configuramos un **router virtual** con dos interfaces:

1.  **Interface interna:** Conexión privada entre el router y la VM cliente.
2.  **Interface externa:** Conexión del router hacia la red física o Internet.

### 🖧 Configuración de la VM Cliente (Netplan)
En sistemas Ubuntu/Debian modernos, usamos Netplan para configurar la red. La VM cliente solicita su IP automáticamente:

```yaml
network:
  version: 2
  ethernets:
    ens18:
      dhcp4: true
