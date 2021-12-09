# Laravel-istio

Este proyecto es para desplegar un proyecto de laravel en kubernetes y ser monitorizado con istio con el clúster de minikube.

Enseguida algunos prerrequisitos:
1. Tener instalado Docker Desktop.
2. Tener instalado Kubernetes (Funciona el que incluye Docker Desktop).
3. Tener instalado Minikube (Funciona el que incluye Docker Desktop).
4. Tener instalado Istio.

Y algunos detalles:
1. Docker Desktop requiere Hyper-V si se ejecuta en Windows, característica sólo incluída en Windows Pro o superior.
2. No se puede ejecutar Hyper-V y Minikube (con driver de VirtualBox) a la vez, por lo que en una consola con permisos de administrador se tendrá que hacer `bcdedit /set hypervisorlaunchtype off` para desactivar Hyper-V y `bcdedit /set hypervisorlaunchtype auto` para volverlo a activar. Cambio únicamente aplicado después de reiniciar el equipo.
3. Si Minikube no cuenta con los recursos suficientes de RAM o CPU a la hora de crear pods, tendremos problemas para establecer conexión estable con el clúster.

## Integrantes

- Daniel Michel
- No one else :(

## Instalación

1. Clonar proyecto `git clone git@github.com:Emulator-Dnl/Laravelistio.git` y cambiarse a directorio `cd Laravelistio`
2. Iniciar clúster `minikube start --memory 3200 --cpus 5` lo que puede tardar un poco.
3. Instalar Istio en el clúster `istioctl manifest apply --set profile=demo`
4. Añadir etiqueta para que los pods de Istio se inyecten en los de nuestro sistema `kubectl label namespace default istio-injection=enabled`
5. Aplicar los yaml de nuestra aplicación `kubectl apply -f volume.yaml,php_deployment.yaml,nginx_configmap.yaml,php_service.yaml,nginx_deployment.yaml,nginx_service.yaml`. Si desea utilizar un proyecto de Laravel distinto es necesario cambiar la imagen 'emulatordnl/laravelistio:1.0.2' a la que se hace referencia en el archivo 'php_deployment.yaml' por la suya. Si no sabe cómo crear la imagen vea el apartado 'Dockerizar...' de este README.
6. Habilitar ingress `minikube addons enable ingress` lo que puede tardar un par de minutos.
7. Aplicar yaml de ingress `kubectl apply -f ingress.yaml`
8. Obtenemos la ip en la cual podremos acceder a nuestra applicación `kubectl get ingress` puede llegar a tardar hasta un minuto.
9. Aplicamos los yaml de mysql `kubectl apply -f mysql`
10. Obtenemos el 'pod_id' de nuestro pod de php `kubectl get po`
11. Ésto para acceder a él `kubectl exec 'pod_id' -it -- bash`. Se desplegará un promt diferente (el del pod).
12. En él ejecutamos las migraciones y seeders para cargar catálogos `php /var/www/html/artisan migrate:fresh --seed`
13. Me gusta aplicar los addons de Istio hast el final para no ralentizar algunos de los pasos anteriores `kubectl apply -f addons`

## Testing

-Para comprobar el funcionamiento del proyecto y su alta disponibilidad se puede eliminar alguno de los pods `kubectl delete pod 'pod_id'`. Al hacer escalamiento `kubectl scale --replicas=3 -f php_deployment.yaml` podremos observar el balanceador y cómo la aplicación sigue en pie siempre y cuando haya al menos un pod por microservicio.

## Dockerizar proyecto Laravel

En el proyecto se incluye el Dockerfile necesario para tal tarea. Sólo se necesita colocar el Dockerfile en la carpeta del proyecto de Laravel y crear la imagen `docker build . -t 'user_name'/'repo_name':1.0.2` para luego hacerle push a Docker Hub `docker push 'user_name'/'repo_name':1.0.2`. Obviamente con su propio usuario y repositorio.

## Funcionmiento del proyecto con [esta aplicación Laravel](https://github.com/Emulator-Dnl/Dinamita/tree/master)

[Ver video](https://drive.google.com/file/d/1tf0Wttdf_jZjINHTHsJUm3TwIJE2CzoO/view?usp=sharing)

## Licencia

[MIT license](https://opensource.org/licenses/MIT).

