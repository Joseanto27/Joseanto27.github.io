## Proyecto Web con Docker Compose

### ❶ Introducción:
En este repositorio os enseño cómo he montado paso a paso mi propio servidor en Ubuntu usando Docker. He configurado todo para que los contenedores funcionen juntos y, en este caso, me he asegurado de que los datos no se borren al apagar el PC. También explico cómo he solucionado los fallos que he tenido a la hora de configurar la red y el correo.

### ❷ Instalación y Gestión de Contenedores
Lo primero que he hecho es preparar un grupo de programas para que trabajen en equipo, usando un solo archivo (el Docker Compose).

### Servicios utilizados:
**Portainer:** Portainer lo he instalado para poder ver y controlar todos los contenedores de forma visual, sin tener que usar tanto la terminal.

**Nginx:** Es el servidor que recibe las visitas. En este caso, lo he configurado para que funcione a través del puerto 89.

**PHP 8-FPM:** Es el motor que hace que la web funcione. Le he añadido una extensión llamada mysqli para que pueda hablar con la base de datos.

**MySQL 8.0:** Aquí es donde se guarda toda la información de mi proyecto picassgti.

**phpMyAdmin:** Es una web que he instalado para poder entrar a la base de datos y ver las tablas de forma fácil por el puerto 8089.

### ❸ Archivos de Configuración
### Docker-compose.yaml:

Este es el archivo que manda en todo. En vez de ir instalando programa por programa a mano, este archivo lo hace todo solo crea una red propia llamada appnet y conecta los 4 programas para que se entiendan entre ellos.

**Persistencia:** He añadido una línea llamada ./mysql_data:/var/lib/mysql. En este caso, sirve para crear una carpeta en mi ordenador y que los datos de la base de datos se queden guardados aunque borre el contenedor.

**Dependencias:** He configurado cada programa para que funcione dentro de una red cerrada, algo que es fundamental a la hora de mejorar la seguridad del sistema.

### Instalación normal vs Docker

Hay varias diferencias importantes entre usar Docker y tener Nginx instalado directamente en el ordenador:

**Directivas de PHP:** Si no usara Docker, tendría que usar una dirección IP local. En este caso es mucho más fácil: solo pongo el nombre del servicio, app:9000, y Docker ya sabe a dónde tiene que ir.

**Rutas de archivos:** A la hora de configurar las rutas, no uso las de mi ordenador real, sino las de "dentro" del contenedor (/var/www/picassgti/).

### ❹ Incidencias Técnicas y Soluciones
### Gestión de Rutas y Volúmenes

**Problema:** Nginx no arrancaba porque se pensaba que un archivo de configuración era en realidad una carpeta.

**Solución:** En este caso, lo que hice fue poner la ruta completa y exacta del archivo en el Docker Compose para que no hubiera confusiones.

### Errores de Sintaxis YAML

**Problema**: Me salían errores de "mapping" y no me dejaba arrancar nada.

**Solución:** El problema eran los espacios. A la hora de escribir estos archivos, los espacios son muy importantes, así que repasé todo el documento para que el sangrado fuera perfecto.

### Fallos de Autenticación en Servicios Externos

**Problema:** Al enviar correos desde mi web, me salía un error de SMTP Error Could not authenticate.

**Solución:** En este caso, Google me bloqueaba por seguridad. Lo arreglé creando una "Contraseña de aplicación" especial de 16 dígitos en mi cuenta de Gmail.
