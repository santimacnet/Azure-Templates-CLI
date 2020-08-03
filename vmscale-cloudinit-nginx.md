TUTORIAL VIRTUAL MACHINE SCALE SET PARA DEPLOY DE NGINX CON LOAD BALANCER
------------------------------------------------------------------------------------

Objetivos de aprendizaje

- Crear conjunto de VMs para servidor web NGINX y publicar una web
- Configurar Balanceador Checks y Reglas para que funcione servidor web NGINX
- Conocimientos de CloudInit: https://cloudinit.readthedocs.io/en/latest
- MSLEARN: https://docs.microsoft.com/es-es/learn/modules/build-app-with-scale-sets/3-exercise-deploy-scale-set-azure-portal
- MSDOCS: https://docs.microsoft.com/en-us/azure/virtual-machines/linux/tutorial-create-vmss

Creamos mediante agente de aprovisionamiento CloudInit el archivo de ejemplo **cloud-init-nginx.yml**
```
#cloud-config
package_upgrade: true
packages:
  - nginx
write_files:
  - owner: www-data:www-data
  - path: /var/www/html/index.html
    content: |
        Hola desde NGINX !!
runcmd:
  - service nginx restart
```

Creamos el grupo de recursos y 3 maquinas virtuales economicas de tipo Standard B1s o B2s.
```
$ az group create --name santi-webserver-demo-recursos --location northeurope 

# Creamos el conjunto de 3 maquinas virtuales
$ az vmss create \
  --resource-group santi-webserver-demo-recursos \
  --name vmss-webserver-demo \
  --vm-sku Standard_B1s \
  --instance-count 3 \
  --image UbuntuLTS \
  --upgrade-policy-mode automatic \
  --admin-username azureuser \
  --generate-ssh-keys \
  --custom-data cloud-init-nginx.yml
```

Configuramos el balanceador de carga para el conjunto de las maquinas virtuales
```
#Configurar Health probes
$ az network lb probe create \
  --lb-name vmss-webserver-demoLB \
  --resource-group santi-webserver-demo-recursos \
  --name webServerHealth \
  --port 80 \
  --protocol Http \
  --path /

#Configurar reglas de enrutado
$ az network lb rule create \
  --resource-group santi-webserver-demo-recursos \
  --name webServerLoadBalancerRuleWeb \
  --lb-name vmss-webserver-demoLB \
  --probe-name webServerHealth \
  --backend-pool-name vmss-webserver-demoLBBEPool \
  --backend-port 80 \
  --frontend-ip-name loadBalancerFrontEnd \
  --frontend-port 80 \
  --protocol tcp
  
# Ahora obtenemos IP publica del frontal o desde informacion general del conjunto de maquinas virtuales y abriendo un navegador.

url: xxx.xx.xx.xx

Hola desde NGINX !!
```

Ref SKUs: https://docs.microsoft.com/en-us/azure/virtual-machines/sizes-b-series-burstable



###Extensión de script de Azure
Para actualizar la aplicacion en un futuro en todas las VMSS necesitamos utilizar una extensión de script personalizado de Azure que descarga y ejecuta un script cada máquina virtual de Azure. 
```
az vmss extension set \
  --publisher Microsoft.Azure.Extensions \
  --version 2.0 \
  --name CustomScript \
  --resource-group myResourceGroup \
  --vmss-name yourScaleSet \
  --settings @yourConfigV1.json
 ```
 
Las máquinas virtuales se actualizan según la directiva de actualización del conjunto de escalado que indicamos en la creacion con el parametro **--upgrade-policy-mode automatic** esta directiva de actualización puede tener uno de los tres modos siguientes:

- Automática: el conjunto de escalado no define cuándo se actualizan las máquinas virtuales. Todas se podrían actualizar al mismo tiempo, lo que causaría una interrupción del servicio.
- Implementación: el conjunto de escalado implementa la actualización en lotes entre las máquinas virtuales del conjunto de escalado. Una pausa opcional puede minimizar o eliminar una interrupción del servicio. En este modo, es posible que las máquinas del conjunto de escalado ejecuten otras versiones de la aplicación durante un breve período de tiempo. En este modo es necesario que agregue un sondeo de mantenimiento al conjunto de escalado, o bien que le aplique la extensión de estado de la aplicación.
- Manual: las máquinas virtuales existentes en el conjunto de escalado no se actualizan. Todos los cambios se deben realizar de forma manual. Este modo es el predeterminado.



