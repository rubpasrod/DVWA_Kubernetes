# Despliegue de DVWA en Kubernetes 
---
Este documento describe el proceso de despliegue de la aplicación DVWA (Damn Vulnerable Web Application) y su base de datos en un clúster de Kubernetes. El despliegue se hará en el namespace training. 

## Pasos previos
Vamos a suponer que ya se ha instalado [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/), vamos a posicionarnos en el namespace que usaremos (training).
```kubectl config set-context --current --namespace=training```
Si no está creado el namespace.
```kubectl create namespace training```
## Cómo probarlo
Para poder probarlo vamos a necesitar algo que nos permita usar kubernetes de forma local. Hemos usado Minikube. [Enlace a la guía de instalación y uso](https://minikube.sigs.k8s.io/docs/start/?arch=%2Fwindows%2Fx86-64%2Fstable%2F.exe+download)
```minikube start```
Si queremos añadir nuestros yaml:
```kubectl apply -f /home/ruta/manifests```
Para probar si la aplicación está funcionando ejecutamos el siguiente comando (cambiar el nombre del servicio):
```kubectl port-forward service/dvwa-service 8080:80 -n training```

## Troubleshooting
En esta sección veremos algunos fallos encontrados y su solución. 

### Comprobar conexión con la DB
Pensaba que no estaba enlazando bien la BD. Aunque se creara no era capaz de verla. Esta es la forma para comprobarlo
Modificar contexto.
```kubectl config set-context --current --namespace=training```
Entrar en el pod (ajustar nombre).
```kubectl exec -it dvwa-55fc4975cd-8l446 -- bash ```
Una vez dentro del pod, entrar en SQL.
```root@xxxx:/# mysql ```
Mostrar bases de datos.
```SHOW DATABASES;```
Usar la que queremos.
```USE DVWA;```
Verificar que se han creado las tablas.
```SHOW TABLES;```
Debería mostrar algo así:
```
+----------------+
| Tables_in_dvwa |
+----------------+
| guestbook      |
| users          |
+----------------+
```

Si no se ve, probar a darle a ```Create / Reset Database``` en setup.php o reiniciar el pod.