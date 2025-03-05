# Despliegue de DVWA en Kubernetes 
---
Este documento describe el proceso de despliegue de la aplicación DVWA (Damn Vulnerable Web Application) y su base de datos en un clúster de Kubernetes. El despliegue se hará en el namespace training. 
## Estructura del Despliegue
El despliegue se divide en los siguientes recursos:
- **ConfigMap y Secret**: Configuración y credenciales de acceso
- **PersistentVolumeClaim (PVC)**: Almacenamiento para MySQL
- **MySQL Deployment y Service**
- **DVWA Deployment y Service**
- **NetworkPolicy**: Permisos de acceso entre DVWA y MySQL
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

### Verificar los Pods

```kubectl get pods -n training```

### Verificar los Services

```kubectl get svc -n training```

### Verificar los Logs

Si hay errores, se pueden revisar los logs de los pods:
```
kubectl logs -n training -l app=mysql
kubectl logs -n training -l app=dvwa
```

## Resultados de LOGS
En esta sección veremos los diferentes logs de los análisis.
### Kubesec

```
[
  {
    "object": "StatefulSet/mysql.training",
    "valid": true,
    "fileName": "manifests/mysql-statefulset.yaml",
    "message": "Passed with a score of 2 points",
    "score": 2,
    "scoring": {
      "passed": [
        {
          "id": "VolumeClaimAccessModeReadWriteOnce",
          "selector": ".spec .volumeClaimTemplates[] .spec .accessModes | index(\"ReadWriteOnce\")",
          "reason": "Setting the access mode of ReadWriteOnce on volumeClaimTemplates (if any exist) allows only one node to mount the persistentVolume",
          "points": 1
        },
        {
          "id": "VolumeClaimRequestsStorage",
          "selector": ".spec .volumeClaimTemplates[] .spec .resources .requests .storage",
          "reason": "Setting a storage request on volumeClaimTemplates (if any exist) allows for the StatefulSet's PVCs to be bound to appropriately sized PVs.",
          "points": 1
        }
      ],
      "advise": [
        {
          "id": "ApparmorAny",
          "selector": ".metadata .annotations .\"container.apparmor.security.beta.kubernetes.io/nginx\"",
          "reason": "Well defined AppArmor policies may provide greater protection from unknown threats. WARNING: NOT PRODUCTION READY",
          "points": 3
        },
        {
          "id": "ServiceAccountName",
          "selector": ".spec .serviceAccountName",
          "reason": "Service accounts restrict Kubernetes API access and should be configured with least privilege",
          "points": 3
        },
        {
          "id": "SeccompAny",
          "selector": ".metadata .annotations .\"container.seccomp.security.alpha.kubernetes.io/pod\"",
          "reason": "Seccomp profiles set minimum privilege and secure against unknown threats",
          "points": 1
        },
        {
          "id": "AutomountServiceAccountToken",
          "selector": ".spec .automountServiceAccountToken == false",
          "reason": "Disabling the automounting of Service Account Token reduces the attack surface of the API server",
          "points": 1
        },
        {
          "id": "RunAsGroup",
          "selector": ".spec, .spec.containers[] | .securityContext .runAsGroup -gt 10000",
          "reason": "Run as a high-UID group to avoid conflicts with the host's groups",
          "points": 1
        },
        {
          "id": "RunAsNonRoot",
          "selector": ".spec, .spec.containers[] | .securityContext .runAsNonRoot == true",
          "reason": "Force the running image to run as a non-root user to ensure least privilege",
          "points": 1
        },
        {
          "id": "RunAsUser",
          "selector": ".spec, .spec.containers[] | .securityContext .runAsUser -gt 10000",
          "reason": "Run as a high-UID user to avoid conflicts with the host's users",
          "points": 1
        },
        {
          "id": "LimitsCPU",
          "selector": "containers[] .resources .limits .cpu",
          "reason": "Enforcing CPU limits prevents DOS via resource exhaustion",
          "points": 1
        },
        {
          "id": "LimitsMemory",
          "selector": "containers[] .resources .limits .memory",
          "reason": "Enforcing memory limits prevents DOS via resource exhaustion",
          "points": 1
        },
        {
          "id": "RequestsCPU",
          "selector": "containers[] .resources .requests .cpu",
          "reason": "Enforcing CPU requests aids a fair balancing of resources across the cluster",
          "points": 1
        },
        {
          "id": "RequestsMemory",
          "selector": "containers[] .resources .requests .memory",
          "reason": "Enforcing memory requests aids a fair balancing of resources across the cluster",
          "points": 1
        },
        {
          "id": "CapDropAny",
          "selector": "containers[] .securityContext .capabilities .drop",
          "reason": "Reducing kernel capabilities available to a container limits its attack surface",
          "points": 1
        },
        {
          "id": "CapDropAll",
          "selector": "containers[] .securityContext .capabilities .drop | index(\"ALL\")",
          "reason": "Drop all capabilities and add only those required to reduce syscall attack surface",
          "points": 1
        },
        {
          "id": "ReadOnlyRootFilesystem",
          "selector": "containers[] .securityContext .readOnlyRootFilesystem == true",
          "reason": "An immutable root filesystem can prevent malicious binaries being added to PATH and increase attack cost",
          "points": 1
        }
      ]
    }
  }
]
[
  {
    "object": "Deployment/dvwa.training",
    "valid": true,
    "fileName": "manifests/dvwa-deployment.yaml",
    "message": "Passed with a score of 0 points",
    "score": 0,
    "scoring": {
      "advise": [
        {
          "id": "ApparmorAny",
          "selector": ".metadata .annotations .\"container.apparmor.security.beta.kubernetes.io/nginx\"",
          "reason": "Well defined AppArmor policies may provide greater protection from unknown threats. WARNING: NOT PRODUCTION READY",
          "points": 3
        },
        {
          "id": "ServiceAccountName",
          "selector": ".spec .serviceAccountName",
          "reason": "Service accounts restrict Kubernetes API access and should be configured with least privilege",
          "points": 3
        },
        {
          "id": "SeccompAny",
          "selector": ".metadata .annotations .\"container.seccomp.security.alpha.kubernetes.io/pod\"",
          "reason": "Seccomp profiles set minimum privilege and secure against unknown threats",
          "points": 1
        },
        {
          "id": "AutomountServiceAccountToken",
          "selector": ".spec .automountServiceAccountToken == false",
          "reason": "Disabling the automounting of Service Account Token reduces the attack surface of the API server",
          "points": 1
        },
        {
          "id": "RunAsGroup",
          "selector": ".spec, .spec.containers[] | .securityContext .runAsGroup -gt 10000",
          "reason": "Run as a high-UID group to avoid conflicts with the host's groups",
          "points": 1
        },
        {
          "id": "RunAsNonRoot",
          "selector": ".spec, .spec.containers[] | .securityContext .runAsNonRoot == true",
          "reason": "Force the running image to run as a non-root user to ensure least privilege",
          "points": 1
        },
        {
          "id": "RunAsUser",
          "selector": ".spec, .spec.containers[] | .securityContext .runAsUser -gt 10000",
          "reason": "Run as a high-UID user to avoid conflicts with the host's users",
          "points": 1
        },
        {
          "id": "LimitsCPU",
          "selector": "containers[] .resources .limits .cpu",
          "reason": "Enforcing CPU limits prevents DOS via resource exhaustion",
          "points": 1
        },
        {
          "id": "LimitsMemory",
          "selector": "containers[] .resources .limits .memory",
          "reason": "Enforcing memory limits prevents DOS via resource exhaustion",
          "points": 1
        },
        {
          "id": "RequestsCPU",
          "selector": "containers[] .resources .requests .cpu",
          "reason": "Enforcing CPU requests aids a fair balancing of resources across the cluster",
          "points": 1
        },
        {
          "id": "RequestsMemory",
          "selector": "containers[] .resources .requests .memory",
          "reason": "Enforcing memory requests aids a fair balancing of resources across the cluster",
          "points": 1
        },
        {
          "id": "CapDropAny",
          "selector": "containers[] .securityContext .capabilities .drop",
          "reason": "Reducing kernel capabilities available to a container limits its attack surface",
          "points": 1
        },
        {
          "id": "CapDropAll",
          "selector": "containers[] .securityContext .capabilities .drop | index(\"ALL\")",
          "reason": "Drop all capabilities and add only those required to reduce syscall attack surface",
          "points": 1
        },
        {
          "id": "ReadOnlyRootFilesystem",
          "selector": "containers[] .securityContext .readOnlyRootFilesystem == true",
          "reason": "An immutable root filesystem can prevent malicious binaries being added to PATH and increase attack cost",
          "points": 1
        }
      ]
    }
  }
]
```
### Kubeval
```
PASS - manifests/dvwa-configmap.yaml contains a valid ConfigMap (training.dvwa-config)
PASS - manifests/dvwa-deployment.yaml contains a valid Deployment (training.dvwa)
PASS - manifests/dvwa-service.yaml contains a valid Service (training.dvwa-service)
PASS - manifests/mysql-pvc.yaml contains a valid PersistentVolumeClaim (training.mysql-pvc)
PASS - manifests/mysql-secret.yaml contains a valid Secret (training.dvwa-secret)
PASS - manifests/mysql-service.yaml contains a valid Service (training.mysql-service)
PASS - manifests/mysql-statefulset.yaml contains a valid StatefulSet (training.mysql)
PASS - manifests/networkpolicy.yaml contains a valid NetworkPolicy (training.allow-dvwa-to-db)
```

### Trivy DVWA
```
Total: 805 (HIGH: 549, CRITICAL: 256)
```

### Trivy MySQL
```
Total: 12 (HIGH: 12, CRITICAL: 0)
```