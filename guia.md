# Despliegue de DVWA en Kubernetes 
---
Este documento describe el proceso de despliegue de la aplicación DVWA (Damn Vulnerable Web Application) y su base de datos asociada (MySQL/MariaDB) en un clúster de Kubernetes. El despliegue se hará en el namespace training. Tendrá tres componentes principales:

- **Deployment para DVWA:** Contendrá la aplicación web DVWA, que necesitará conectarse a la base de datos.
- **Deployment para MySQL/MariaDB:** Contendrá la base de datos y deberá estar configurado para almacenamiento persistente.
- **Services:** Facilitarán la comunicación entre los pods de DVWA y la base de datos, y proporcionarán acceso al servicio de DVWA si es necesario.
## Deployment DVWA
En esta sección
``` el codigo ome ```

