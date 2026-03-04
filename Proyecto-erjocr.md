# Informe Técnico: Despliegue de Servidor de Mensajería Instantánea con Ejabberd

**Proyecto:** Infraestructura de Comunicaciones XMPP  
**Tecnología:** Ejabberd 24.x sobre Ubuntu (Máquina Virtual)  
**Alcance:** Instalación, configuración, integración de clientes y resolución de incidencias  
**Estado:** ✅ Completado y operativo

---

## Tabla de Contenidos

1. [Introducción y Objetivos](#introducción-y-objetivos)
2. [Arquitectura del Sistema](#arquitectura-del-sistema)
3. [Requisitos del Sistema](#requisitos-del-sistema)
4. [Instalación y Configuración Inicial](#instalación-y-configuración-inicial)
5. [Configuración de Red con IP Estática](#configuración-de-red-con-ip-estática)
6. [Configuración del Cliente en Windows 11](#configuración-del-cliente-en-windows-11)
7. [Resolución de Incidencias](#resolución-de-incidencias)
8. [Verificación y Pruebas](#verificación-y-pruebas)
9. [Conclusiones y Buenas Prácticas](#conclusiones-y-buenas-prácticas)

---

## Introducción y Objetivos

Este informe documenta de forma exhaustiva el proceso de despliegue de un servidor de mensajería instantánea basado en el protocolo **XMPP (Extensible Messaging and Presence Protocol)**, utilizando el software **ejabberd** sobre una máquina virtual Ubuntu. El objetivo principal es proporcionar una infraestructura de comunicación interna fiable, multiplataforma y de código abierto.

### ¿Qué es Ejabberd?

**ejabberd** es uno de los servidores XMPP más robustos y utilizados a nivel mundial. Está desarrollado en **Erlang/OTP**, lo que le confiere alta tolerancia a fallos, escalabilidad horizontal y capacidad para gestionar grandes volúmenes de conexiones simultáneas. Es la solución elegida por organizaciones que requieren comunicaciones en tiempo real sin depender de servicios de terceros.

### Objetivos del Proyecto

- Desplegar un servidor XMPP funcional sobre una máquina virtual en red local.
- Registrar y gestionar usuarios de forma centralizada.
- Permitir la comunicación entre la VM (Linux) y el host físico (Windows 11) mediante adaptador puente.
- Documentar el proceso de instalación para facilitar su replicabilidad.
- Identificar y solventar las incidencias surgidas durante la puesta en marcha.

---

## Arquitectura del Sistema

El entorno de este proyecto se basa en una **máquina virtual Ubuntu** ejecutada sobre un **host físico Windows 11** mediante VirtualBox con **adaptador puente (Bridged Adapter)**. Este modo de red hace que la VM se comporte como un equipo más dentro de la red local, obteniendo su propia IP directamente del router, lo que permite la comunicación directa entre la VM y el host sin necesidad de redirección de puertos ni NAT.
```
┌──────────────────────────────────────────────────────────────────┐
│                     HOST FÍSICO — Windows 11                     │
│                        192.168.109.X                             │
│                                                                  │
│   Cliente Pidgin (usuario: punky)                                │
│   Archivo hosts: 192.168.109.48 → erjoay.local                  │
│                                                                  │
│   ┌────────────────────────────────────────────────────────┐     │
│   │          VM — Ubuntu Desktop (VirtualBox)              │     │
│   │          Adaptador de Red: PUENTE (Bridged)            │     │
│   │                                                        │     │
│   │   ejabberd — Dominio: erjoay.local                     │     │
│   │   IP estática: 192.168.109.48                          │     │
│   │   Usuario admin: admin@erjoay.local                    │     │
│   │   Puerto XMPP: 5222                                    │     │
│   └────────────────────────────────────────────────────────┘     │
│                          │                                       │
│              Adaptador puente (tráfico directo a LAN)            │
│                          │                                       │
└──────────────────────────┼───────────────────────────────────────┘
                           │
                    ┌──────┴──────┐
                    │   ROUTER    │
                    │ 192.168.109.1│
                    └─────────────┘
```

### ¿Por qué Adaptador Puente?

| Modo de Red | Comportamiento | Válido para este proyecto |
|:---|:---|:---:|
| **NAT** | La VM comparte la IP del host. El host no puede conectarse a la VM por nombre o IP directamente. | ❌ |
| **Red interna** | Aislada del host y del exterior. Sin comunicación con Windows. | ❌ |
| **Adaptador puente** | La VM obtiene su propia IP en la misma red que el host. Comunicación directa y bidireccional. | ✅ |

| Componente | Detalle |
|:---|:---|
| **Host físico** | Windows 11 — ejecuta VirtualBox y el cliente Pidgin |
| **Máquina virtual** | Ubuntu Desktop — ejecuta ejabberd con adaptador puente |
| **Dominio XMPP** | `erjoay.local` |
| **IP del servidor (VM)** | `192.168.109.48` (estática) |
| **Protocolo** | XMPP (puerto 5222) |
| **Usuarios registrados** | `admin@erjoay.local`, `punky@erjoay.local` |

---

## Requisitos del Sistema

| Componente | Especificación |
|:---|:---|
| **Host físico** | Windows 11 con VirtualBox instalado |
| **Máquina virtual** | Ubuntu Desktop / Server (20.04 LTS o superior) |
| **Adaptador de red VM** | Modo **Puente (Bridged Adapter)** obligatorio |
| **Cliente XMPP** | Pidgin instalado en el host Windows 11 |
| **Configuración de red** | IP estática asignada a la VM |

### Puertos de Red Necesarios

| Puerto | Protocolo | Uso |
|:---|:---|:---|
| `5222` | TCP | Conexiones de clientes XMPP (STARTTLS) |
| `5223` | TCP | Conexiones de clientes XMPP (SSL Legacy) |
| `5269` | TCP | Comunicación entre servidores (S2S) |
| `5280` | TCP | Panel de administración web (HTTP) |
| `5443` | TCP | Panel de administración web (HTTPS) |

> ⚠️ Al usar adaptador puente, la VM está expuesta directamente en la red local. En producción se recomienda configurar `ufw` dentro de la VM para limitar el acceso solo a los puertos necesarios.

---

## Instalación y Configuración Inicial

### 1.1 Instalación del Servidor Ejabberd

Se actualiza el índice de paquetes del sistema para garantizar que se instalen las versiones más recientes disponibles en los repositorios oficiales de Ubuntu:
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install ejabberd -y
```

Para verificar que el servicio arrancó correctamente:
```bash
sudo systemctl status ejabberd
```

La salida esperada debe mostrar el estado `active (running)`, confirmando que el proceso Erlang/OTP está en ejecución.

---

### 1.2 Configuración del Dominio Virtual

El archivo central de configuración se encuentra en `/etc/ejabberd/ejabberd.yml`. Este fichero utiliza sintaxis **YAML** y controla todos los aspectos del servidor.
```bash
sudo nano /etc/ejabberd/ejabberd.yml
```

Los parámetros clave modificados fueron:
```yaml
# Dominio virtual que gestiona este servidor
hosts:
  - erjoay.local

# Definición del administrador del sistema
acl:
  admin:
    user:
      - admin@erjoay.local

# Interfaz de escucha para clientes
listen:
  -
    port: 5222
    ip: "::"
    module: ejabberd_c2s
    starttls_required: false
```

> 💡 El parámetro `starttls_required: false` se estableció para simplificar la configuración en el entorno de laboratorio. En producción se recomienda forzar cifrado TLS.

Tras modificar el archivo, se reinicia el servicio:
```bash
sudo systemctl restart ejabberd
```

---

### 1.3 Registro de Usuarios

La herramienta **`ejabberdctl`** permite administrar el servidor directamente desde la terminal. Se utilizó para registrar los dos usuarios del sistema:
```bash
sudo ejabberdctl register admin erjoay.local jose1234
sudo ejabberdctl register punky erjoay.local jose1234
```

Para verificar que los usuarios se han registrado correctamente:
```bash
sudo ejabberdctl registered_users erjoay.local
```

La salida esperada:
```
admin
punky
```

---

## Configuración de Red con IP Estática

### 2.1 Justificación

Al trabajar con una VM en modo adaptador puente, la dirección IP la asigna el servidor DHCP del router de la red local. Si esta IP cambiara entre reinicios, el host Windows 11 perdería la conectividad con el servidor ejabberd y la entrada del archivo `hosts` quedaría desactualizada.

Para garantizar la estabilidad del servicio, se asignó una **IP estática a la interfaz de red de la VM** mediante **Netplan**, el sistema de gestión de red por defecto en Ubuntu desde la versión 17.10. Es importante que esta IP esté fuera del rango DHCP del router para evitar conflictos de direcciones.

### 2.2 Configuración del Archivo Netplan
```bash
sudo nano /etc/netplan/01-network-manager-all.yaml
```
```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    ens33:                        # Nombre de la interfaz (puede variar, verificar con: ip link show)
      dhcp4: no
      addresses:
        - 192.168.109.48/24       # IP estática de la VM en la red local
      gateway4: 192.168.109.1     # Puerta de enlace del router
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
```

> ⚠️ El nombre de interfaz (`ens33`) puede variar según el hipervisor y la configuración de VirtualBox. Verificar con `ip link show` antes de editar. En VirtualBox suele ser `enp0s3` o `ens33`.

### 2.3 Aplicación de la Configuración
```bash
sudo netplan apply
```

Para confirmar que la IP estática se aplicó correctamente:
```bash
ip addr show
```

Desde el host Windows 11, se puede verificar la conectividad con la VM ejecutando en cmd:
```cmd
ping 192.168.109.48
```

---

## Configuración del Cliente en Windows 11

### 3.1 Resolución de Nombres Local (Archivo Hosts)

Dado que `erjoay.local` es un dominio privado no registrado en ningún DNS público, el host Windows 11 (que actúa como cliente XMPP) necesita saber qué IP corresponde a ese nombre. Esto se consigue editando el archivo `hosts` del sistema, que actúa como una tabla DNS local con prioridad sobre cualquier servidor DNS externo.

El archivo se encuentra en:
```
C:\Windows\System32\drivers\etc\hosts
```

Se añadió la siguiente línea al final del archivo, apuntando al la IP estática de la VM:
```
192.168.109.48   erjoay.local
```

> ⚠️ La edición requiere abrir el editor de texto con **"Ejecutar como Administrador"**. Sin permisos elevados, los cambios no se guardan y el fallo es silencioso, lo que puede llevar a confusión al depurar problemas de conectividad.

### 3.2 Instalación de Pidgin

**Pidgin** es un cliente de mensajería multiplataforma compatible con XMPP, instalado en el **host físico Windows 11** para comunicarse con el servidor ejabberd alojado en la VM. Se instaló mediante **winget**:
```bash
winget install Pidgin.Pidgin
```

### 3.3 Configuración de la Cuenta XMPP en Pidgin

**Pestaña "Básico":**

| Campo | Valor |
|:---|:---|
| **Protocolo** | XMPP |
| **Nombre de usuario** | `punky` |
| **Dominio** | `erjoay.local` |
| **Contraseña** | `jose1234` |

**Pestaña "Avanzado":**

| Campo | Valor |
|:---|:---|
| **Seguridad de la conexión** | Usar cifrado si está disponible |
| **Puerto** | `5222` |
| **Conectar con el servidor** | `192.168.109.48` |

> 💡 Especificar la IP directa de la VM en "Conectar con el servidor" es especialmente importante en entornos con adaptador puente, ya que garantiza que Pidgin se dirija exactamente a la VM y no intente resolver el dominio por DNS externo, lo que fallaría al tratarse de un dominio `.local`.

---

## Resolución de Incidencias

### Incidencia 1 — Error `ERR_EMPTY_RESPONSE` en el Panel de Administración

**Síntoma:** Al acceder a `http://erjoay.local:5280/admin` desde el navegador del host Windows 11, la página no cargaba y devolvía `ERR_EMPTY_RESPONSE`.

**Causa:** Tras cambiar la configuración de IP y dominio en la VM, quedaron procesos "zombi" de la máquina virtual Erlang (`beam.smp`) bloqueando los puertos de escucha e impidiendo que el nuevo proceso de ejabberd arrancara correctamente.

**Diagnóstico:**
```bash
ps aux | grep beam
sudo lsof -i :5280
sudo lsof -i :5222
```

**Solución:**
```bash
sudo pkill -9 beam
sudo pkill -9 epmd
sudo rm -f /var/lib/ejabberd/.erlang.cookie
sudo systemctl start ejabberd
```

---

### Incidencia 2 — Error `No such file or directory`

**Síntoma:** Al intentar visualizar el archivo `hosts` de la VM con `ls`, el sistema devolvía un error.

**Causa:** El comando `ls` se utilizó incorrectamente sobre un archivo en lugar de un directorio.

**Solución:**
```bash
cat /etc/hosts       # Para visualizar
sudo nano /etc/hosts # Para editar
```

---

### Incidencia 3 — Bloqueo del Servicio con `systemctl`

**Síntoma:** Tras modificar `ejabberd.yml` en la VM, el servicio se quedaba bloqueado indefinidamente al reiniciar.

**Causa:** Error de sintaxis o indentación en el archivo YAML. Los ficheros YAML son extremadamente sensibles al espaciado.

**Diagnóstico:** Se lanzó el servidor en modo interactivo para ver el error exacto:
```bash
sudo ejabberdctl live
```

**Solución:** Se corrigió el archivo respetando la indentación de 2 espacios por nivel, sin tabulaciones. Para validar la sintaxis antes de reiniciar:
```bash
python3 -c "import yaml; yaml.safe_load(open('/etc/ejabberd/ejabberd.yml'))"
```

---

### Incidencia 4 — Fallo de Resolución DNS en el Cliente Windows

**Síntoma:** Pidgin en el host Windows 11 no podía conectarse a la VM. El intento fallaba con `Host not found`.

**Causa:** El archivo `hosts` de Windows fue editado sin permisos de administrador. El editor no advirtió del error al guardar, dando la falsa impresión de que los cambios se persistieron. Al no existir `erjoay.local` en ningún DNS externo y no estar tampoco en el `hosts` local, la resolución fallaba completamente.

**Solución:**
1. Abrir el Bloc de notas con **"Ejecutar como administrador"**.
2. Abrir `C:\Windows\System32\drivers\etc\hosts`.
3. Añadir al final: `192.168.109.48   erjoay.local`
4. Guardar y verificar desde cmd:
```cmd
ping erjoay.local
```

---

### Resumen de Incidencias

| # | Incidencia | Causa Raíz | Solución |
|:---:|:---|:---|:---|
| 1 | `ERR_EMPTY_RESPONSE` | Procesos Erlang zombi bloqueando puertos en la VM | `pkill -9 beam` + reinicio limpio |
| 2 | `No such file or directory` | Uso incorrecto del comando `ls` | Sustituir por `cat` o `nano` |
| 3 | Bloqueo de `systemctl` | Error de sintaxis YAML en `ejabberd.yml` | Depurar con `ejabberdctl live` y corregir |
| 4 | Fallo DNS en Windows | Archivo `hosts` editado sin permisos admin | Editar como Administrador y verificar con `ping` |

---

## Verificación y Pruebas

### En la VM (Servidor)
```bash
sudo systemctl status ejabberd
sudo ejabberdctl registered_users erjoay.local
sudo ejabberdctl connected_users
sudo ejabberdctl stats users
```

### Desde el Host Windows 11 (Cliente)
```cmd
:: Verificar que Windows alcanza la VM
ping 192.168.109.48

:: Verificar resolución del dominio local
ping erjoay.local

:: Verificar que el puerto XMPP está accesible
telnet 192.168.109.48 5222
```

Se verificó el envío y recepción de mensajes entre `admin@erjoay.local` y `punky@erjoay.local`, confirmando la comunicación bidireccional en tiempo real entre la VM Ubuntu y el host Windows 11.

---

## Conclusiones y Buenas Prácticas

El despliegue del servidor ejabberd ha sido completado con éxito. El uso del **adaptador puente** en VirtualBox fue clave para que el host Windows 11 pudiera comunicarse directamente con la VM Ubuntu como si fueran dos equipos independientes en la misma red, sin necesidad de redireccionamiento de puertos ni configuraciones NAT adicionales.

Las principales lecciones aprendidas son: la gestión de procesos Erlang requiere atención especial ante reinicios; la sintaxis YAML es crítica y el modo `live` es la mejor herramienta para depurar errores de configuración; y los permisos del sistema operativo son una fuente frecuente de errores silenciosos, especialmente en Windows al editar archivos del sistema.

### Recomendaciones para Entornos de Producción

| Área | Recomendación |
|:---|:---|
| **Virtualización** | En producción, preferir una instalación bare-metal o un servidor dedicado en lugar de una VM de escritorio |
| **Seguridad** | Habilitar TLS/SSL con certificados válidos (Let's Encrypt o CA interna) |
| **Contraseñas** | Utilizar contraseñas fuertes y únicas por usuario |
| **Backups** | Copias periódicas de `/etc/ejabberd/` y `/var/lib/ejabberd/` |
| **Monitorización** | Integrar con Prometheus/Grafana vía los endpoints de ejabberd |
| **Firewall** | Configurar `ufw` dentro de la VM exponiendo únicamente los puertos necesarios |
| **DNS** | Desplegar un servidor DNS interno (p.ej. dnsmasq) en lugar de editar `hosts` manualmente en cada cliente |

---

*Documento generado como parte de la documentación técnica del proyecto de infraestructura de comunicaciones XMPP.*
