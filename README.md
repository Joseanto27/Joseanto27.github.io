## Proyecto Web con Docker Compose

### ❶ Introducción:
Este repositorio documenta el proceso completo de configuración de un servidor local en Ubuntu mediante contenedores Docker,incluyendo la gestión de contenedores, persistencia de datos y resolución de incidencias de red y correo.

### ❷ Instalación y Gestión de Contenedores
El primer paso consistió en levantar un stack de servicios mediante Docker Compose.

Stack de Servicios:
Portainer: Instalado para la gestión visual de contenedores, imágenes y volúmenes.

Nginx: Servidor web configurado para escuchar en el puerto 89.

PHP 8-FPM: Procesador de scripts PHP con la extensión mysqli para conectar con la base de datos.

MySQL 8.0: Motor de base de datos para el proyecto picassgti.

phpMyAdmin: Interfaz gráfica para la gestión de MySQL en el puerto 8089.

### ❸ Especificaciones de los Archivos de Configuración
📄 Docker-compose.yaml
Es el orquestador del proyecto. A diferencia de un despliegue manual, este archivo automatiza la creación de la red appnet y la interconexión de los 4 servicios (Nginx, PHP, MySQL, phpMyAdmin).

Persistencia: Se incluyó el volumen ./mysql_data:/var/lib/mysql para asegurar que los datos no se pierdan al borrar el contenedor.

Dependencias: Cada servicio está configurado para operar dentro de una red aislada, mejorando la seguridad del stack.

📄 Default.conf (Nginx) vs Standalone
Comparado con una instalación Standalone (Nginx instalado directamente en el SO), existen diferencias críticas:

Directivas de PHP: En modo standalone, se usa 127.0.0.1:9000 o un socket de Unix. En Docker, usamos el nombre del servicio app:9000 gracias al DNS interno de Docker.

Rutas de archivos: Las rutas deben coincidir con el volumen montado dentro del contenedor (/var/www/picassgti/), no con la ruta física de la máquina host.

### ❹ Incidencias Técnicas y Soluciones
🔧 Gestión de Rutas y Volúmenes
Problema: Nginx fallaba al intentar montar un archivo como si fuera un directorio.

Solución: Se corrigió la ruta absoluta en el archivo YAML para apuntar específicamente al archivo de configuración del host virtual.

🔧 Errores de Sintaxis YAML
Problema: Errores de "mapping" que impedían levantar los servicios.

Solución: Estandarización del sangrado (espacios) para cumplir con el formato estricto de YAML.

🔧 Fallos de Autenticación en Servicios Externos
Problema: SMTP Error: Could not authenticate al enviar correos.

Solución: Implementación de "Contraseñas de aplicación" de Google para saltar la seguridad 2FA en entornos de desarrollo.
