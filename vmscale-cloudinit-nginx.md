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

Creamos el grupo de recursos y un conjunto de 3 maquinas virtuales economica tipo Standard B1s o B2s.
```
$ az group create --name santi-webserver-demo-recursos --location northeurope 

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
Ref SKUs: https://docs.microsoft.com/en-us/azure/virtual-machines/sizes-b-series-burstable

Configuramos el balanceador de carga para el conjunto de las maquinas virtuales
```
#configurar probe
$ az network lb probe create \
  --lb-name webServerScaleSetLB \
  --resource-group scalesetrg \
  --name webServerHealth \
  --port 80 \
  --protocol Http \
  --path /

#configurar reglas de enrutado
$ az network lb rule create \
  --resource-group scalesetrg \
  --name webServerLoadBalancerRuleWeb \
  --lb-name webServerScaleSetLB \
  --probe-name webServerHealth \
  --backend-pool-name webServerScaleSetLBBEPool \
  --backend-port 80 \
  --frontend-ip-name loadBalancerFrontEnd \
  --frontend-port 80 \
  --protocol tcp
```

Ahora solo nos falta verificar que todo esto funciona obteniendo la IP publica del frontal del balanceador o desde la informacion general del conjunto de maquinas virtuales y abriendo un navegador.

```
url: xxx.xx.xx.xx

Hola desde NGINX !!

```

