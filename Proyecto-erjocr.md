# Guía de Configuración y Gestión de Proxmox VE

Este repositorio contiene la documentación técnica para la implementación de **Proxmox Virtual Environment**.

---

## 1. Configuración de Proxmox

### 💻 Requisitos de Hardware y Software
| Componente | Requisito Mínimo | Recomendado |
| :--- | :--- | :--- |
| **CPU** | 64-bit (Intel VT/AMD-V) | Intel Xeon / AMD EPYC |
| **RAM** | 2 GB + consumo de VMs | 8 GB o más (ECC) |
| **Disco** | 20 GB (SO + ISOs) | SSD/NVMe de alto rendimiento |
| **Red** | 100 Mbps | 1 Gbps o 10 Gbps |

### 🌐 Requisitos de Red
Proxmox requiere una **IP estática** para garantizar la accesibilidad permanente:
* **IP:** Dirección única dentro de nuestro rango de red.
* **Máscara de subred:** Define el tamaño y alcance de la red.
* **Gateway:** IP del router para salida a internet.

---

## 2. Gestión de Redes y Almacenamiento

### ¿Qué es el `vmbr0`?
Es un **Linux Bridge** (puente virtual) que actúa como un **switch virtual** interno.
* **Función:** Se asocia a la tarjeta de red física (**NIC**).
* **Objetivo:** Permite que las VMs y el Router instalado se comuniquen entre sí y con el exterior usando una sola conexión física.

---

## 3. Contenedores y Almacenamiento

### 📦 LXC vs. Docker
> **LXC (Linux Containers):** Se comporta como una "máquina ligera". Tiene su propio sistema de archivos y gestión de usuarios. Proxmox lo gestiona de forma **nativa**.
> 
> **Docker:** Diseñado para microservicios (ejecutar una sola aplicación o proceso).

### 📂 Ubicación de la Información
* `/etc/pve/`: Configuración sincronizada entre nodos (sistema *pmxcfs*).
* `/var/lib/vz/`: Almacenamiento de archivos ISO y plantillas.

---

## 4. 📖 Glosario de Términos

| Término | Definición |
| :--- | :--- |
| **Datacenter** | Nivel más alto de jerarquía para gestionar múltiples nodos. |
| **Summary** | Panel con gráficos de CPU, RAM y Red en tiempo real. |
| **Shell** | Terminal de comandos integrada en el navegador. |
| **Node pve** | Nombre físico del servidor individual. |
| **LVM** | Gestor de volúmenes lógicos para discos flexibles. |
| **LVM-Thin** | Variante que solo ocupa espacio real a medida que la VM escribe datos. |
| **ZFS** | Sistema de archivos avanzado con protección de datos y RAID nativo. |
